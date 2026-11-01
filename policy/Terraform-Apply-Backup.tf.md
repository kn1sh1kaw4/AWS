## Terraform-Apply-Backup.tf

---

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "TerraformApplyBackup",
            "Effect": "Allow",
            "Action": [
                "backup:*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformApplyBackupVaultCreate",
            "Effect": "Allow",
            "Action": [
                "backup:CreateBackupVault"
            ],
            "Resource": "arn:aws:backup:[リージョン]:[アカウントID]:backup-vault:*"
        },
        {
            "Sid": "TerraformApplyBackupStorage",
            "Effect": "Allow",
            "Action": [
                "backup-storage:MountCapsule"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformApplyBackupKms",
            "Effect": "Allow",
            "Action": [
                "kms:CreateGrant",
                "kms:DescribeKey",
                "kms:RetireGrant",
                "kms:Decrypt",
                "kms:GenerateDataKey"
            ],
            "Resource": "arn:aws:kms:[リージョンID]:[アカウントID]:key/[キー]"
        }
    ]
}
```
### 01. "Version": "2012-10-17"
現在標準で使われているIAMポリシーの文法・スキーマのバージョン番号を示します。

### 02. "Effect": "Allow"
TerraformでAWSのIAMポリシーや信頼ポリシーを記述する際、"Effect": "Allow"は許可（アクセスを許可する）を指定するJSON要素です。

### 03. "Statement" : [...]
アクセス権限のルール（許可・拒否）を記述するために使用するブロック

### 04. "Action": [ "backup:*" ]
AWS Backup サービスに関するすべての操作（アクション）を許可または拒否する設定です。

### 05. "Resource": "*"
アクセス許可の対象（リソース）を「すべてのリソース（全範囲）」に指定するという意味の記述です。

### 06. "Action": [ "backup:CreateBackupVault" ]
AWS Backup において「新しいバックアップボルト（保管庫）を作成する権限」を指定するアクションです。

### 07. "Resource": "arn:aws:backup:[リージョン]:[アカウントID]:backup-vault:*"
WS Backup のバックアップボルト（Backup Vault）を指す識別子です。

### 08. "backup-storage:MountCapsule"
AWS Backup でバックアップボールト（Backup Vault）を作成・管理する際に必要となる AWS IAM のアクション（権限）です。

### 09. "kms:CreateGrant"
AWSの暗号化キー管理サービスである AWS KMS（Key Management Service） において、他のAWSサービスや別のアカウントに対して、KMSキーを一時的に操作・利用する権限（グラント：Grant）を発行するためのAPI権限です。

### 10. kms:DescribeKey
AWSの暗号化キー管理サービスである AWS Key Management Service (AWS KMS) の権限（アクション）の一つです。

### 11. kms:DescribeKey
作成された「グラント（Grant/権限委任）」を自ら、または他者が破棄（終了）する際に必要な権限です。

### 12. kms:RetireGrant
AWS KMS のアクセス権限（グラント）を削除・抹消する際のアプローチを指定するもの。

### 13. kms:Decrypt
AWSなどのクラウドサービスで暗号化された機密情報（パスワードやAPIキーなど）をTerraformの実行時に復号化してリソースに渡す仕組みを指す。

### 14. kms:GenerateDataKey
データ暗号化に使用する「データキー（対称鍵）」を生成するためのAPIおよびIAMアクション。

