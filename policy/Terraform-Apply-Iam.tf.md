## Terraform-Apply-Iam.tf

```hcl
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "TerraformApplyIam",
            "Effect": "Allow",
            "Action": [
                "iam:GetRole",
                "iam:CreateRole",
                "iam:UpdateRole",
                "iam:DeleteRole",
                "iam:AttachRolePolicy",
                "iam:DetachRolePolicy",
                "iam:PutRolePolicy",
                "iam:GetRolePolicy",
                "iam:DeleteRolePolicy",
                "iam:ListRolePolicies",
                "iam:ListAttachedRolePolicies",
                "iam:TagRole",
                "iam:UntagRole"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformApplyIamLambda",
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "arn:aws:iam::[ユーザID]:role/ai-content-platform-lambda"
        },
        {
            "Sid": "TerraformApplyIamPassRole",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": [
                "arn:aws:iam::[ユーザID]:role/ai-aws-*",
                "arn:aws:iam::[ユーザID]:role/ai-content-platform-ecs-task",
                "arn:aws:iam::[ユーザID]:role/ai-content-platform-ecs-execution",
                "arn:aws:iam::[ユーザID]:role/ai-content-platform-vpc-flowlogs"
            ]
        },
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
        {
            "Sid": "TerraformApplyStepFunctionsPassRole",
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "arn:aws:iam::[ユーザID]:role/ai-content-platform-sfn"
        },
        {
            "Sid": "TerraformApplyBackupPassRole",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::[ユーザID]:role/ai-content-platform-backup-role"
        },
        {
            "Sid": "TerraformListInstanceProfilesForRole",
            "Effect": "Allow",
            "Action": "iam:ListInstanceProfilesForRole",
            "Resource": "arn:aws:iam::[ユーザID]:role/*"
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

### 04. iam:GetRole
AWSのIAMロールに関する設定情報を取得するためのAWS API権限（アクション）です。

### 05. iam:GetRole
AWSのIAMロールに関する設定情報を取得するためのAWS API権限（アクション）です。

### 06. iam:CreateRole
AWS上にIAMロールを新規作成するために必要なAWS APIの権限（IAMポリシーのアクション）を指します。

### 07. iam:UpdateRole
AWSのIAMロールの設定（説明文や最大セッション時間など）を変更・更新するためのAWS APIアクション権限です。

### 08. iam:DeleteRole
AWSのIAM（Identity and Access Management）ロールを削除するためのAWS API権限です。

### 09. iam:AttachRolePolicy
AWS IAM（Identity and Access Management）の権限（APIアクション）の1つで、「特定のIAMロールに、管理ポリシー（AWS管理ポリシーまたはカスタマー管理ポリシー）をアタッチ（紐付け）する権限」を指します。

### 10. iam:DetachRolePolicy
IAMロールからAWS管理ポリシーまたはカスタマー管理ポリシーをデタッチ（解除）するためのAWS IAM APIアクション（権限）です。

### 11. iam:PutRolePolicy
IAMロールに「インラインポリシー」を埋め込む・更新するためのAWS API権限です。

### 12. iam:GetRolePolicy
特定のIAMロールに埋め込まれている「インラインポリシー」の定義ドキュメント（JSON記述）を取得するためのAPIアクション（権限）です。

### 13. iam:DeleteRolePolicy
AWSのIAMロールに直接埋め込まれているインラインポリシーを削除するためのAWS APIアクション（権限）のことです。

### 14. iam:ListRolePolicies
AWS（Amazon Web Services）のIAM（Identity and Access Management）が提供するAPIアクションの一つで、指定したIAMロールに直接埋め込まれているインラインポリシー名のリストを取得するための権限です。

### 15. iam:ListAttachedRolePolicies
AWS IAM（Identity and Access Management）のAPIアクションの1つで、指定したIAMロールにアタッチ（紐付け）されているマネージドポリシーの一覧を取得するための権限です。

### 16. iam:TagRole
TerraformでAWSのIAMロール（aws_iam_role）を作成・更新する際に、ロールへタグを付与・変更するために必要なAWSのIAM権限（APIアクション）です。

### 17. iam:UntagRole
AWSの IAMロールから特定のタグ（メタデータ）を削除する権限 を付与するAWS IAMのAPIアクションです。

### 18. iam:PassRole
IAMユーザーやIAMロール（実行主体）が、AWSサービス（EC2やLambda、ECS、RDSなど）に対して「自分の代わりにこのIAMロールを使ってね」とロールを渡す（引き渡す）ために必要な権限です。

### 19. iam:CreateServiceLinkedRole
AWSの「サービスリンク型ロール（Service-Linked Role）」を新しく作成するために必要なAWS IAMの権限（アクション）です。

### 20. iam:AWSServiceName
「作成または削除しようとしている『サービスリンク型ロール（Service-Linked Role）』が、どのAWSサービスに紐づくものか」を制限・指定するための条件キーです。

### 21. iam:ListInstanceProfilesForRole
特定のIAMロールに関連付けられている（紐づいている）すべての「IAMインスタンスプロファイル」を一覧取得するためのAWS IAM権限（APIアクション）

