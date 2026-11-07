## Trouble on Making_11
---
### Message

Error: creating ELBv2 application Load Balancer (ai-content-platform-alb): operation error Elastic Load Balancing v2: CreateLoadBalancer, https response error StatusCode: 403, RequestID: 4dd9d192-a0bd-4cbb-8ac2-c58429cec8ed, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: iam:CreateServiceLinkedRole on resource: arn:aws:iam::[ユーザID]:role/aws-service-role/elasticloadbalancing.amazonaws.com/AWSServiceRoleForElasticLoadBalancing because no identity-based policy allows the iam:CreateServiceLinkedRole action.

---

###【要約】

エラー: ELBv2 アプリケーションロードバランサー (ai-content-platform-alb) の作成中にエラーが発生しました。

Elastic Load Balancing v2: CreateLoadBalancer の操作において、HTTP 応答エラー (ステータスコード: 403、

リクエストID: 4dd9d192-a0bd-4cbb-8ac2-c58429cec8ed) が返されました。

API エラー「AccessDenied」が発生しています。

ユーザー `arn:aws:iam::[ユーザID]:user/[ユーザ名]` には、リソース `arn:aws:iam::[ユーザID]:role/aws-service-role/elasticloadbalancing.amazonaws.com/AWSServiceRoleForElasticLoadBalancing` に対して 

`iam:CreateServiceLinkedRole` アクションを実行する権限がありません。

これは、`iam:CreateServiceLinkedRole` アクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

アクションを実行する権限が不足している

iam:CreateServiceLinkedRole

---


###【対応方法】

#### 1. 許可ポリシーに以下の記述を追加する

```hcl
        {
            "Sid": "TerraformApplyIamCreateServiceLinkedRole",
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "arn:aws:iam::[ユーザID]:role/aws-service-role/elasticloadbalancing.amazonaws.com/AWSServiceRoleForElasticLoadBalancing",
            "Condition": {
                "StringLike": {
                    "iam:AWSServiceName": "elasticloadbalancing.amazonaws.com"
                }
            }
        },
```


#### 2. プランの再作成

```hcl
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform show -no-color tfplan > plan.txt
terraform apply tfplan
```

