## versions.tf
---
```hcl
terraform {
  required_version = ">= 1.6.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.62"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.7"
    }
  }
}

provider "aws" {
  region = var.region

  default_tags {
    tags = {
      Project     = var.project
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

# CloudFront-scoped WAF resources must use us-east-1.
provider "aws" {
  alias  = "use1"
  region = "us-east-1"

  default_tags {
    tags = {
      Project     = var.project
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}
```

### 01. terraform {...}
Terraform自身やプロジェクト全体の動作環境（使用するTerraformのバージョン、利用するプロバイダー、状態ファイル（State）の保存場所など）を定義するためのメタ設定ブロックです。
実際のインフラ（AWSのEC2やネットワーク等）を直接作るものではなく、「このコードを実行するためにTerraform側に要求するルール」を指定します。

### 02. provider "aws"
Terraformの記述（HCL）をAWSのAPIコールに変換し、AWS上のインフラストラクチャ（EC2、S3、VPCなど）の作成・変更・削除を自動化するための公式プラグインです。
Terraform本体はクラウドサービスの操作手順を直接持っていないため、プロバイダーが「ドライバー」や「通訳」の役割を果たします。

### 03. provider "aws" {...}
TerraformがAmazon Web Services (AWS) のAPIと通信してリソースを操作（作成・変更・削除）するための接続設定を行う記述です。
Terraform本体はクラウドサービスの操作方法を直接知らないため、AWS Providerという専用のプラグイン（ドライバー）を介してAWSを制御します。

