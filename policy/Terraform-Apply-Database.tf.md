## Terraform-Apply-Database.tf

---

```hcl
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "TerraformApplyDatabase",
            "Effect": "Allow",
            "Action": [
                "dynamodb:DescribeTable",
                "dynamodb:CreateTable",
                "dynamodb:UpdateTable",
                "dynamodb:DeleteTable",
                "dynamodb:ListTagsOfResource",
                "dynamodb:TagResource",
                "dynamodb:UntagResource",
                "dynamodb:DescribeContinuousBackups",
                "dynamodb:DescribeTimeToLive",
                "dynamodb:UpdateContinuousBackups"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformApplyRds",
            "Effect": "Allow",
            "Action": [
                "rds:Describe*",
                "rds:CreateDBSubnetGroup",
                "rds:ModifyDBSubnetGroup",
                "rds:DeleteDBSubnetGroup",
                "rds:CreateDBCluster",
                "rds:ModifyDBCluster",
                "rds:DeleteDBCluster",
                "rds:AddTagsToResource",
                "rds:CreateDBInstance",
                "rds:ModifyDBInstance",
                "rds:DeleteDBInstance",
                "rds:CreateDBClusterParameterGroup",
                "rds:RemoveTagsFromResource",
                "cloudwatch:PutMetricAlarm",
                "cloudwatch:DescribeAlarms",
                "cloudwatch:DeleteAlarms"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformApplySecretsManager",
            "Effect": "Allow",
            "Action": [
                "secretsmanager:DescribeSecret",
                "secretsmanager:GetSecretValue",
                "secretsmanager:CreateSecret",
                "secretsmanager:UpdateSecret",
                "secretsmanager:DeleteSecret",
                "secretsmanager:PutSecretValue",
                "secretsmanager:TagResource",
                "secretsmanager:UntagResource"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "*",
            "Condition": {
                "StringLike": {
                    "iam:PassedToService": "rds.amazonaws.com"
                }
            }
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

### 04. dynamodb:DescribeTable
Amazon DynamoDB テーブルの構造や状態（スキーマ、プロビジョニング設定、ステータスなど）を取得するための AWS IAM 権限です。

### 05. dynamodb:CreateTable
Terraformを実行するIAMユーザーやロール（またはアプリケーション）に許可する必要がある「AWS IAMポリシーのアクション（権限）」を指します。

### 06. dynamodb:CreateTable
Terraformを実行するIAMユーザーやロール（またはアプリケーション）に許可する必要がある「AWS IAMポリシーのアクション（権限）」を指します。

### 07. dynamodb:UpdateTable
AWSのアイデンティティとアクセス管理（IAM）ポリシーで定義される「Amazon DynamoDBテーブルの設定を変更・更新する権限（APIアクション）」です。

### 08. dynamodb:DeleteTable
Amazon DynamoDB のテーブルを完全に削除するための IAM ポリシーの権限（アクション）です。

### 09. dynamodb:ListTagsOfResource
AWSのIAMポリシー（アクセス権限設定）で定義されるアクション（API操作の権限）の1つです。

### 10. dynamodb:TagResource
AWS DynamoDBのリソース（テーブルやバックアップなど）にタグを追加・変更する際に必要なAWS IAMのポリシー権限（アクション）です。

### 11. dynamodb:UntagResource
Amazon DynamoDB のテーブルやバックアップなどのリソースから割り当てられているタグ（Tag）を削除するための AWS IAM 権限（アクション）です。

### 12. dynamodb:DescribeContinuousBackups
AWSの Amazon DynamoDB における 継続的バックアップ（ポイントインタイムリカバリ：PITR）の設定やステータスを確認するためのIAM権限（APIアクション） です。

### 13. dynamodb:DescribeTimeToLive
Amazon DynamoDB テーブルの Time to Live（TTL）の設定状況を確認（読み取り）するための IAM アクションです。

### 14. dynamodb:UpdateContinuousBackups
Amazon DynamoDB テーブルの 継続的バックアップ（Point-in-Time Recovery: PITR / ポイントインタイムリカバリ） の有効化・無効化を変更するために必要な権限を指します。

### 15. rds:Describe*
AWS IAMポリシー内で使用される、Amazon RDS（Relational Database Service）の情報取得・参照系API操作（Describe）をまとめて許可・拒否するためのワイルドカード表記です。

### 16. rds:CreateDBSubnetGroup
Amazon RDS で「DBサブネットグループ」を新規作成するためのAWS APIアクション（権限）です。

### 17. rds:ModifyDBSubnetGroup
AWSのIAMポリシー（権限管理）で定義されるAmazon RDSのAPIアクション（操作権限）のことです。

### 18. rds:DeleteDBSubnetGroup
AWS RDSの「DBサブネットグループ」を削除するために必要なAWS IAMの権限（アクション）です。

### 19. rds:CreateDBCluster
主にAWSのIAMポリシー（権限設定）内で使用される「Amazon RDS の DB クラスターを作成する権限」を示すAPIアクションです。

### 20. rds:ModifyDBCluster
Amazon RDS（または Aurora）のDBクラスターの設定を変更するAWS APIアクション（権限）を指します。

### 21. rds:DeleteDBCluster
Amazon RDS や Amazon Aurora などの DB クラスターを削除する際に必要となる AWS の IAM アクション（権限）です。

### 22. rds:AddTagsToResource
Amazon RDSリソースにメタデータタグ（キーと値のペア）を追加・付与するためのAWS IAMアクション（権限）です。

### 23. rds:CreateDBInstance
Terraform自体の構文ではなく、AWSのIAMポリシー（権限）で定義されているAmazon RDSのAPIアクション名です。

### 24. rds:ModifyDBInstance
Amazon RDSのインスタンス設定を変更する際にバックエンド（AWS API）側で呼び出されるIAMアクション（権限）のことです。

### 25. rds:DeleteDBInstance
AWSのRelational Database Service (RDS) インスタンスを削除する権限（IAMアクション）を指します。

### 26. rds:AddTagsToResource
TerraformがAWSのRDSリソースに対してタグを追加・更新する際に、内部のAWS API（またはIAMポリシー）で必要となる権限（アクション）です。

### 27. rds:AddTagsToResource
AWSのIAMポリシー内で指定するアクション（API権限）であり、TerraformがAWS上にAmazon RDSインスタンスを新規作成・プロビジョニングする際に内部で呼び出す権限を指します。

### 28. rds:CreateDBInstance
Terraform自体が持つ構文ではなく、Terraformが裏側でAWSを操作する際に呼び出すAWSのAPIアクション（権限）です。

### 29. rds:ModifyDBInstance
Terraform自身の設定項目ではなく、Amazon RDSのDBインスタンスを変更するために必要なAWSのIAM権限（アクション）です。

### 30. rds:DeleteDBInstance
Amazon RDS（Relational Database Service）のデータベースインスタンスを削除する権限を制御するアクションです。

### 31. rds:CreateDBClusterParameterGroup
AWSのRDS（主にAmazon Aurora）のクラスターパラメータグループを作成するAPIアクション（IAM権限）のことです。

### 32. rds:RemoveTagsFromResource
Amazon RDS（Relational Database Service）のリソースからメタデータタグを削除する権限（IAMアクション）、およびTerraformが内部的に実行するAWS API操作のことです。

### 33. cloudwatch:PutMetricAlarm
AWSリソースのアクセス権限（IAM）におけるCloudWatchアラームを作成・更新するためのAPIアクションを指します。一方、TerraformでCloudWatchアラームをコード管理する際は、リソース定義として aws_cloudwatch_metric_alarm を使用します。

### 34. cloudwatch:PutMetricAlarm
cloudwatch:DescribeAlarms は、AWSのAmazon CloudWatchアラームの情報を取得・参照するために、IAMポリシー（aws_iam_policy_document など）内で定義されるAWSの権限（アクション）です。

### 35. cloudwatch:DescribeAlarms
Amazon CloudWatch のアラーム情報を取得（表示）するための IAM 権限です。

### 36. cloudwatch:DescribeAlarms
Amazon CloudWatch のアラーム情報を取得（一覧表示・詳細参照）するための権限（アクション）を指します。

### 37. cloudwatch:DeleteAlarms
Amazon CloudWatchのアラームを削除するためのIAMアクセス許可（アクション）です。

### 38. secretsmanager:DescribeSecret
指定したシークレット（秘密情報）のメタデータ（管理情報）を取得するための IAM 権限（API アクション）です。

### 39. secretsmanager:GetSecretValue
AWS Secrets Managerに保存されているシークレットの具体的な値（パスワードやAPIキーなどの文字列・バイナリ）を取得するために必要なAWS IAMのアクション（権限）です。

### 40. secretsmanager:CreateSecret
AWS Secrets Manager で新しいシークレット（秘密情報）を作成するために必要な IAM ポリシーのアクション（権限）です。

### 41. secretsmanager:UpdateSecret
AWS Secrets Manager に登録されているシークレットのメタデータ（説明、暗号化キー、タグ、ローテーション設定など）や、シークレットの値を変更するための AWS IAM アクション権限です。

### 42. secretsmanager:DeleteSecret
AWS IAMポリシーやAWS Secrets Managerの「リソースポリシー」内で使用される AWSのAPIアクション（権限） です。

### 43. secretsmanager:PutSecretValue
AWS Secrets Manager のシークレット（文字列やバイナリ）の新しいバージョンを作成し、値を書き込む（更新または初期設定する）ための IAM 権限（アクション）です。

### 44. secretsmanager:TagResource
AWS Secrets Manager のシークレット（秘密情報）に対してメタデータ（タグ）を付与・変更する権限（IAMアクション）を指します。

### 45. secretsmanager:TagResource
AWS Secrets Managerに保存されているシークレットから、特定のタグ（メタデータ）を削除する操作を実行するためのIAM権限（アクション）です。

### 46. secretsmanager:UntagResource
AWS Secrets Managerに保存されているシークレットから、特定のタグ（メタデータ）を削除する操作を実行するためのIAM権限（アクション）です。

### 47. iam:PassRole
実行ユーザー（またはTerraformを実行しているIAMロール）が、作成・起動するAWSリソース（EC2、Lambda、ECSなど）に「IAMロールを割り当てる（渡す）」ために必要な権限のことです。

### 48. Condition
主に「変数のバリデーション（入力値チェック）」、「リソースのライフサイクル制御（precondition / postcondition）」、そして「条件分岐（三項演算子）」の3つの文脈で使われます。

### 49. StringLike
AWSのIAMポリシー（主に信頼ポリシーやIAMロールのConditionなど）を記述する際に使用される文字列比較の演算子（ワイルドカードが使える条件句）です。

### 50. StringLike
iam:PassRole 権限とセットで使用する「条件キー（Condition Key）」です。

