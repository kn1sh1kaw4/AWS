# AI × AWS Content Generation System — Terraform 修正版 v2

AWS ap-northeast-3（大阪）を主リージョンとする、AI×AWSコンテンツ生成基盤のTerraformリファレンス実装です。

## v2で修正した主な点

- HCLのインライン多引数ブロックを通常の複数行ブロックへ修正
- `route` / `health_check` / `condition` / `path_pattern` / `network_configuration` 等のネストブロックを整理
- `variables.tf` / `versions.tf` を標準的なTerraform記法へ整理
- `terraform fmt -recursive` → `terraform init` → `terraform validate` → `terraform plan` の順で確認しやすい構成に整理
- CloudFront用WAFの `us-east-1` provider aliasを明示
- Auroraはデフォルト無効、CloudFrontもデフォルト無効
- Auroraを有効化した場合のDBパスワードは `random_password` + Secrets Manager を使用
- AWS Organizations / Control Towerはアプリケーションスタックから分離
- Route 53 / ACMの「実装済み」表記は削除（v2ではDNS・カスタムドメインリソースをまだ実装していません）

## 構成

- VPC / Public・Private App・Private DB subnet（2 AZ）
- Internet Gateway / NAT Gateway
- ALB
- ECS Fargate Web / API
- Lambda / SQS / DLQ / Step Functions
- S3 / DynamoDB
- Aurora PostgreSQL（任意）
- Secrets Manager（Aurora有効時）
- CloudWatch Logs / Container Insights / VPC Flow Logs
- CloudTrail
- AWS Backup（任意）
- CloudFront + AWS WAF（任意、WAFはus-east-1）
- ECR repository

## 前提

Terraform CLIとAWS CLIを利用できるWindows PowerShell環境を想定しています。
AWS認証は環境変数、AWS CLI profile、IAM Identity Center等、利用環境の安全な方式を使用してください。
アクセスキーや秘密鍵を `terraform.tfvars` やGitへ保存しないでください。

v2ではAWS Provider `~> 6.62` を指定しています。

## 最初の確認手順

```powershell
cd .\ai_aws_iac_apne3
Copy-Item .\terraform.tfvars.example .\terraform.tfvars
terraform fmt -recursive
terraform init
terraform validate
terraform plan -out=tfplan
```

またはPowerShellで一括実行できます。

```powershell
.\run_terraform_check.ps1
```

### planだけ行う場合

```powershell
terraform plan
```

`terraform plan` はAWS APIへ問い合わせるため、AWS認証情報と必要なIAM権限が必要です。

## 低コスト確認用設定

`terraform.tfvars` の初期値では以下を無効化しています。

```hcl
enable_aurora     = false
enable_cloudfront = false
```

NAT Gatewayはprivate ECSから外部へ通信する設計のためデフォルト有効です。NAT Gatewayは料金が発生するため、完全にコストを抑えたい場合は `enable_nat = false` にできます。ただし、その場合private ECSタスクから外部への通信や公開イメージ取得等が成立しない構成になるため、planの検証と実行時の構成意図を分けて考えてください。

## 重要：v2は「構文修正版」であり、本番完成版ではない

今回の目的は、まずTerraformの構文エラーを解消し、`init → validate → plan` まで進められる土台を作ることです。
`plan` が通った後に、AWS上の実際のサービス仕様・IAM最小権限・コスト・可用性・セキュリティ・アプリケーション動作を順番に検証してください。

特に以下は次段階のレビュー対象です。

1. ECSの実コンテナイメージをECRへ置き換える
2. Bedrock権限を対象モデルARN等へ限定する
3. CloudFrontを有効化する場合はCache Policy / Origin Request Policyを含めて見直す
4. Auroraを有効化する場合はDB認証情報の利用経路とSecrets Manager参照権限を実装する
5. AWS Backupの保護対象を必要なリソースへ拡張する
6. Route 53 / ACMによるカスタムドメインを必要に応じて別途追加する
7. Organizations / Control Towerは別のLanding Zone / Governance IaCとして管理する
8. Terraform stateの保存先・ロック・アクセス権を設計する

## CloudFrontについて

現状のCloudFront設定は、既存構成を大きく変えずにplanへ進めるため `forwarded_values` を使用しています。AWS Providerの現行ドキュメントでは `forwarded_values` はdeprecatedで、Cache Policy / Origin Request Policyが推奨されています。plan確認後の改善項目です。

## コスト

これは無料前提のアーキテクチャではありません。NAT Gateway、ALB、CloudTrail、CloudFront/WAF、Aurora等は利用状況に応じて料金が発生します。検証後は必ず `terraform destroy` または個別リソースの削除を検討してください。

## State / Secret

Terraform stateにはリソース属性や、構成によっては機密情報が含まれる可能性があります。stateをGitへコミットしないでください。
Auroraを有効にした場合、生成されたパスワードを含むSecret関連情報がstateに現れる可能性があるため、stateの保護は必須です。
