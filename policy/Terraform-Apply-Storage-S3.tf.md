## Terraform-Apply-Storage-S3.tf

---

```hcl

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "TerraformApplyStorageS3",
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets",
                "s3:GetBucketLocation",
                "s3:GetBucketVersioning",
                "s3:PutBucketVersioning",
                "s3:GetBucketPublicAccessBlock",
                "s3:PutBucketPublicAccessBlock",
                "s3:GetBucketPolicy",
                "s3:PutBucketPolicy",
                "s3:DeleteBucketPolicy",
                "s3:GetEncryptionConfiguration",
                "s3:PutEncryptionConfiguration",
                "s3:GetBucketTagging",
                "s3:PutBucketTagging",
                "s3:DeleteBucket",
                "s3:CreateBucket",
                "s3:GetBucketAcl",
                "s3:GetBucketWebsite",
                "s3:GetBucketCors",
                "s3:GetBucketRequestPayment",
                "s3:GetBucketLogging",
                "s3:GetLifecycleConfiguration",
                "s3:GetReplicationConfiguration",
                "s3:GetBucketObjectLockConfiguration",
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::ai-content-platform-*"
        },
        {
            "Sid": "TerraformDeleteStorageS3",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucketVersions",
                "s3:DeleteObject",
                "s3:DeleteObjectVersion"
            ],
            "Resource": [
                "arn:aws:s3:::ai-content-platform-audit-[ARN ID]",
                "arn:aws:s3:::ai-content-platform-audit-[ARN ID]/*"
            ]
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

### 04. s3:ListAllMyBuckets
　AWSのIAMポリシーで定義されているアクションの1つで、「AWSアカウント内にあるすべてのS3バケットを一覧表示する権限」を指します。（AWSの内部的には ListBuckets API に対応しています）。

### 05. s3:GetBucketLocation
　AWSのIAMポリシー（アクセス権限）において、指定したAmazon S3バケットがどのAWSリージョン（場所）に存在するかを確認するAPIアクションのことです。

### 06. s3:GetBucketVersioning
　Amazon S3 バケットのバージョニング設定（有効、停止、未設定）の状態を取得することを許可するAWS IAMのアクション権限です。

### 07. s3:PutBucketVersioning
　Amazon S3バケットのバージョニング状態（有効化・停止）を設定・変更するために必要なIAMポリシーのアクション（権限）です。

### 08. s3:GetBucketPublicAccessBlock
　Amazon S3 バケットの「パブリックアクセスブロック（Block Public Access）」の設定情報を読み取るための AWS IAM 権限（アクション）です。

### 09. s3:PutBucketPublicAccessBlock
　Amazon S3 バケットの「ブロックパブリックアクセス (Block Public Access)」を設定・変更するために必要なAWSのIAM権限（アクション）です。

### 09. s3:PutBucketPublicAccessBlock
　S3 バケットの「ブロックパブリックアクセス（Block Public Access）」を設定・変更することを許可（または拒否）する権限（API アクション）です。

### 10. s3:GetBucketPolicy
　AWS IAMポリシーやTerraformの操作において、指定したAmazon S3バケットに設定されている「バケットポリシー」の内容を読み取る（取得する）ために必要な権限およびアクションです。

### 11. s3:PutBucketPolicy
　Amazon S3バケットに対してバケットポリシーを設定・更新するAWSのAPIアクション権限を指します。

### 12. s3:DeleteBucketPolicy
　Amazon S3のバケットポリシーを削除するAPI操作を許可または拒否するためのIAM権限（アクション）です。

### 13. s3:GetEncryptionConfiguration
　Amazon S3 バケットに設定されているサーバー側の暗号化設定（SSE-S3、SSE-KMSなど）の情報を取得するための AWS IAM 権限（アクション）です。

### 14. s3:PutEncryptionConfiguration
　Amazon S3 バケットに対してサーバー側の暗号化設定（SSE）を新しく設定・変更・削除するための AWS IAM ポリシーのアクション（権限）です。

### 15. s3:GetBucketTagging
　Amazon S3 バケットに付与されている「タグ（Tags）」の情報を取得するための AWS IAM アクション権限です。

### 16. s3:PutBucketTagging
　Amazon S3バケットに対して「タグ（Tag）の付与・変更・削除」を行うためのAWS IAMポリシーのアクション権限です。

### 17. s3:DeleteBucket
　AWSのIAM（Identity and Access Management）権限、またはTerraformがAWSのS3バケットを削除する際に実行するAPIアクションのことです。

### 19. s3:GetBucketAcl
　S3バケットのアクセスコントロールリスト（ACL）の設定情報を読み取るためのAWS IAMアクション（権限）です。

### 20. s3:GetBucketCors
　Amazon S3バケットに設定されているCORS（Cross-Origin Resource Sharing / クロスオリジンリソース共有）の設定内容を読み取る（取得する）ためのAWS IAM権限です。

### 21. s3:GetBucketRequestPayment
　Amazon S3 バケットの「リクエスト支払い（Requester Pays）」設定を取得するための AWS IAM ポリシーアクションです。

### 22. s3:GetBucketLogging
　AWSのIAMポリシー（アクセス権限）で定義される Amazon S3のAPIアクション の一つです。

### 23. s3:GetLifecycleConfiguration
　Amazon S3 バケットに設定されているライフサイクルルール（オブジェクトの自動移行や削除のルール）の設定情報を取得するための AWS IAM 権限（アクション）です。

### 24. s3:GetLifecycleConfiguration
　Amazon S3 バケットに設定されているライフサイクルルール（オブジェクトの自動移行や削除のルール）の設定情報を取得するための AWS IAM 権限（アクション）です。

### 25. s3:GetReplicationConfiguration
　Amazon S3バケットに設定されている「レプリケーション（複製）設定」の情報を取得することを許可するAWSのIAMアクション（権限）です。

### 26. s3:GetBucketObjectLockConfiguration
　Amazon S3 バケットに設定されているオブジェクトロック（Object Lock）の設定状況を取得するために必要な IAM パーミッション（権限）です。

### 27. s3:ListBucket
　AWSのS3バケット自体のメタデータ（バケット内に含まれるオブジェクトの一覧）を確認・リスト表示するために必要なIAM権限です。

### 28. s3:ListBucketVersions
　Amazon S3バケット内にあるすべてのオブジェクトの「すべてのバージョン（履歴）」および「削除マーカー」の一覧を取得するためのIAMアクション（権限）です。

### 29. s3:ListBucketVersions
　Amazon S3バケット内にあるすべてのオブジェクトの「すべてのバージョン（履歴）」および「削除マーカー」の一覧を取得するためのIAMアクション（権限）です。

### 30. s3:DeleteObject
　Amazon S3バケット内のファイル（オブジェクト）を削除する権限を定義するAWSのIAMアクション

### 31. s3:DeleteObjectVersion
　バージョニングが有効なS3バケット内の「特定のバージョンを指定してオブジェクトを完全に削除する」ための権限


