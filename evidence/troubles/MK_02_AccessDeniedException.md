## Trouble on Making_02
---
### Message

Error: creating Backup Vault (ai-content-platform-dev-backup): operation error Backup: CreateBackupVault, https response error StatusCode: 403, RequestID: af806ddd-df36-4de8-9870-25d32f585ab2, api error AccessDeniedException: Insufficient privileges to create a backup vault. Creating a backup vault requires backup-storage and KMS permissions.

---

###【要約】
エラー: バックアップコンテナ (ai-content-platform-dev-backup) の作成に失敗しました。

操作エラー: Backup: CreateBackupVault。HTTP レスポンスエラー (ステータスコード: 403、リクエストID: af806ddd-df36-4de8-9870-25d32f585ab2)。

API エラー: AccessDeniedException (バックアップコンテナを作成するための権限が不足しています。

バックアップコンテナの作成には、backup-storage および KMS に関する権限が必要です)。

---

### 【原因】

単純な

```text
backup:CreateBackupVault
```

だけではなく、

* AWS Backup
* backup-storage
* KMS

の権限が必要です。

AWS公式ドキュメントでも、Backup Vault作成には `backup:CreateBackupVault` に加えて、KMS関連権限と `backup-storage:MountCapsule` が必要とされています。([AWS ドキュメント][3])

---

###【対応方法】

#### 1. 最低限、AWS公式の要件に沿って、

```json
{
  "Effect": "Allow",
  "Action": [
    "backup:CreateBackupVault"
  ],
  "Resource": "arn:aws:backup:[リージョン]:[ユーザID]:backup-vault:*"
},
{
  "Effect": "Allow",
  "Action": [
    "backup-storage:MountCapsule"
  ],
  "Resource": "*"
}
```

さらに、**Terraformの `aws_backup_vault.main` でKMSキーを指定している場合**は、そのKMSキーに対して、

```text
kms:CreateGrant
kms:DescribeKey
kms:RetireGrant
kms:Decrypt
kms:GenerateDataKey
```

が必要です。([AWS ドキュメント][3])

ここは、現在の `governance.tf` の `aws_backup_vault.main` を確認してから最終的なIAMポリシーを決めるのが安全です。


#### 2. 修正後の作業手順

```hcl
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform show -no-color tfplan > plan.txt
terraform apply tfplan
```

