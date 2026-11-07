## Trouble on Making_09
---
### Message

Error: creating Backup Vault (ai-content-platform-dev-backup): operation error Backup: CreateBackupVault, https response error StatusCode: 403, RequestID: e0c20fb1-45c5-40d8-934f-715da21600ea, api error AccessDeniedException: Insufficient privileges to create a backup vault. Creating a backup vault requires backup-storage and KMS permissions.

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

