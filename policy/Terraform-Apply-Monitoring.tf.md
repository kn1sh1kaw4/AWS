## Terraform-Apply-Monitoring.tf

```hcl
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "TerraformApplyMonitoring",
            "Effect": "Allow",
            "Action": [
                "logs:*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformManagedGrafana",
            "Effect": "Allow",
            "Action": [
                "grafana:CreateWorkspace",
                "grafana:DescribeWorkspace",
                "grafana:UpdateWorkspace",
                "grafana:DeleteWorkspace",
                "grafana:ListWorkspaces",
                "grafana:UpdatePermissions"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformStates",
            "Effect": "Allow",
            "Action": [
                "states:ValidateStateMachineDefinition"
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

### 04. logs:*
主に AWSのIAMポリシー（権限設定） を定義する際や、一部のクラウドプロバイダにおけるログ監視・ストリーミングの対象を指定する際に使われるワイルドカード記述です。

### 05. grafana:CreateWorkspace
Amazon Managed Grafana (AMG) において新規のワークスペース（Grafanaサーバー環境）を作成する際に必要となる AWSのIAMアクション（権限） です。

### 06. grafana:CreateWorkspace
Amazon Managed Grafana (AMG) において新規のワークスペース（Grafanaサーバー環境）を作成する際に必要となる AWSのIAMアクション（権限） です。

### 07. grafana:CreateWorkspace
AWSが提供するマネージドサービスである Amazon Managed Grafana (AMG) において、指定したワークスペースの詳細情報を取得するためのIAM（AWSの権限管理）アクション（API操作）です。

### 08. grafana:DescribeWorkspace
Amazon Managed Grafana (AMG) のワークスペースに関する詳細情報を取得するための AWS IAM（アクセス許可）アクション です。

### 09. grafana:UpdateWorkspace
Amazon Managed Grafana (AMG) のワークスペースの設定を更新する際、Terraform（AWS プロバイダー）が内部的に呼び出す AWS API の IAM アクション（権限）です。

### 10. grafana:DeleteWorkspace
Amazon Managed Grafana (AMG) のワークスペースを削除するために必要な AWSのIAMアクション（権限） です。

### 11. grafana:ListWorkspaces
AWSが提供するフルマネージドサービスであるAmazon Managed Grafana (AMG) のIAMアクション（権限）の1つです。

### 12. grafana:ListWorkspaces
主に Amazon Managed Grafana (AMG) において、ワークスペース内のユーザーやグループに関連付けられたアクセス許可（パーミッション）を更新・管理するための IAM ポリシーのアクション（権限）です。

### 13. grafana:UpdatePermissions
主に Amazon Managed Grafana (AMG) において、ワークスペース内のユーザーやグループに関連付けられたアクセス許可（パーミッション）を更新・管理するための IAM ポリシーのアクション（権限）です。

### 14. states:ValidateStateMachineDefinition
AWS Step Functionsのステートマシン定義（JSON/YAML形式のAmazon States Languageによる記述）の構文や構造が正しいかを検証（バリデーション）するためのAWS APIアクションです。

