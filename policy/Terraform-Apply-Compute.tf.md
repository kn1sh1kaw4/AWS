## Terraform-Apply-Compute.tf

```hcl
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "TerraformApplyComputeEcs",
            "Effect": "Allow",
            "Action": [
                "ecs:CreateCluster",
                "ecs:DescribeClusters",
                "ecs:UpdateCluster",
                "ecs:DeleteCluster",
                "ecs:DeregisterTaskDefinition",
                "ecs:TagResource"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformApplyComputeEcr",
            "Effect": "Allow",
            "Action": [
                "ecr:CreateRepository",
                "ecr:DescribeRepositories",
                "ecr:GetRepositoryPolicy",
                "ecr:SetRepositoryPolicy",
                "ecr:DeleteRepository",
                "ecr:ListTagsForResource",
                "ecr:TagResource"
            ],
            "Resource": "arn:aws:ecr:[リージョン]:[ユーザID]:repository/ai-content-platform-*"
        },
        {
            "Sid": "TerraformApplyComputeLambda",
            "Effect": "Allow",
            "Action": [
                "lambda:Get*",
                "lambda:List*",
                "lambda:CreateFunction",
                "lambda:UpdateFunctionCode",
                "lambda:UpdateFunctionConfiguration",
                "lambda:DeleteFunction",
                "lambda:CreateEventSourceMapping",
                "lambda:UpdateEventSourceMapping",
                "lambda:DeleteEventSourceMapping",
                "lambda:AddPermission",
                "lambda:RemovePermission",
                "lambda:TagResource",
                "lambda:UntagResource"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformApplyComputeStates",
            "Effect": "Allow",
            "Action": [
                "states:Describe*",
                "states:List*",
                "states:CreateStateMachine",
                "states:UpdateStateMachine",
                "states:DeleteStateMachine",
                "states:TagResource",
                "states:UntagResource"
            ],
            "Resource": "*"
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

### 04. ecs:CreateCluster
AWSのコンテナオーケストレーションサービスである Amazon ECS のクラスターを作成するための AWS IAM アクション（権限）です。

### 05. ecs:DescribeClusters
AWS IAM（Identity and Access Management）のポリシーで定義されるAmazon ECSのアクション（権限）です。

### 06. ecs:UpdateCluster
aws_ecs_cluster リソースの設定変更（更新）を行う際に、Terraform の実行 IAM ロール（またはユーザー）に必要となる AWS の権限（API） です。

### 07. ecs:DeleteCluster
AWSのAmazon ECS（Elastic Container Service）でクラスターを削除するために必要なAWS IAMのアクション（権限）です。

### 08. ecs:DeregisterTaskDefinition
削除時に自動的にAWS側で「Deregister（アクティブ状態から非アクティブ状態へ変更）」される仕組み

### 09. ecs:TagResource
AWSのIAMポリシー（権限）の1アクションを指します。

### 10. ecr:CreateRepository
AWSのコンテナレジストリ（Amazon ECR）に新しいリポジトリを作成するためのIAM権限（アクション）を指します。

### 11. ecr:DescribeRepositories
Amazon Elastic Container Registry（ECR）のレジストリ内にあるコンテナリポジトリの情報を取得するための AWS IAM 権限（および API アクション）

### 12. ecr:GetRepositoryPolicy
厳密にはTerraformの独自リソースやプロパティではなく、AWSのIAM（Identity and Access Management）アクション（権限）の1つです。

### 13. ecr:SetRepositoryPolicy
Amazon ECR（Elastic Container Registry）のリポジトリポリシー（アクセス権限）を設定・更新するために必要なAWS IAMのアクション（権限）です。

### 14. ecr:DeleteRepository
Amazon ECR（Elastic Container Registry）のリポジトリを削除するアクション（権限）を指します。

### 15. ecr:DeleteRepository
AWSのマネージドコンテナレジストリである Amazon ECR リポジトリに付与されている「メタデータタグ（リソースタグ）」の一覧を取得するためのIAMポリシーのアクション（権限）です。

### 16. ecr:ListTagsForResource
AWSのマネージドコンテナレジストリである Amazon ECR リポジトリに付与されている「メタデータタグ（リソースタグ）」の一覧を取得するためのIAMポリシーのアクション（権限）です。

### 17. ecr:TagResource
AWSの Amazon ECR（Elastic Container Registry）リポジトリに対してタグ（メタデータ）を付与・変更する権限（AWS IAMのアクション）を指します。

### 18. lambda:Get*
AWS Lambda の「取得・読み取り系アクション」のすべてを許可するワイルドカード指定です。

### 19. lambda:List*
AWS Lambdaの情報を「一覧で取得（リスト表示）」する操作すべてを許可するワイルドカード権限です。

### 20. lambda:CreateFunction
AWS Lambda 関数を新規に作成するために必要な AWS IAM（Identity and Access Management）のアクション（権限）です。

### 21. lambda:UpdateFunctionCode
AWS Lambda 関数のコード（関数の本体やコンテナイメージ）を更新・デプロイするために必要な権限です。

### 22. lambda:UpdateFunctionConfiguration
AWS Lambda関数のコード以外の設定情報（環境変数、メモリサイズ、タイムアウト時間、VPC設定、ランタイム、IAMロールなど）を更新するための AWS IAMアクション（API操作） です。

### 23. lambda:DeleteFunction
Terraform（を実行するIAMユーザーやロール）がAWSのLambda関数を削除（terraform destroy やリソースの定義削除）するために必要な「AWS IAMの権限（アクション）」です。

### 24. lambda:CreateEventSourceMapping
AWS Lambda 関数とストリーム系やキュー系のイベントソース（Amazon SQS、Kinesis、DynamoDB Streams、Amazon MQ、MSK / Apache Kafkaなど）を接続し、自動的にLambdaをポーリング・起動させるための設定を管理するリソースです。

### 25. lambda:UpdateEventSourceMapping
AWS Lambdaのイベントソースマッピング（SQS、DynamoDB Streams、Kinesisなどからデータを読み取ってLambdaを起動する設定）を更新・変更する際に必要なIAMポリシーのアクション（権限）を指します。

### 26. lambda:DeleteEventSourceMapping
AWS Lambdaのイベントソースマッピング（SQS、DynamoDB、Kinesisなどとの連携設定）を削除するために必要なAWS IAMポリシーのアクション（権限）です。

### 27. lambda:AddPermission
AWS Lambda 関数に対してリソースベースポリシー（他のサービスやアカウントからの呼び出し許可）を追加する権限・API アクションを指します。

### 28. lambda:AddPermission
lambda:RemovePermission は、AWS Lambdaの「リソースベースのポリシー」から、特定のトリガー（API Gateway、EventBridge、S3など）の呼び出し権限を削除する権限（APIアクション）を意味します。

### 29. lambda:RemovePermission
AWS Lambdaの「リソースベースのポリシー」から、特定のトリガー（API Gateway、EventBridge、S3など）の呼び出し権限を削除する権限（APIアクション）を意味します。

### 30. lambda:TagResource
AWS Lambda 関数などのリソースにタグ（管理用のキーと値のペア）を付与・変更する際、裏側で呼び出される AWS の API アクション（および IAM 権限）です。

### 31. lambda:UntagResource
Terraformを実行するIAMユーザーやIAMロール（実行アイデンティティ）に対して必要となる、AWS Lambda関数から特定のタグを削除するためのAWS IAMアクション（権限）です。

### 32. states:Describe*
APIや、各種CI/CD・連携ツールのIAMポリシー（またはそれに類する権限管理）における「State（状態ファイル）の詳細情報を取得・閲覧するための権限定義」を指しています。

### 33. states:List*
Terraformのステートファイル（tfstate）に記録されている管理下のリソースの一覧（リソースアドレス）を表示するためのCLIコマンドです。

### 34. states:CreateStateMachine
AWS Step Functionsのステートマシン（ワークフロー）を新しく作成するために必要なIAM権限（アクション）のことです。

### 35. states:UpdateStateMachine
AWS Step Functions のステートマシン（ワークフロー定義や設定）を更新する際に呼び出される AWS IAM の API アクション（権限）です。

### 36. states:UpdateStateMachine
AWS Step Functions のステートマシン（ワークフロー定義や設定）を更新する際に呼び出される AWS IAM の API アクション（権限）です。

### 37. states:DeleteStateMachine
AWS Step Functionsのステートマシンを削除するために必要なAWS IAMのアクション（権限）です。

### 38. states:TagResource
Terraform自体の機能や構文ではなく、AWS Step Functions（サービスプレフィックスが states）のAPIアクション（IAM権限）のことです。

### 39. states:UntagResource
AWS API（またはAWSプロバイダー経由の処理）における UntagResource API、またはIAMポリシー等で指定される操作名（例：kms:UntagResource、lambda:UntagResource など）を指しています。

