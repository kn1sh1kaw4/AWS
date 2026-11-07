## Trouble on Making_14
---
### Message

terraform plan -out=tfplanのログで、以下のような記述が見つかった。

```
  # aws_sns_topic.alerts is tainted, so must be replaced
-/+ resource "aws_sns_topic" "alerts" {
      - application_success_feedback_sample_rate = 0 -> null
      ~ arn                                      = "arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts" -> (known after apply)
      + beginning_archive_time                   = (known after apply)
      + fifo_throughput_scope                    = (known after apply)
      - firehose_success_feedback_sample_rate    = 0 -> null
      - http_success_feedback_sample_rate        = 0 -> null
      ~ id                                       = "arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts" -> (known after apply)
      - lambda_success_feedback_sample_rate      = 0 -> null
        name                                     = "ai-content-platform-dev-alerts"
      + name_prefix                              = (known after apply)
      ~ owner                                    = "[ユーザID]" -> (known after apply)
      ~ policy                                   = jsonencode(
            {
              - Id        = "__default_policy_ID"
              - Statement = [
                  - {
                      - Action    = [
                          - "SNS:GetTopicAttributes",
                          - "SNS:SetTopicAttributes",
                          - "SNS:AddPermission",
                          - "SNS:RemovePermission",
                          - "SNS:DeleteTopic",
                          - "SNS:Subscribe",
                          - "SNS:ListSubscriptionsByTopic",
                          - "SNS:Publish",
                        ]
                      - Condition = {
                          - StringEquals = {
                              - "AWS:SourceOwner" = "[ユーザID]"
                            }
                        }
                      - Effect    = "Allow"
                      - Principal = {
                          - AWS = "*"
                        }
                      - Resource  = "arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts"
                      - Sid       = "__default_statement_ID"
                    },
                ]
              - Version   = "2008-10-17"
            }
        ) -> (known after apply)
      ~ signature_version                        = 0 -> (known after apply)
      - sqs_success_feedback_sample_rate         = 0 -> null
      - tags                                     = {} -> null
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

Terraform state上でtaintedになっているため再作成予定になっている


過程：

1. AWS SNS Topicの状態を確認する

```terraform
terraform state show aws_sns_topic.alerts

# aws_sns_topic.alerts:
resource "aws_sns_topic" "alerts" {
    application_failure_feedback_role_arn    = null
    application_success_feedback_role_arn    = null
    application_success_feedback_sample_rate = 0
    archive_policy                           = null
    arn                                      = "arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts"
    beginning_archive_time                   = null
    content_based_deduplication              = false
    delivery_policy                          = null
    display_name                             = null
    fifo_throughput_scope                    = null
    fifo_topic                               = false
    firehose_failure_feedback_role_arn       = null
    firehose_success_feedback_role_arn       = null
    firehose_success_feedback_sample_rate    = 0
    http_failure_feedback_role_arn           = null
    http_success_feedback_role_arn           = null
    http_success_feedback_sample_rate        = 0
    id                                       = "arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts"
    kms_master_key_id                        = null
    lambda_failure_feedback_role_arn         = null
    lambda_success_feedback_role_arn         = null
    lambda_success_feedback_sample_rate      = 0
    name                                     = "ai-content-platform-dev-alerts"
    name_prefix                              = null
    owner                                    = "[ユーザID]"
    policy                                   = jsonencode(
        {
            Id        = "__default_policy_ID"
            Statement = [
                {
                    Action    = [
                        "SNS:GetTopicAttributes",
                        "SNS:SetTopicAttributes",
                        "SNS:AddPermission",
                        "SNS:RemovePermission",
                        "SNS:DeleteTopic",
                        "SNS:Subscribe",
                        "SNS:ListSubscriptionsByTopic",
                        "SNS:Publish",
                    ]
                    Condition = {
                        StringEquals = {
                            "AWS:SourceOwner" = "[ユーザID]"
                        }
                    }
                    Effect    = "Allow"
                    Principal = {
                        AWS = "*"
                    }
                    Resource  = "arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts"
                    Sid       = "__default_statement_ID"
                },
            ]
            Version   = "2008-10-17"
        }
    )
    region                                   = "[リージョン]"
    signature_version                        = 0
    sqs_failure_feedback_role_arn            = null
    sqs_success_feedback_role_arn            = null
    sqs_success_feedback_sample_rate         = 0
    tags_all                                 = {
        "Environment" = "dev"
        "ManagedBy"   = "Terraform"
        "Project"     = "ai-content-platform"
    }
    tracing_config                           = null
}
```

2. 1.の確認から、以下のことが判明した。

```text
AWS:
arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts
```

Terraform state：

```text
arn = "arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts"
id  = "arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts"
```

で一致しています。

さらに、

```text
Owner = [ユーザID]
SubscriptionsPending = 0
SubscriptionsConfirmed = 0
```

など、Topic自体が正常に取得できています。

したがって、

```text
aws_sns_topic.alerts is tainted, so must be replaced
```

についても、

**「SNS Topicが壊れているから再作成が必要」ではなく、「Terraform state上でtaintedになっているため再作成予定になっている」**

と考えるのが妥当です。

---

# 3. `tags = {} -> null` について

ここも心配になるところだと思われる。

以前のplanでは、

```text
- tags = {} -> null
```

などが出ています。

しかし今回のstateを見ると、

```text
tags_all = {
    "Environment" = "dev"
    "ManagedBy"   = "Terraform"
    "Project"     = "ai-content-platform"
}
```

となっています。

これは**今回の置換を正当化するような重大な設定不整合とは判断しません。**

むしろ今回の `-/+` の直接的な理由は、

```text
is tainted, so must be replaced
```

です。


③結論

現段階で `tags` のためにTerraformコードを変更する必要はありません。

と判断できます。

したがって、今回の

```text
aws_sns_topic.alerts is tainted, so must be replaced
```

は、**SNSそのものが壊れていることを示しているわけではない。**

---


###【対応方法】

#### 1. 汚染状態を除去する

`terraform untaint` を実施する。

```powershell
terraform untaint aws_sns_topic.alerts
```


#### 2. プランの再作成

```hcl
terraform plan -out=tfplan
```


#### 3. aws_sns_topic.alertsが`-/+` から消えているか確認する。

特に、[x] to destroyの [x]が"0"になっていることを確認する。


