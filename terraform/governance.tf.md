## governance.tf

---

```hcl
# Account/organization governance is intentionally separated from the application stack.
# AWS Organizations and Control Tower should be managed from the Organizations management
# account as a dedicated landing-zone/governance project.

resource "aws_backup_vault" "main" {
  count       = var.enable_backup ? 1 : 0
  name        = "${var.project}-${var.environment}-backup"
  kms_key_arn = "arn:aws:kms:[リージョン]:[ユーザID]:key/[キー]"
}

resource "aws_iam_role" "backup" {
  count = var.enable_backup ? 1 : 0
  name  = "${var.project}-backup-role"
  assume_role_policy = jsonencode({
    Version   = "2012-10-17",
    Statement = [{ Effect = "Allow", Principal = { Service = "backup.amazonaws.com" }, Action = "sts:AssumeRole" }]
  })
}

resource "aws_iam_role_policy_attachment" "backup" {
  count      = var.enable_backup ? 1 : 0
  role       = aws_iam_role.backup[0].name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSBackupServiceRolePolicyForBackup"
}

resource "aws_backup_plan" "main" {
  count = var.enable_daily_backup ? 1 : 0
  name  = "${var.project}-${var.environment}-backup-daily-plan"
  rule {
    rule_name         = "daily"
    target_vault_name = aws_backup_vault.main[0].name
    schedule          = "cron(0 18 * * ? *)"
    lifecycle {
      delete_after = var.backup_retention_days
    }
  }
}

resource "aws_backup_plan" "example" {
  count = var.enable_weekly_backup ? 1 : 0
  name  = "${var.project}-${var.environment}-backup-weekly-plan"
  rule {
    rule_name         = "weekly"
    target_vault_name = aws_backup_vault.main[0].name
    schedule          = "cron(0 18 ? * 1 *)"
    lifecycle {
      delete_after = var.backup_retention_weeks
    }
  }
}

resource "aws_backup_selection" "aurora" {
  count        = var.enable_backup && var.enable_aurora ? 1 : 0
  name         = "aurora"
  plan_id      = aws_backup_plan.main[0].id
  iam_role_arn = aws_iam_role.backup[0].arn
  resources    = [aws_rds_cluster.aurora[0].arn]
}

resource "aws_sns_topic" "alerts" {
  name = "${var.project}-${var.environment}-alerts"
}

resource "aws_sns_topic_subscription" "email" {
  count     = var.alarm_email == "" ? 0 : 1
  topic_arn = aws_sns_topic.alerts.arn
  protocol  = "email"
  endpoint  = var.alarm_email
}
```


### 01. resource "aws_backup_vault" "main"
　AWS Backup でバックアップ（リカバリポイント）を安全に保存・管理するための「バックアップボルト（保管庫）」を作成・設定する

### 02. resource "aws_iam_role" "backup"
　AWS上でIAMロール（認証・認可の仕組み）を作成・管理するために使用されます。

### 03. resource "aws_iam_role_policy_attachment" "backup"
　IAMロールに対して特定のIAMポリシーを紐付ける（アタッチする）ためのリソースです。

### 04. resource "aws_backup_plan" "main"
　AWS Backup でバックアップのスケジュールや保持ポリシーを定義するための Terraform リソースです。

### 05. resource "aws_backup_plan" "example"
　AWS Backupで自動バックアップを管理するための「バックアッププラン（計画）」を作成・管理するためのリソースです。

### 06. resource "aws_backup_selection" "aurora"
　AWS Backup でどのリソースをバックアップ対象にするか（選択ルール）を定義するための Terraform リソースです。

### 07. resource "aws_sns_topic" "alerts"
　Amazon Web Services (AWS) のSimple Notification Service (SNS) トピックを作成・管理するためのリソースです。

### 08. resource "aws_sns_topic_subscription" "email"
　Amazon SNS（Simple Notification Service）のトピックに対して、通知を受け取るサブスクライバー（配信先）を登録・紐付けるためのリソースです。

