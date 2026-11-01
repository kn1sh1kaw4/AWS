## api.tf

---

```hcl
resource "aws_cloudwatch_log_group" "api" {
  name              = "/ecs/${var.project}/api"
  retention_in_days = 30
}

resource "aws_lb_target_group" "app" {
  name        = "${var.project}-app-tg"
  port        = 80
  protocol    = "HTTP"
  target_type = "ip"
  vpc_id      = aws_vpc.main.id
  health_check {
    path    = "/"
    matcher = "200-399"
  }
}


resource "aws_lb_target_group" "api" {
  name        = "${var.project}-api-tg"
  port        = 80
  protocol    = "HTTP"
  target_type = "ip"
  vpc_id      = aws_vpc.main.id

  health_check {
    path    = "/"
    matcher = "200-399"
  }
}

resource "aws_lb_listener_rule" "api" {
  listener_arn = aws_lb_listener.http.arn
  priority     = 10
  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.api.arn
  }
  condition {
    path_pattern {
      values = ["/api/*"]
    }
  }
}

resource "aws_ecs_task_definition" "api" {
  family                   = "${var.project}-api"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = "256"
  memory                   = "512"
  execution_role_arn       = aws_iam_role.ecs_execution.arn
  task_role_arn            = aws_iam_role.ecs_task.arn

  container_definitions = jsonencode([{
    name         = "api"
    image        = var.container_image
    essential    = true
    portMappings = [{ containerPort = 80, protocol = "tcp" }]
    environment = [
      { name = "CONTENT_BUCKET", value = aws_s3_bucket.content.bucket },
      { name = "METADATA_TABLE", value = aws_dynamodb_table.metadata.name },
      { name = "GENERATION_QUEUE_URL", value = aws_sqs_queue.generation.url }
    ]
    logConfiguration = {
      logDriver = "awslogs"
      options = {
        awslogs-group         = aws_cloudwatch_log_group.api.name
        awslogs-region        = var.region
        awslogs-stream-prefix = "api"
      }
    }
  }])
}

resource "aws_ecs_service" "api" {
  name            = "${var.project}-api"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.api.arn
  desired_count   = 1
  launch_type     = "FARGATE"

  network_configuration {
    subnets          = aws_subnet.app[*].id
    security_groups  = [aws_security_group.ecs.id]
    assign_public_ip = false
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.api.arn
    container_name   = "api"
    container_port   = 80
  }

  depends_on = [aws_lb_listener_rule.api]
}
```


### 01. resource "aws_cloudwatch_log_group" "api"{...}
　Amazon Web Services (AWS) の CloudWatch Logs に「api」という識別名（リソース名）でロググループを作成・管理する定義

### 02. resource "aws_lb_target_group" "app" {...}
　AWSのロードバランサー（ALBやNLB）が通信を振り分ける先（ターゲットグループ） を作成・管理するためのリソース

### 03. resource "aws_lb_listener_rule" "api" {...}
　Application Load Balancer（ALB）のリスナールールを作成・管理するリソース

### 04. resource "aws_lb_listener_rule" "api" {...}
　作成したロードバランサー（ALB/NLB）のリスナーの識別子（ARN）を別のリソースに引き渡す、または変数に代入しているコードを意味する。

### 05. resource "aws_ecs_task_definition" "api"
　Amazon ECS（Elastic Container Service）上で動作するコンテナの実行・管理単位（サービス）を定義するためのリソース 

### 06. resource "aws_ecs_service" "api"
　ECSサービスが起動する前に、Application Load Balancer (ALB) のリスナールールが確実に作成されている状態を作る


