## Terraform-Apply-Audit.tf

---

```hcl

{r, attr.source='.numberLines'}
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "TerraformApplyAudit",
            "Effect": "Allow",
            "Action": [
                "cloudtrail:DescribeTrails",
                "cloudtrail:GetTrail",
                "cloudtrail:GetTrailStatus",
                "cloudtrail:CreateTrail",
                "cloudtrail:UpdateTrail",
                "cloudtrail:DeleteTrail",
                "cloudtrail:StartLogging",
                "cloudtrail:StopLogging",
                "cloudtrail:AddTags",
                "cloudtrail:ListTags",
                "cloudtrail:RemoveTags"
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

### 04. cloudtrail:DescribeTrails 
AWS CloudTrailの証跡（Trail）情報を取得・確認するために、AWS Providerが内部的に呼び出すAWS APIの権限アクション（IAMポリシーのアクション）です。

### 05. cloudtrail:GetTrail
cloudtrail:GetTrail は、指定した AWS CloudTrail トレイルの設定情報を取得するための AWS IAM 権限（API アクション）です。

### 06. cloudtrail:GetTrailStatus
Terraform が AWS 上の CloudTrail 証跡（Trail）を管理（リソースの作成・更新・削除・読み込み）する際に、内部の AWS Provider がトレイルの現在の動作状況（ログ記録が有効か、最後のエラーは何か等）を確認するために呼び出す必須の API 権限です。

### 07.cloudtrail:CreateTrail
TerraformでAWS CloudTrailの証跡（Trail）を作成し、AWS APIの監査ログ（cloudtrail:CreateTrailに該当するようなイベント記録）を有効化するには、aws_cloudtrail リソースを使用します。

### 08. cloudtrail:UpdateTrail
Terraformにおいて、AWS CloudTrailの UpdateTrail APIアクションは独立したコマンドとして呼び出すのではなく、aws_cloudtrail リソースのパラメータを変更して terraform apply を実行することで自動的に呼び出されます。

### 09. cloudtrail:DeleteTrail
aws_cloudtrail リソースをコードから削除、または terraform destroy を実行した際に内部の DeleteTrail APIが呼び出され、対象の証跡（Trail）がAWS上から削除される仕組みです。

### 10. cloudtrail:StartLogging
AWSのログ記録サービスである「AWS CloudTrail」の証跡（Trail）によるログ記録を開始するために必要な AWS IAM権限（アクション） です。

### 11. cloudtrail:StopLogging
TerraformでAWS CloudTrailのログ記録を一時的に停止（StopLogging）させたい場合、またはセキュリティ監視観点で StopLogging APIの実行を検知・制限したい場合、その目的（文脈）に応じて設定方法が異なります。

### 12. cloudtrail:AddTags
TerraformでAWS CloudTrailにタグを管理・追加するには、主に aws_cloudtrail リソースの tags 引数 を使用します。

### 13. cloudtrail:ListTags
Terraformにおける cloudtrail:ListTags は、AWS CloudTrailリソース（証跡など）に付与されている **タグを読み取る（リスト化する）ためのAWS APIアクション（権限）** です。

### 14. cloudtrail:RemoveTags
Terraformを実行する際、cloudtrail:RemoveTags は **AWS CloudTrailリソース（Trailなど）からタグを削除・更新するためにTerraform（AWSプロバイダー）が必要とする必須のIAM権限（アクション）** です。

