## Trouble on Making_13
---
### Message

terraform plan -out=tfplanのログで、以下のような記述が見つかった。

```

  # aws_cloudtrail.main is tainted, so must be replaced
-/+ resource "aws_cloudtrail" "main" {
      ~ arn                           = "arn:aws:cloudtrail:[リージョン]:[ユーザID]:trail/ai-content-platform-trail" -> (known after apply)
      ~ home_region                   = "[リージョン]" -> (known after apply)
      ~ id                            = "arn:aws:cloudtrail:[リージョン]:[ユーザID]:trail/ai-content-platform-trail" -> (known after apply)
        name                          = "ai-content-platform-trail"
      + sns_topic_arn                 = (known after apply)
      - tags                          = {} -> null
        # (13 unchanged attributes hidden)
    }
```

---

###【要約】

エラー: Step Functions ステートマシン (ai-content-platform-generation) の作成中にエラーが発生しました。

操作エラー SFN: CreateStateMachine、HTTP レスポンスエラー (ステータスコード: 400、

リクエスト ID: 9f80abda-c765-4903-b943-95330f02fdc2)、API エラー AccessDeniedException: 

ユーザー arn:aws:iam::[ユーザID]:user/[ユーザ名] は、リソース arn:aws:iam::[ユーザID]:role/ai-content-platform-sfn に対して 

iam:PassRole アクションを実行する権限がありません。

これは、iam:PassRole アクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

結論：

過去のTerraform apply中にCloudTrailについて `ListTags` の権限不足が発生し、その途中状態をTerraformがtaintedとして扱った可能性が非常に高い


過程：

1. AWS Cloud Trailの状態を確認する

```terraform
terraform state show aws_cloudtrail.main

# aws_cloudtrail.main:
resource "aws_cloudtrail" "main" {
    arn                           = "arn:aws:cloudtrail:[リージョン]:[ユーザID]:trail/ai-content-platform-trail"
    cloud_watch_logs_group_arn    = null
    cloud_watch_logs_role_arn     = null
    enable_log_file_validation    = true
    enable_logging                = true
    home_region                   = "[リージョン]"
    id                            = "arn:aws:cloudtrail:[リージョン]:[ユーザID]:trail/ai-content-platform-trail"
    include_global_service_events = true
    is_multi_region_trail         = true
    is_organization_trail         = false
    kms_key_id                    = null
    name                          = "ai-content-platform-trail"
    region                        = "[リージョン]"
    s3_bucket_name                = "ai-content-platform-audit-8624ffafdb958ac5bf6f77dee7"
    s3_key_prefix                 = null
    sns_topic_arn                 = null
    sns_topic_name                = null
    tags_all                      = {
        "Environment" = "dev"
        "ManagedBy"   = "Terraform"
        "Project"     = "ai-content-platform"
    }
}

```

2. 1.の確認から、以下のことが判明した。

①**AWS側の実体は正常、Terraform stateとの紐付けも正常です。**

* AWS ARN：`arn:aws:cloudtrail:...:trail/ai-content-platform-trail`
* Terraform state ARN：同一
* AWS側：`IsLogging = true`
* Terraform state：`enable_logging = true`
* S3バケットも一致

したがって、**CloudTrailを破棄・再作成する必要性は現時点では確認できません。**


②AWS側とTerraform stateを比較

AWS側：

```text
TrailARN
arn:aws:cloudtrail:ap-northeast-3:030635937105:trail/ai-content-platform-trail
```

Terraform state：

```text
arn = "arn:aws:cloudtrail:[リージョン]:[ユーザID]:trail/ai-content-platform-trail"
id  = "arn:aws:cloudtrail:[リージョン]:[ユーザID]:trail/ai-content-platform-trail"
```

完全に一致している。

さらに、

```text
AWS:
IsLogging = true

Terraform:
enable_logging = true
```

です。

しかもAWS側では、

```text
LatestDeliveryTime = 2026-10-07T13:49:21...
```

まで更新されています。


③結論

> **CloudTrailは単に「存在する」だけではなく、現在実際にログを記録・配信している**

と判断できます。

したがって、今回の

```text
aws_cloudtrail.main is tainted, so must be replaced
```

は、**CloudTrailそのものが壊れていることを示しているわけではない。**

過去のTerraform apply中にCloudTrailについて `ListTags` の権限不足が発生し、その途中状態をTerraformがtaintedとして扱った可能性が非常に高い。

Terraform公式も、作成処理中のエラーによってリソースがtaintedになり、後続planで置換対象になることを説明しています。正常動作を確認できた場合は `terraform untaint` でtaintを解除できる。([HashiCorp Developer][2])

[2]: https://developer.hashicorp.com/terraform/cli/commands/untaint?utm_source=chatgpt.com "terraform untaint command reference | Terraform | HashiCorp Developer"



---


###【対応方法】

#### 1. 汚染状態を除去する

`terraform untaint` を実施する。

```powershell
terraform untaint aws_cloudtrail.main
```


#### 2. プランの再作成

```hcl
terraform plan -out=tfplan
```


#### 3. aws_cloudtrail.mainが`-/+` から消えているか確認する。

特に、[x] to destroyの [x]が"0"になっていることを確認する。


