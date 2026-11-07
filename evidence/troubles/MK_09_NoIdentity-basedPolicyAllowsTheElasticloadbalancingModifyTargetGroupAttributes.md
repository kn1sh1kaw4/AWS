## Trouble on Making_09
---
### Message

Error: modifying ELBv2 Target Group (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17) attributes: operation error Elastic Load Balancing v2: ModifyTargetGroupAttributes, https response error StatusCode: 403, RequestID: 8e442cf9-0968-48e3-a315-211600ec266a, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: elasticloadbalancing:ModifyTargetGroupAttributes on resource: arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17 because no identity-based policy allows the elasticloadbalancing:ModifyTargetGroupAttributes action

---

###【要約】

エラー: ELBv2 ターゲットグループ (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17) の属性を変更しようとした際にエラーが発生しました。

Elastic Load Balancing v2: ModifyTargetGroupAttributes 操作において、HTTP 応答エラー (ステータスコード: 403、リクエストID: 8e442cf9-0968-48e3-a315-211600ec266a) が返されました。

API エラー AccessDenied: ユーザー (arn:aws:iam::[ユーザID]:user/[ユーザ名]) には、リソース (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17) に対して 

`elasticloadbalancing:ModifyTargetGroupAttributes` アクションを実行する権限がありません。

これは、このアクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

アクションを実行する権限が不足している

elasticloadbalancing:ModifyTargetGroupAttributes

---


###【対応方法】

#### 1. 許可ポリシーに以下の記述を追加する

```hcl
        {
            "Sid": "TerraformApplyElbv2Read",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:ModifyTargetGroupAttributes"
            ],
            "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/*"
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

