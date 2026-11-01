## Terraform-Apply-Network.tf

```hcl
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "TerraformApplyNetwork",
            "Effect": "Allow",
            "Action": [
                "ec2:Describe*",
                "ec2:CreateVpc",
                "ec2:DeleteVpc",
                "ec2:ModifyVpcAttribute",
                "ec2:CreateSubnet",
                "ec2:DeleteSubnet",
                "ec2:ModifySubnetAttribute",
                "ec2:CreateRouteTable",
                "ec2:DeleteRouteTable",
                "ec2:CreateRoute",
                "ec2:ReplaceRoute",
                "ec2:DeleteRoute",
                "ec2:AssociateRouteTable",
                "ec2:DisassociateRouteTable",
                "ec2:CreateInternetGateway",
                "ec2:AttachInternetGateway",
                "ec2:DetachInternetGateway",
                "ec2:DeleteInternetGateway",
                "ec2:AllocateAddress",
                "ec2:ReleaseAddress",
                "ec2:CreateNatGateway",
                "ec2:DeleteNatGateway",
                "ec2:CreateSecurityGroup",
                "ec2:DeleteSecurityGroup",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:AuthorizeSecurityGroupEgress",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:RevokeSecurityGroupEgress",
                "ec2:CreateFlowLogs",
                "ec2:DeleteFlowLogs",
                "ec2:CreateTags",
                "ec2:DisassociateAddress",
                "route53:CreateHostedZone",
                "route53:ChangeResourceRecordSets",
                "route53:GetHostedZone",
                "route53:ListHostedZones",
                "route53:CreateHealthCheck"
            ],
            "Resource": "*"
        },
        {
            "Sid": "GetSecurityGroupsForVpc",
            "Effect": "Allow",
            "Action": "ec2:GetSecurityGroupsForVpc",
            "Resource": "arn:aws:ec2:[リージョン]:[ユーザID]:vpc/vpc-[vpcID]"
        },
        {
            "Sid": "TerraformApplyElbv2Read",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:ModifyTargetGroupAttributes"
            ],
            "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/*"
        },
        {
            "Sid": "CreateApiTargetGroup",
            "Effect": "Allow",
            "Action": "elasticloadbalancing:CreateTargetGroup",
            "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-api-tg/*"
        },
        {
            "Sid": "TerraformApplyModifyTargetGroupAttributes",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticloadbalancing:DescribeTargetGroups",
                "elasticloadbalancing:DescribeTags",
                "elasticloadbalancing:DescribeTargetGroupAttributes"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformApplyFlowLogsPassRole",
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "arn:aws:iam::[ユーザID]:role/ai-content-platform-vpc-flowlogs"
        },
        {
            "Sid": "TerraformApplyElbv2CreateLoadBalancer",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:CreateLoadBalancer"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformApplyElbv2AddTags",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:AddTags"
            ],
            "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:loadbalancer/*"
        },
        {
            "Sid": "AddTagsToApiTargetGroup",
            "Effect": "Allow",
            "Action": "elasticloadbalancing:AddTags",
            "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-api-tg/*"
        },
        {
            "Sid": "CreateAlbListener",
            "Effect": "Allow",
            "Action": "elasticloadbalancing:CreateListener",
            "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:loadbalancer/app/ai-content-platform-alb/[リソースID]"
        },
        {
            "Sid": "DescribeAlbListeners",
            "Effect": "Allow",
            "Action": "elasticloadbalancing:DescribeListeners",
            "Resource": "*"
        },
        {
            "Sid": "DescribeLoadBalancerAttributes",
            "Effect": "Allow",
            "Action": "elasticloadbalancing:DescribeLoadBalancerAttributes",
            "Resource": "*"
        },
        {
            "Sid": "AllowCreateECSServiceLinkedRole",
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "arn:aws:iam::*:role/aws-service-role/ecs.amazonaws.com/AWSServiceRoleForECS*",
            "Condition": {
                "StringLike": {
                    "iam:AWSServiceName": "ecs.amazonaws.com"
                }
            }
        },
        {
            "Sid": "DescribeListenerAttributes",
            "Effect": "Allow",
            "Action": "elasticloadbalancing:DescribeListenerAttributes",
            "Resource": "*"
        },
        {
            "Sid": "AllowCreateAlbListenerRule",
            "Effect": "Allow",
            "Action": "elasticloadbalancing:CreateRule",
            "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener/app/ai-content-platform-alb/[ID]/*"
        },
        {
            "Sid": "AllowAddTagsToApiListenerRule",
            "Effect": "Allow",
            "Action": "elasticloadbalancing:AddTags",
            "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener-rule/app/ai-content-platform-alb/[ID]/[ID]/*"
        },
        {
            "Sid": "AllowDescribeAlbRules",
            "Effect": "Allow",
            "Action": "elasticloadbalancing:DescribeRules",
            "Resource": "*"
        },
        {
            "Sid": "AllowDescribeTargetHealth",
            "Effect": "Allow",
            "Action": "elasticloadbalancing:DescribeTargetHealth",
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

### 04. ec2:Describe*
　Amazon EC2に関するすべての参照・読み取り（Describe系）APIアクションを許可するワイルドカード設定を意味します。

### 05. ec2:CreateVpc
　AWSの仮想ネットワーク環境であるVPC（Virtual Private Cloud）を新規作成するために必要なAWS IAMのアクション（権限）です。

### 06. ec2:DeleteVpc
　TerraformがAWS上に構築されたVPC（Virtual Private Cloud）を削除（または変更にともなう再作成）する際に、実行ユーザーやロールに必要となるAWSのIAM権限（アクション）です。

### 07. ec2:ModifyVpcAttribute
　AWS上で VPCの属性（DNS解決やDNSホスト名の有効化など）を変更するために必要なIAMポリシーのアクション（権限） です。

### 08. ec2:CreateSubnet
　AWS上にサブネット（Subnet）を新規作成する際に必要となる AWSのIAM（Identity and Access Management）アクション（権限） です。

### 09. ec2:DeleteSubnet
　AWS上でサブネットを削除する際に呼び出されるAWSのAPIアクション（IAM権限）です。

### 10. ec2:ModifySubnetAttribute
　AWSのサブネット属性を変更する際に必要な IAMポリシーのアクション（権限） です。

### 11. ec2:CreateRouteTable
　AWSのVPC内において新しいルートテーブルを作成する権限（IAMアクション）です。

### 12. ec2:DeleteRouteTable
　AWSが提供するIAM（Identity and Access Management）のポリシーアクション（権限）のことです。

### 13. ec2:CreateRoute
　AWSのVPC内にあるルートテーブルに新しい経路（ルート）を追加するための AWS IAM（Identity and Access Management）のアクション（権限）です。

### 14. ec2:ReplaceRoute
　TerraformでAWSのルートテーブル内の既存ルートを更新（変更）する際に内部的に呼び出されるAWS EC2 APIアクションです。

### 15. ec2:DeleteRoute
　Amazon VPCのルートテーブルから特定のルート（経路）を削除する権限を付与するAWSのAPIアクションです。

### 16. ec2:AssociateRouteTable
　AWSのVPC（Virtual Private Cloud）内において、ルートテーブルをサブネットやインターネットゲートウェイに関連付けるためのAWS IAM（権限）アクションです。

### 17. ec2:DisassociateRouteTable
　AWSのサブネットやゲートウェイと、ルートテーブルの「関連付け（アソシエーション）」を解除するAWSのAPIアクション（IAM権限）です。

### 18. ec2:CreateInternetGateway
　Terraformを実行するIAMユーザーやロールに付与するAWSのIAMポリシー（権限）のアクション名です。

### 19. ec2:AttachInternetGateway
　AWSのインターネットゲートウェイ（IGW）をVPC（Virtual Private Cloud）にアタッチ（接続）するために必要なAWS IAMのアクション（権限）です。

### 20. ec2:DetachInternetGateway
　AWSの仮想プライベートクラウド（VPC）からインターネットゲートウェイ（IGW）を取り外す（デタッチする）ためのIAM権限（アクション）です。

### 21. ec2:DeleteInternetGateway
　AWSのインターネットゲートウェイ（Internet Gateway: IGW）を削除するために必要なIAMポリシーのアクション（権限）です。

### 22. ec2:DeleteInternetGateway
　AWSのインターネットゲートウェイ（Internet Gateway: IGW）を削除するために必要なIAMポリシーのアクション（権限）です。

### 23. ec2:AllocateAddress
　AWSの固定パブリックIPv4アドレスである Elastic IP (EIP) を新しく取得（割り当て）する際に呼び出される AWS EC2 の API アクション（IAM 権限）です。

### 24. ec2:ReleaseAddress
　AWSの Elastic IP アドレス（EIP）を解放（削除）する権限 を表すIAMアクションです。

### 25. ec2:CreateNatGateway
　AWS上でNAT ゲートウェイ（NAT Gateway）を作成するために必要なIAMポリシーのアクション（権限）です。

### 26. ec2:CreateNatGateway
　AWS上でNAT ゲートウェイ（NAT Gateway）を作成するために必要なIAMポリシーのアクション（権限）です。

### 27. ec2:DeleteNatGateway
　AWSの管理する NAT ゲートウェイ（NAT Gateway）を削除するために必要なAWS IAM（アクセス権限）のアクションです。

### 28. ec2:CreateSecurityGroup
　AWSの仮想ファイアウォールであるセキュリティグループを新しく作成するためのAWS IAM（Identity and Access Management）の権限（アクション）です。

### 29. ec2:DeleteSecurityGroup
　TerraformがAWS上のセキュリティグループを削除（変更による再作成や terraform destroy など）する際に、AWS APIに対して発行するIAM権限（アクション）のことです。

### 30. ec2:AuthorizeSecurityGroupIngress
　AWSのインフラをコードで管理する Terraform において、EC2 セキュリティグループにインバウンド（受信用）ルールを追加する権限（IAMアクション）、またはそれに付随するリソースの設定を指します。

### 31. ec2:AuthorizeSecurityGroupEgress
　AWSのVPC内にあるセキュリティグループに対して、アウトバウンド（送信 / Egress）通信の許可ルールを追加・作成するためのAWS IAM（Identity and Access Management）のアクション権限です。

### 32. ec2:RevokeSecurityGroupIngress
　セキュリティグループのインバウンド（受信）ルールを削除（撤回）するためのIAM権限（APIアクション）です。

### 33. ec2:RevokeSecurityGroupEgress
　AWSのEC2（VPC）環境において、セキュリティグループから特定のアウトバウンド（送信元から外部への通信）ルールを削除するためのAWS IAM権限（APIアクション）です。

### 34. ec2:CreateFlowLogs
　AWSのVPCフローログ（VPC Flow Logs）を作成するために必要な AWSのIAM（権限）アクション の名称です。

### 35. ec2:DeleteFlowLogs
　AWSのVPCフローログ（VPC Flow Logs）を削除するために必要な AWS IAMポリシーのアクション（権限） です。

### 36. ec2:CreateTags
　Amazon EC2 リソース（インスタンス、VPC、セキュリティグループ、EBSボリュームなど）にタグを追加・上書きするためのAWS IAMポリシーのアクション（権限）です。

### 37. ec2:DisassociateAddress
AWS の Elastic IP（EIP）と EC2 インスタンス（またはネットワークインターフェース）の紐付けを解除する（ディソシエイトする）権限（APIアクション）

### 38. route53:CreateHostedZone
　Amazon Route 53 で新しいパブリックまたはプライベートのホストゾーン（DNSレコードを管理するコンテナ）を作成するための IAM アクション（権限）です。

### 39. route53:ChangeResourceRecordSets
　Amazon Route 53の特定のホストゾーン内にあるDNSレコードセット（AレコードやCNAMEレコードなど）を作成、変更、削除するためのAWS IAM権限です。

### 40. route53:GetHostedZone
　AWSのIAM（Identity and Access Management）ポリシーで使われる権限（アクション）の1つです。

### 41. route53:ListHostedZones
　AWS Route 53のホストゾーン（DNS設定の管理単位）の一覧を取得するためのIAMポリシーのアクション（権限）です。

### 42. route53:CreateHealthCheck
　Amazon Route 53のヘルスチェックを作成するためのAWS IAM（識別およびアクセス管理）のポリシーアクション（権限）です。

### 43. ec2:GetSecurityGroupsForVpc
　AWSのIAMポリシー内で使用される、指定されたVPCに関連付けられているセキュリティグループの一覧を取得するための権限（アクション）です。

### 44. elasticloadbalancing:ModifyTargetGroupAttributes
　AWSの Elastic Load Balancing (ELB v2 / ALB・NLB・GWLB) のターゲットグループに設定された「属性（Attributes）」を修正・変更するためのAPIアクション（IAM権限）です。

### 45. elasticloadbalancing:CreateTargetGroup
　AWSのElastic Load Balancing（特にALBやNLB、GWLB）において、リクエストの振り分け先となる「ターゲットグループ」を新規作成するためのIAMアクション（権限）です。

### 46. elasticloadbalancing:DescribeLoadBalancers
　AWSのロードバランサー（ALB、NLB、CLB）の情報を取得・表示するためのAWS IAM（認可）のアクション権限です。

### 47. elasticloadbalancing:DescribeTargetGroups
　AWSのElastic Load Balancing（ELB/ALB/NLB/GWLB）のターゲットグループ情報を参照・取得するAPIアクション（権限）のことです。

### 48. elasticloadbalancing:DescribeTags
　AWSのロードバランサー（ALB、NLB、CLBなど）やターゲットグループに付与されている「タグ（Tags）の情報を取得（読み取り）する」ためのAWS IAMアクション（権限）です。

### 49. elasticloadbalancing:DescribeTargetGroupAttributes
　AWSの Elastic Load Balancing (ELB v2: ALB/NLB/GWLB) のターゲットグループの属性情報を取得するAPI権限（IAMアクション） を指します。

### 50. iam:PassRole
　ユーザーやサービス（Terraformの実行者など）が、EC2、Lambda、ECSなどのAWSリソースを作成・起動する際に「このロールをそのリソースに渡す（アタッチする）こと」を許可する権限です。

### 51. elasticloadbalancing:CreateLoadBalancer
　AWS の IAM（Identity and Access Management）ポリシーで定義される「ALB/NLB（Elastic Load Balancing）を作成する権限」のアクション名です。

### 52. elasticloadbalancing:AddTags
　AWS の Elastic Load Balancing（ELB/ALB/NLB）リソースに対して「タグを追加・更新する」権限（IAMアクション）を指します。

### 53. elasticloadbalancing:CreateListener
　AWSの Elastic Load Balancing (ELB/ALB/NLB) でリスナー（接続を受け付けるポートやプロトコルの設定）を新規作成する際に必要なIAMポリシーのアクション（権限）です。

### 54. elasticloadbalancing:DescribeListeners
　AWSのロードバランサー（ALB、NLB、GLB）に設定されている「リスナー（通信を受け付けるポートやプロトコルの設定）」の情報を取得（参照）するためのAWS IAM権限（APIアクション）です。

### 55. elasticloadbalancing:DescribeLoadBalancerAttributes
　AWSのロードバランサー（ALB、NLB、GLBなど）の属性情報を取得するために必要なAWS IAMポリシーのアクション（権限）です。

### 56. iam:CreateServiceLinkedRole
　AWSのサービスリンク（サービスにリンクされた）ロールを作成するために必要な「AWS IAMの権限（アクション）」を指しています。

### 57. Condition
　主に入力変数の検証（validation）、リソース作成前後のチェック（precondition / postcondition）、および三項演算子などの条件分岐の文脈で使われます。

### 58. StringLike
　AWSのIAMポリシー（aws_iam_policy_document など）の Condition（条件）で使う比較演算子の一つです。ワイルドカード（* や ?）を用いた 部分一致（あいまい一致） で文字列を評価します。

### 59. iam:AWSServiceName
　AWS サービスが他の AWS リソースにアクセスする権限（サービスロール）を制御・制限するために使用される条件キー（Condition Key）です。

### 60. elasticloadbalancing:DescribeListenerAttributes
　AWSのロードバランサー（ALB / NLB）におけるリスナー属性（リスナーの各種設定値）の情報を読み取る（取得する）ためのIAM権限です。

### 61. elasticloadbalancing:CreateRule
　AWSのロードバランサー（特に ALB: Application Load Balancer）において、新しいリスナールール（ルーティング規則）を作成するための、AWS IAM（Identity and Access Management）の権限（アクション）です。

### 62. elasticloadbalancing:AddTags
　AWSのElastic Load Balancing（ELB/ALB/NLB）リソースに「タグ（Key-Value）」を追加または更新するためのIAMポリシーの権限（アクション）です。

### 63. elasticloadbalancing:CreateListener
　AWSのロードバランサー（ALB、NLB、GLB）にリスナー（接続を待ち受ける設定）を新規作成する際に呼び出される、AWS IAMのAPIアクション権限のことです。

### 64. elasticloadbalancing:DescribeListeners
　AWSのロードバランサー（ALB、NLB、GLB）に設定されている「リスナー（通信の受け口）」の情報を取得するためのIAMポリシー（権限）のアクションです。

### 65. elasticloadbalancing:DescribeLoadBalancerAttributes
　AWSのロードバランサー（ALB、NLB、GLBなど）に設定されている属性（追加設定）の情報を取得するためのAWS IAM権限（APIアクション）です。

### 66. iam:CreateServiceLinkedRole
　AWSが提供する特定のサービス（例: ECS、Auto Scaling、EC2 Spot Fleetなど）専用の「サービスにリンクされたロール（Service-Linked Role）」を作成するためのIAM権限（アクション）です。

### 67. iam:AWSServiceName
　主にサービスリンクロール（Service-Linked Role）の作成や制限を行う際に使用される、AWS IAMの条件キー（Condition Key）です。

### 68. elasticloadbalancing:DescribeListenerAttributes
　AWSのElastic Load Balancing (ELBv2：ALB / NLB / GWLB) における リスナーの属性（Attributes）を取得・確認するための一読取（Read）用のIAM権限 です。

### 69. elasticloadbalancing:CreateRule
　AWSの Application Load Balancer (ALB) などのリスナールール（条件に応じたトラフィックの振り分け設定）を作成するために必要な AWS IAMポリシーのアクション（権限） です。

### 70. elasticloadbalancing:AddTags
　AWSのELB（Elastic Load Balancing）リソースにタグを追加・更新するためのIAMポリシーアクション（権限）です。

### 71. elasticloadbalancing:DescribeRules
　リソースやプロラクティスそのものの名前ではなく、AWSのIAMポリシーで定義されるALB（Application Load Balancer）のAPIアクション（権限）です。

### 72. elasticloadbalancing:DescribeTargetHealth
　AWSのロードバランサー（ALBやNLBなど）において、ターゲットグループ内に登録されているターゲット（EC2やECSタスクなど）の「ヘルスチェック状態（正常か異常か）」を取得するためのIAMアクション（権限）です。
