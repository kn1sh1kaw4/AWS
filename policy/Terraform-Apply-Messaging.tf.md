## Terraform-Apply-Messaging.tf

```hcl
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "TerraformApplyMessagingSqs",
            "Effect": "Allow",
            "Action": [
                "sqs:CreateQueue",
                "sqs:GetQueueAttributes",
                "sqs:SetQueueAttributes",
                "sqs:DeleteQueue",
                "sqs:ListQueueTags",
                "sqs:TagQueue"
            ],
            "Resource": "arn:aws:sqs:[リージョン]:[ユーザID]:ai-content-platform-*"
        },
        {
            "Sid": "TerraformApplyMessagingSns",
            "Effect": "Allow",
            "Action": [
                "sns:CreateTopic",
                "sns:GetTopicAttributes",
                "sns:SetTopicAttributes",
                "sns:TagResource",
                "sns:DeleteTopic"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformApplyMessagingSns2",
            "Effect": "Allow",
            "Action": [
                "sns:ListTagsForResource"
            ],
            "Resource": "arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts"
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

### 04. sqs:CreateQueue
AWSのメッセージキューイングサービスであるAmazon SQSで「新しくキューを作成する権限」を指すAWSのIAMアクション（ポリシーの要素）です。

### 05. sqs:GetQueueAttributes
AWSのAmazon Simple Queue Service（SQS）キューが持つ各種属性（メッセージ数、可視性タイムアウト、保持期間、暗号化設定など）を読み取る（取得する）権限を示すIAMアクションです。

### 06. sqs:SetQueueAttributes
Amazon SQS（Simple Queue Service）のキュー属性（設定値）を変更・更新するために使用される AWS IAMポリシーのアクション（権限） です。

### 07. sqs:DeleteQueue
Amazon SQS（Simple Queue Service）のメッセージキュー自体を削除する権限を指すAWSのIAMアクションです。

### 08. sqs:ListQueueTags
Amazon SQS キューに付与されているすべてのコスト配分タグ（メタデータ）を一覧取得するための AWS IAM アクション（権限）です。

### 09. sqs:TagQueue
AWSのIAMポリシー（アクセス権限）を定義する際に使用される Amazon SQSのAPIアクション（権限） の一つです。

### 10. sns:CreateTopic
AWS のリクエストや IAM ポリシー内で、「Amazon SNS（Simple Notification Service）のトピックを作成する」というアクション（権限）を表す識別子です。

### 11. sns:GetTopicAttributes
Amazon Simple Notification Service (Amazon SNS) において、特定のトピックの属性情報（ARN、ポリシー、サブスクライバーのリスト、配信設定など）を取得する権限を表す IAM アクションです。

### 12. sns:SetTopicAttributes
Amazon SNS (Simple Notification Service) のトピックポリシー（アクセス制御設定）でトピックの属性を変更する権限を付与する際のアクション名です。

### 13. sns:TagResource
Amazon SNS（Simple Notification Service）のトピックに対してタグを付与・管理するAPI操作（アクション）を指します。

### 14. sns:DeleteTopic
指定したトピックを削除する権限を許可（Allow）または拒否（Deny）するためのアクション名です。

### 15. sns:ListTagsForResource
Amazon SNS トピックに設定されているタグ（リソースタグ）の一覧を取得するための AWS IAM アクション（権限）です。

