## main.tf

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}
data "aws_caller_identity" "current" {
}
# ---------------- VPC / Network ----------------
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true
}
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
}

locals {
  azs = slice(data.aws_availability_zones.available.names, 0, 2)
}

# Public Subnet
resource "aws_subnet" "public" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = local.azs[count.index]
  map_public_ip_on_launch = false
  tags                    = { Name = "${var.project}-public-${count.index + 1}", Tier = "public" }
}

# Private App Subnet
resource "aws_subnet" "app" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 11}.0/24"
  availability_zone       = local.azs[count.index]
  map_public_ip_on_launch = false

  tags = {
    Name = "${var.project}-private-app-${count.index + 1}"
    Tier = "private-app"
  }
}

# Private DB Subnet
resource "aws_subnet" "db" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 21}.0/24"
  availability_zone       = local.azs[count.index]
  map_public_ip_on_launch = false

  tags = {
    Name = "${var.project}-private-db-${count.index + 1}"
    Tier = "private-db"
  }
}

# Public Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

# Public Subnet -> Public Route Table
resource "aws_route_table_association" "public" {
  count          = 2
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

#EIP
resource "aws_eip" "nat" {
  count  = var.enable_nat ? 1 : 0
  domain = "vpc"
}

# NAT Gateway
resource "aws_nat_gateway" "nat" {
  count         = var.enable_nat ? 1 : 0
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  depends_on    = [aws_internet_gateway.igw]
}

#Private App Route Table
resource "aws_route_table" "app" {
  count  = 1
  vpc_id = aws_vpc.main.id
  dynamic "route" {
    for_each = var.enable_nat ? [1] : []
    content {
      cidr_block     = "0.0.0.0/0"
      nat_gateway_id = aws_nat_gateway.nat[0].id
    }
  }
}

#Private DB Route Table
resource "aws_route_table" "db" {
  vpc_id = aws_vpc.main.id
}

# Private DB Subnet -> DB Route Table
resource "aws_route_table_association" "db" {
  count          = 2
  subnet_id      = aws_subnet.db[count.index].id
  route_table_id = aws_route_table.db.id
}

# Private App Subnet -> Private App Route Table
resource "aws_route_table_association" "app" {
  count          = 2
  subnet_id      = aws_subnet.app[count.index].id
  route_table_id = aws_route_table.app[0].id
}

# ---------------- Security Groups ----------------
resource "aws_security_group" "alb" {
  name   = "${var.project}-alb-sg"
  vpc_id = aws_vpc.main.id
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
resource "aws_security_group" "ecs" {
  name   = "${var.project}-ecs-sg"
  vpc_id = aws_vpc.main.id
  ingress {
    from_port       = 80
    to_port         = 80
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
resource "aws_security_group" "aurora" {
  count  = var.enable_aurora ? 1 : 0
  name   = "${var.project}-aurora-sg"
  vpc_id = aws_vpc.main.id
  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.ecs.id]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# ---------------- S3 / DynamoDB ----------------
resource "aws_s3_bucket" "content" {
  bucket_prefix = "${var.project}-content-"
  force_destroy = true
}
resource "aws_s3_bucket_public_access_block" "content" {
  bucket                  = aws_s3_bucket.content.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
resource "aws_s3_bucket_versioning" "content" {
  bucket = aws_s3_bucket.content.id
  versioning_configuration {
    status = "Enabled"
  }
}
resource "aws_s3_bucket_server_side_encryption_configuration" "content" {
  bucket = aws_s3_bucket.content.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
resource "aws_dynamodb_table" "metadata" {
  name         = "${var.project}-${var.environment}-metadata"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "content_id"
  attribute {
    name = "content_id"
    type = "S"
  }
  point_in_time_recovery {
    enabled = true
  }
  server_side_encryption {
    enabled = true
  }
}

resource "aws_backup_selection" "main" {
  name         = "ai-content-platform-backup-selection"
  iam_role_arn = aws_iam_role.backup[0].arn
  plan_id      = aws_backup_plan.main[0].id

  resources = [
    aws_dynamodb_table.metadata.arn
  ]
}

# ---------------- IAM ----------------
resource "aws_iam_role" "ecs_execution" {
  name               = "${var.project}-ecs-execution"
  assume_role_policy = jsonencode({ Version = "2012-10-17", Statement = [{ Effect = "Allow", Principal = { Service = "ecs-tasks.amazonaws.com" }, Action = "sts:AssumeRole" }] })
}
resource "aws_iam_role_policy_attachment" "ecs_execution" {
  role       = aws_iam_role.ecs_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}
resource "aws_iam_role" "ecs_task" {
  name               = "${var.project}-ecs-task"
  assume_role_policy = jsonencode({ Version = "2012-10-17", Statement = [{ Effect = "Allow", Principal = { Service = "ecs-tasks.amazonaws.com" }, Action = "sts:AssumeRole" }] })
}
resource "aws_iam_role_policy" "ecs_task" {
  role = aws_iam_role.ecs_task.id
  policy = jsonencode({ Version = "2012-10-17", Statement = [
    { Effect = "Allow", Action = ["s3:GetObject", "s3:PutObject"], Resource = "${aws_s3_bucket.content.arn}/*" },
    { Effect = "Allow", Action = ["s3:ListBucket"], Resource = aws_s3_bucket.content.arn },
    { Effect = "Allow", Action = ["dynamodb:GetItem", "dynamodb:PutItem", "dynamodb:UpdateItem", "dynamodb:Query"], Resource = aws_dynamodb_table.metadata.arn },
    { Effect = "Allow", Action = ["sqs:SendMessage", "sqs:GetQueueAttributes"], Resource = aws_sqs_queue.generation.arn },
    { Effect = "Allow", Action = ["bedrock:InvokeModel", "bedrock:InvokeModelWithResponseStream"], Resource = "*" }
  ] })
}
resource "aws_iam_role" "lambda" {
  name               = "${var.project}-lambda"
  assume_role_policy = jsonencode({ Version = "2012-10-17", Statement = [{ Effect = "Allow", Principal = { Service = "lambda.amazonaws.com" }, Action = "sts:AssumeRole" }] })
}
resource "aws_iam_role_policy" "lambda" {
  role = aws_iam_role.lambda.id
  policy = jsonencode({ Version = "2012-10-17", Statement = [
    { Effect = "Allow", Action = ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"], Resource = "*" },
    { Effect = "Allow", Action = ["sqs:ReceiveMessage", "sqs:DeleteMessage", "sqs:GetQueueAttributes"], Resource = aws_sqs_queue.generation.arn }
  ] })
}

# ---------------- ALB / ECS Fargate ----------------
resource "aws_cloudwatch_log_group" "ecs" {
  name              = "/ecs/${var.project}"
  retention_in_days = 30
}
resource "aws_ecr_repository" "app" {
  name                 = "${var.project}-app"
  image_tag_mutability = "IMMUTABLE"
  image_scanning_configuration {
    scan_on_push = true
  }
}
resource "aws_ecs_cluster" "main" {
  name = "${var.project}-cluster"
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}
resource "aws_lb" "app" {
  name               = "${var.project}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = aws_subnet.public[*].id
}
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.app.arn
  port              = 80
  protocol          = "HTTP"
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}

resource "aws_ecs_task_definition" "app" {
  family                   = "${var.project}-app"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "256"
  memory                   = "512"
  execution_role_arn       = aws_iam_role.ecs_execution.arn
  task_role_arn            = aws_iam_role.ecs_task.arn
  container_definitions    = jsonencode([{ name = "app", image = var.container_image, essential = true, portMappings = [{ containerPort = 80, protocol = "tcp" }], logConfiguration = { logDriver = "awslogs", options = { awslogs-group = aws_cloudwatch_log_group.ecs.name, awslogs-region = var.region, awslogs-stream-prefix = "ecs" } } }])
}
resource "aws_ecs_service" "app" {
  name            = "${var.project}-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 1
  launch_type     = "FARGATE"
  network_configuration {
    subnets          = aws_subnet.app[*].id
    security_groups  = [aws_security_group.ecs.id]
    assign_public_ip = false
  }
  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = 80
  }
  depends_on = [aws_lb_listener.http]
}

# ---------------- SQS / Lambda / Step Functions ----------------
resource "aws_sqs_queue" "generation_dlq" {
  name = "${var.project}-generation-dlq"
}
resource "aws_sqs_queue" "generation" {
  name                       = "${var.project}-generation"
  visibility_timeout_seconds = 180
  redrive_policy             = jsonencode({ deadLetterTargetArn = aws_sqs_queue.generation_dlq.arn, maxReceiveCount = 3 })
}
resource "aws_lambda_function" "worker" {
  function_name    = "${var.project}-worker"
  role             = aws_iam_role.lambda.arn
  runtime          = "python3.12"
  handler          = "index.handler"
  filename         = "${path.module}/lambda.zip"
  source_code_hash = filebase64sha256("${path.module}/lambda.zip")
  timeout          = 30
  memory_size      = 256
  environment {
    variables = { 
      QUEUE_URL = aws_sqs_queue.generation.url
      BUCKET = aws_s3_bucket.content.bucket
    }
  }
}
resource "aws_lambda_event_source_mapping" "sqs" {
  event_source_arn = aws_sqs_queue.generation.arn
  function_name    = aws_lambda_function.worker.arn
  batch_size       = 1
}
resource "aws_iam_role" "sfn" {
  name               = "${var.project}-sfn"
  assume_role_policy = jsonencode({ Version = "2012-10-17", Statement = [{ Effect = "Allow", Principal = { Service = "states.amazonaws.com" }, Action = "sts:AssumeRole" }] })
}
resource "aws_iam_role_policy" "sfn" {
  role   = aws_iam_role.sfn.id
  policy = jsonencode({ Version = "2012-10-17", Statement = [{ Effect = "Allow", Action = ["sqs:SendMessage"], Resource = aws_sqs_queue.generation.arn }] })
}
resource "aws_sfn_state_machine" "generation" {
  name       = "${var.project}-generation"
  role_arn   = aws_iam_role.sfn.arn
  definition = jsonencode({ StartAt = "Queue", States = { Queue = { Type = "Task", Resource = "arn:aws:states:::sqs:sendMessage", Parameters = { QueueUrl = aws_sqs_queue.generation.url, "MessageBody.$" = "$" }, End = true } } })
}

# ---------------- Aurora (optional, paid) ----------------
resource "aws_db_subnet_group" "aurora" {
  count      = var.enable_aurora ? 1 : 0
  name       = "${var.project}-aurora-subnets"
  subnet_ids = aws_subnet.db[*].id
}
resource "random_password" "aurora" {
  count   = var.enable_aurora ? 1 : 0
  length  = 24
  special = true
}
resource "aws_rds_cluster" "aurora" {
  count                   = var.enable_aurora ? 1 : 0
  cluster_identifier      = "${var.project}-aurora"
  engine                  = "aurora-postgresql"
  database_name           = "contentdb"
  master_username         = "appadmin"
  master_password         = random_password.aurora[0].result
  db_subnet_group_name    = aws_db_subnet_group.aurora[0].name
  vpc_security_group_ids  = [aws_security_group.aurora[0].id]
  storage_encrypted       = true
  backup_retention_period = 7
  skip_final_snapshot     = true
}
resource "aws_secretsmanager_secret" "aurora" {
  count = var.enable_aurora ? 1 : 0
  name  = "${var.project}/${var.environment}/aurora"
}
resource "aws_secretsmanager_secret_version" "aurora" {
  count         = var.enable_aurora ? 1 : 0
  secret_id     = aws_secretsmanager_secret.aurora[0].id
  secret_string = jsonencode({ username = "appadmin", password = random_password.aurora[0].result, database = "contentdb", host = aws_rds_cluster.aurora[0].endpoint, port = 5432 })
}
resource "aws_rds_cluster_instance" "aurora" {
  count              = var.enable_aurora ? 1 : 0
  identifier         = "${var.project}-aurora-1"
  cluster_identifier = aws_rds_cluster.aurora[0].id
  instance_class     = "db.t4g.medium"
  engine             = "aurora-postgresql"
}

# ---------------- VPC Flow Logs ----------------
resource "aws_cloudwatch_log_group" "flow" {
  name              = "/aws/vpc/flowlogs/${var.project}"
  retention_in_days = 30
}
resource "aws_iam_role" "flow" {
  name               = "${var.project}-vpc-flowlogs"
  assume_role_policy = jsonencode({ Version = "2012-10-17", Statement = [{ Effect = "Allow", Principal = { Service = "vpc-flow-logs.amazonaws.com" }, Action = "sts:AssumeRole" }] })
}
resource "aws_iam_role_policy" "flow" {
  role   = aws_iam_role.flow.id
  policy = jsonencode({ Version = "2012-10-17", Statement = [{ Effect = "Allow", Action = ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents", "logs:DescribeLogGroups", "logs:DescribeLogStreams"], Resource = "*" }] })
}
resource "aws_flow_log" "vpc" {
  vpc_id          = aws_vpc.main.id
  traffic_type    = "ALL"
  iam_role_arn    = aws_iam_role.flow.arn
  log_destination = aws_cloudwatch_log_group.flow.arn
}

# ---------------- CloudTrail ----------------
resource "aws_s3_bucket" "audit" {
  bucket_prefix = "${var.project}-audit-"
  force_destroy = true
}
resource "aws_s3_bucket_public_access_block" "audit" {
  bucket                  = aws_s3_bucket.audit.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
resource "aws_s3_bucket_versioning" "audit" {
  bucket = aws_s3_bucket.audit.id
  versioning_configuration {
    status = "Enabled"
  }
}
resource "aws_s3_bucket_policy" "audit" {
  bucket = aws_s3_bucket.audit.id
  policy = jsonencode({ Version = "2012-10-17", Statement = [
    { Sid = "AclCheck", Effect = "Allow", Principal = { Service = "cloudtrail.amazonaws.com" }, Action = "s3:GetBucketAcl", Resource = aws_s3_bucket.audit.arn },
    { Sid = "Write", Effect = "Allow", Principal = { Service = "cloudtrail.amazonaws.com" }, Action = "s3:PutObject", Resource = "${aws_s3_bucket.audit.arn}/AWSLogs/${data.aws_caller_identity.current.account_id}/*", Condition = { StringEquals = { "s3:x-amz-acl" = "bucket-owner-full-control" } } }
  ] })
}
resource "aws_cloudtrail" "main" {
  name                          = "${var.project}-trail"
  s3_bucket_name                = aws_s3_bucket.audit.id
  include_global_service_events = true
  is_multi_region_trail         = true
  enable_log_file_validation    = true
  depends_on                    = [aws_s3_bucket_policy.audit]
}

# ---------------- CloudFront + WAF ----------------
resource "aws_wafv2_web_acl" "cf" {
  provider = aws.use1
  count    = var.enable_cloudfront ? 1 : 0
  name     = "${var.project}-waf"
  scope    = "CLOUDFRONT"
  default_action {
    allow {
    }
  }
  rule {
    name     = "AWSManagedCommon"
    priority = 1
    override_action {
      none {
      }
    }
    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesCommonRuleSet"
        vendor_name = "AWS"
      }
    }
    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "common"
      sampled_requests_enabled   = true
    }
  }
  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "${var.project}-waf"
    sampled_requests_enabled   = true
  }
}
resource "aws_cloudfront_distribution" "app" {
  count   = var.enable_cloudfront ? 1 : 0
  enabled = true
  origin {
    domain_name = aws_lb.app.dns_name
    origin_id   = "alb"
    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "http-only"
      origin_ssl_protocols   = ["TLSv1.2"]
    }
  }
  default_cache_behavior {
    allowed_methods        = ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "alb"
    viewer_protocol_policy = "redirect-to-https"
    forwarded_values {
      query_string = true
      cookies {
        forward = "all"
      }
    }
  }
  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }
  viewer_certificate {
    cloudfront_default_certificate = true
  }
  web_acl_id = aws_wafv2_web_acl.cf[0].arn
}
```

### 01. data "aws_availability_zones" "available"
使用しているAWSリージョン内で現在利用可能なアベイラビリティゾーン（AZ）の一覧を動的に取得するための機能です。

### 02. data "aws_caller_identity" "current"
現在Terraformを実行しているAWSアカウントやIAMユーザー（またはロール）の情報を取得するためのデータソースです。

### 03. resource "aws_vpc" "main"
Terraformで AWSのVPC（Virtual Private Cloud：仮想ネットワーク）を「main」という識別名（論理名）で作成するためのコードの記述です。

### 04. resource "aws_internet_gateway" "igw"
Amazon Web Services（AWS）のインターネットゲートウェイ（Internet Gateway: IGW）を構築・管理するために使用します。 インターネットゲートウェイは、VPC（Virtual Private Cloud）内のリソース（EC2インスタンスなど）とインターネットとの間で双方向の通信を可能にするためのコンポーネントです。

### 05. locals
Terraformのコード内だけで完結する「内部用の変数」です。外部から値を受け取ったり上書きしたりすることはできません。

### 06. resource "aws_subnet" "public"
　AWSのVPC（Virtual Private Cloud）内にサブネットを作成するための設定です。

### 07. resource "aws_subnet" "app"
AWSのVPC（Virtual Private Cloud）内にサブネット（Subnet）を作成・管理するためのリソースです。

### 08. resource "aws_subnet" "db"
AWSのVPC（Virtual Private Cloud）内にサブネットを作成するためのリソースです。質問にある "aws_subnet" "db" という記述は、データベース（DB）用のサーバーを配置するためのサブネットを定義する際によく使われる命名パターンです。

### 09. resource "aws_route_table" "public"
Amazon VPC（Virtual Private Cloud）内にルーティングテーブル（経路制御表）を作成・管理するためのものです。

### 10. resource "aws_route_table_association" "public"
VPC内のサブネット（またはインターネットゲートウェイや仮想プライベートゲートウェイ）と、特定のルートテーブル（ルーティング情報）を紐付ける（関連付ける）ためのリソースです。

### 11. resource "aws_eip" "nat"
AWSのNATゲートウェイに割り当てるための固定グローバルIPアドレス（Elastic IP）を確保・管理するための記述です。

### 12. resource "aws_nat_gateway" "nat"
AWSのNATゲートウェイを構築・管理するためのリソースです。

### 13. resource "aws_route_table" "app"
AWSのVPC（Virtual Private Cloud）内におけるルーティングルール（ルートテーブル）を作成・管理するためのものです。

### 14. resource "aws_route_table" "db"
AWS の VPC（Virtual Private Cloud）内において、サブネットから送信されるネットワークトラフィックのルーティング（経路）を制御するためのルートテーブルを作成・管理するものです。

### 15. resource "aws_route_table_association" "db"
サブネットまたはインターネットゲートウェイ/仮想プライベートゲートウェイを、特定のルートテーブル（経路制御表）に関連付けるために使用されます。

### 16. resource "aws_route_table_association" "app"
VPC内のサブネット（またはインターネットゲートウェイや仮想プライベートゲートウェイ）とルートテーブルを紐付ける（関連付ける）ためのリソースです。

### 17. resource "aws_security_group" "alb"
AWS上にセキュリティグループ（仮想ファイアウォール）を作成・管理するためのリソースです。

### 18. resource "aws_security_group" "ecs"
Amazon ECS（Elastic Container Service）のタスクやサービス、あるいはEC2起動タイプのインスタンスに対して、インバウンド（受信）およびアウトバウンド（送信）のネットワークトラフィックを制御する仮想ファイアウォールを定義するために使用します。

### 19. resource "aws_security_group" "aurora"
Amazon Aurora（RDS）データベースクラスターに適用するファイアウォール（セキュリティグループ）を定義するために使用します。

### 20. resource "aws_s3_bucket" "content"
AWS上にS3バケット（オブジェクトストレージ）を作成・管理するための最も基本的かつ重要なリソースです。

### 21. resource "aws_s3_bucket_public_access_block" "content"
Amazon S3 バケットに対するパブリックアクセスを完全に遮断（ブロック）するリソースです。

### 22. resource "aws_s3_bucket_versioning" "content"
Amazon S3 バケットのバージョニング機能（オブジェクトの世代管理）を有効化、無効化、または一時停止（サスペンド）するための Terraform リソースです。

### 23. resource "aws_s3_bucket_server_side_encryption_configuration" "content"
Amazon S3 バケットのサーバー側暗号化（SSE）ルールを設定・管理するための Terraform リソースです。

### 24. resource "aws_dynamodb_table" "metadata"
AWSのフルマネージドNoSQLデータベースサービスである Amazon DynamoDBのテーブルを作成・管理するため のリソースです。

### 25. resource "aws_backup_selection" "main"
Terraformで AWS Backupのバックアッププランに対して、どのリソース（EC2、RDS、EFSなど）をバックアップ対象にするかを割り当てる（選択する）ためのリソースです。

### 26. resource "aws_iam_role" "ecs_execution"
Amazon ECS（Elastic Container Service）のコンテナエージェント自体が、AWSの他のサービス（Amazon ECRやAWS Secrets Managerなど）にアクセスするために使用するIAMロールです。

### 27. resource "aws_iam_role_policy_attachment" "ecs_execution"
IAMロール（リソースが作成・利用する権限の定義）に、特定のIAMポリシー（具体的な権限のセット）を紐付ける（アタッチする）ためのものです。

### 28. resource "aws_iam_role" "ecs_task"
通常 ECSタスク実行ロール（Task Execution Role） または ECSタスクロール（Task Role） のいずれかとして使用されるIAMロールを指します。

### 29. resource "aws_iam_role_policy" "ecs_task"
特定のIAMロールに「インラインポリシー（埋め込み型の権限）」を直接定義してアタッチするためのリソースです。

### 30. resource "aws_iam_role" "lambda"
AWS Lambda関数がAWSの他のサービスにアクセスしたり実行したりするために必要なIAMロール（実行ロール）を作成・管理するリソースです。

### 31. resource "aws_iam_role_policy" "lambda"
特定のIAMロールに「インラインポリシー（埋め込みポリシー）」を直接作成してアタッチするためのリソースです。

### 32. resource "aws_cloudwatch_log_group" "ecs"
AWSの監視サービスであるAmazon CloudWatchにロググループ（Log Group）を作成・管理するためのリソースです。

### 33. resource "aws_ecr_repository" "app"
AWSのプライベートまたはパブリックのコンテナイメージレジストリである Amazon ECR (Elastic Container Registry) にリポジトリを作成・管理するため のリソースです。

### 34. resource "aws_ecs_cluster" "main"
AWS上に Amazon ECS（Elastic Container Service）クラスターを作成・管理するため のリソースです。

### 35. resource "aws_lb" "app"
AWSのロードバランサー（Application Load Balancer / Network Load Balancer / Gateway Load Balancer）を作成・管理するためのリソース定義です。

### 36. resource "aws_lb_listener" "http"
AWSのロードバランサー（ALBやNLB）がクライアントからの接続リクエストを受け付けるための「リスナー」を作成・管理するものです。

### 37. resource "aws_ecs_task_definition" "app" 
Amazon ECS（Elastic Container Service）でコンテナを実行するためのタスク定義（Task Definition）を作成・管理するためのものです。

### 38. resource "aws_ecs_service" "app"
AWS の Elastic Container Service (ECS) 上でコンテナ（タスク）を起動し、指定した個数を維持・管理するための「サービス」を定義するリソースです。

### 39. resource "aws_sqs_queue" "generation_dlq"

### 40. resource "aws_sqs_queue" "generation"
AWSのマネージド型メッセージキューサービスである SQS（Simple Queue Service）を構築・管理するためのリソース です。

### 41. resource "aws_lambda_function" "worker"
AWS Lambda関数（aws_lambda_function）を定義する設定です。ワーカー（処理実行用）としてPython 3.12を実行するLambdaリソースを作成しています。

### 42. resource "aws_lambda_event_source_mapping" "sqs"
Amazon SQSなどのイベントソースとAWS Lambda関数を紐付ける（マッピングする）ために使用されます。

### 43. resource "aws_iam_role" "sfn"
AWS上でIAMロール（Identity and Access Management Role）を作成・管理するためのコードです。

### 44. resource "aws_iam_role_policy" "sfn"
IAMロールに対して「インラインポリシー（固有のポリシー）」を直接埋め込んで定義するために使用されます。

### 45. resource "aws_sfn_state_machine" "generation"
AWS Step Functionsのステートマシン（ワークフロー）を作成・管理するためのリソースです。

### 46. resource "aws_db_subnet_group" "aurora"
Amazon RDSやAuroraなどのデータベースクラスターをデプロイする VPC内のサブネットをグループ化するためのリソース です。

### 47. resource "random_password" "aurora"
暗号論的に強力なランダムなパスワード（文字列）を生成するためのリソースです。

### 48. resource "aws_rds_cluster" "aurora"
special = true は、おもにランダムな文字列を生成する random_password や random_string などのリソースで使われる設定（引数）です。

### 49. resource "aws_secretsmanager_secret" "aurora"
AWS Secrets Manager内で「シークレット（秘密情報）の箱（器）」を作成・管理するためのリソースです。

### 50. resource "aws_secretsmanager_secret_version" "aurora"
AWS Secrets Managerの「シークレットの値（データ本体）」を管理・登録するためのリソースです。

### 51. resource "aws_rds_cluster_instance" "aurora"
Amazon Auroraなどの RDSクラスター内に配置する個々のデータベースインスタンス（DBインスタンス）を管理・構築するため のリソースです。

### 52. resource "aws_cloudwatch_log_group" "flow"
AWSの CloudWatch Logsに「ロググループ」を作成・管理するため のものです。
VPCフローログ、Lambdaの出力、ECSのタスクログなど、様々なAWSサービスのログを集約する箱（グループ）を定義する際に必ず使用されます。

### 53. resource "aws_iam_role" "flow"
AWS上でIAMロール（Identity and Access Management Role）を作成・管理するためのリソースです。
IAMロールは、AWSのサービス（VPC Flow LogsやEC2など）やユーザーに対して、特定のAWSリソースを操作する一時的な権限（認証情報）を付与するために使用されます。

### 54. resource "aws_iam_role_policy" "flow"
AWSの特定のIAMロールに、CloudWatch Logsへログを出力・管理するための権限（インラインポリシー）を付与する

### 55. resource "aws_flow_log" "vpc"
Amazon VPC 内のネットワークトラフィック（送信元/送信先IP、ポート、プロトコル、パケット数など）を記録する「VPC フローログ」を作成・管理するために使用されます。
このログを収集することで、セキュリティ監査、ネットワークのトラブルシューティング、および異常なトラフィックの検出に役立てることができます。

### 56. resource "aws_s3_bucket" "audit"
AWS上にS3バケットを作成・管理するための設定です。AWSでログの保存やデータのアーカイブを行う際の最も基本となるコンポーネントの1つです。

### 57. resource "aws_s3_bucket_public_access_block" "audit"
Amazon S3 バケットに対するパブリックアクセスを制限・ブロックするための Terraform リソースです。

### 58. resource "aws_s3_bucket_versioning" "audit"
Amazon S3 バケットのバージョニング機能（オブジェクトの履歴管理）を有効化、無効化、または一時停止するための Terraform リソースです。
従来（AWS Provider v3 まで）は aws_s3_bucket リソース内のインライン引数として設定していましたが、AWS Provider v4 以降は独立したリソースとして分割されました。

### 59. resource "aws_s3_bucket_policy" "audit"
特定のAmazon S3バケットに対して「バケットポリシー」を設定・管理するために使用します。

### 60. resource "aws_cloudtrail" "main"
AWSアカウント内のAPIコールやイベント履歴を記録する「AWS CloudTrail」を設定・管理するためのものです。
CloudTrailを有効にすることで、「誰が・いつ・どのサービスに対して・何をしたか」という監査ログを自動的に取得し、セキュリティの監視やトラブルシューティングに役立てることができます。

### 61. resource "aws_wafv2_web_acl" "cf"
AWS WAFv2（Web Application Firewall）の Web ACL（アクセス制御リスト）を作成・管理するためのリソースです。
CloudFront（cf）や、ALB、Amazon API Gateway、AWS AppSync、Amazon Cognito などのリソースに適用して、SQLインジェクションやクロスサイトスクリプティング（XSS）、DDoS攻撃などの悪意のあるWebリクエストからアプリケーションを保護します。

### 62. resource "aws_cloudfront_distribution" "app"
AWSのコンテンツ配信ネットワーク（CDN）サービスである Amazon CloudFront のディストリビューションを構築・管理するためのリソースです。
ウェブサイトやAPIの配信を高速化し、セキュリティ（SSL/TLSやWAF）を向上させるために非常によく使われる重要なリソースの一つです。
