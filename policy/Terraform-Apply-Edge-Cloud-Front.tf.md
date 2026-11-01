## Terraform-Apply-Edge-Cloud-Front.tf

```hcl
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "TerraformApplyEdgeCloudFront",
            "Effect": "Allow",
            "Action": [
                "cloudfront:GetDistribution",
                "cloudfront:CreateOriginAccessControl",
                "cloudfront:GetDistributionConfig",
                "cloudfront:ListDistributions",
                "cloudfront:ListDistributionsByWebACLId",
                "cloudfront:CreateDistribution",
                "cloudfront:UpdateDistribution",
                "cloudfront:DeleteDistribution",
                "cloudfront:TagResource",
                "cloudfront:UntagResource",
                "cloudfront:AssociateDistributionWebACL",
                "wafv2:ListWebACLs",
                "wafv2:CreateWebACL",
                "wafv2:GetWebACL",
                "wafv2:UpdateWebACL",
                "wafv2:DeleteWebACL",
                "wafv2:ListTagsForResource",
                "wafv2:TagResource",
                "wafv2:UntagResource",
                "wafv2:AssociateWebACL",
                "wafv2:DisassociateWebACL"
            ],
            "Resource": "*"
        },
        {
            "Sid": "TerraformApplyCloudTrail",
            "Effect": "Allow",
            "Action": [
                "cloudtrail:ListTags"
            ],
            "Resource": "arn:aws:cloudtrail:[リージョン名]:[アカウントID]:trail/ai-content-platform-trail"
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

### 04. cloudfront:GetDistribution 
AWSのポリシーや仕組みに関する文脈に応じて、主に以下の2つの意味を持ちます。

### 05. cloudfront:CreateOriginAccessControl
AWSのIAMポリシーで許可を定義、あるいはTerraformのPikeなどのツールが検出する「Amazon CloudFrontのOrigin Access Control (OAC) を作成するAPI操作権限」のことです。

### 06. cloudfront:GetDistributionConfig
AWSの CloudFrontディストリビューションの設定情報を取得するために必要なIAMポリシーのアクション（権限） です。

### 07. cloudfront:ListDistributions
AWS の Amazon CloudFront に存在するすべてのディストリビューション（配信設定）の一覧を取得するための AWS IAM 権限（アクション）です。

### 08. cloudfront:ListDistributionsByWebACLId
指定した AWS WAFのWeb ACL IDに関連付けられている Amazon CloudFront ディストリビューションの一覧を取得するためのAWS APIアクション（権限）です。

### 09. cloudfront:CreateDistribution
AWSのCloudFrontディストリビューションを新規作成するために必要な、AWS IAM（Identity and Access Management）の権限（アクション）を指します。

### 10. Cloudfront:GetDistribution
AWSのポリシーやIAMロールの権限設定（aws_iam_policy_document など）内で使われる AWS CloudFrontのIAMアクション（アクセス権限） の一つです。

### 11. cloudfront:UpdateDistribution
AWSのマネージドポリシーやIAMポリシーで定義される IAM（Identity and Access Management）のアクション（権限） の一つです。

### 12. cloudfront:DeleteDistribution
AWSのマネージド型CDNサービスである Amazon CloudFront のディストリビューションを削除するためのIAMアクション（権限）です。

### 13. cloudfront:TagResource
AWSの CloudFrontリソースに対してメタデータ（タグ）を付与・管理するためのAPIアクション権限 のことです。

### 14. cloudfront:UntagResource
Amazon CloudFrontのリソース（ディストリビューションや関数など）から「タグ（Tag）」を削除（解除）するために必要なAWS IAMのアクション（権限）です。

### 15. cloudfront:AssociateDistributionWebACL
web_acl_id（または関連するAWS APIの AssociateDistributionWebACL）は、Amazon CloudFront ディストリビューションと AWS WAF（Web Application Firewall）の Web ACL を紐付けるための設定および機能を指します。

### 16. wafv2:ListWebACLs
AWS WAFv2（Web Application Firewall v2）に作成されている Web ACL（Webアクセスコントロールリスト）の一覧を取得するための、AWS IAM（Identity and Access Management）のアクション（権限）です。

### 17. wafv2:CreateWebACL
AWS WAFv2のWeb ACL（Webアクセスコントロールリスト）を新規作成する際に、裏側で呼び出されるAWS APIのアクション名（IAM権限）を指します。

### 18. wafv2:GetWebACL
AWS WAF v2 において特定の Web ACL（Web アクセス制御リスト）の詳細情報を取得するための AWS IAM アクション（権限）です。

### 19. wafv2:UpdateWebACL
既存の Web ACL（Web アクセスコントロールリスト）の定義や設定を変更するための API アクションです。

### 20. wafv2:DeleteWebACL
AWS WAFv2のWeb ACL（Web Access Control List）を削除する際に、バックエンドで実行されるAWS APIアクション（権限）のことです。

### 21. wafv2:ListTagsForResource
AWS WAF v2 のリソース（Web ACL、IP Set、Rule Group など）に付与されているタグ（メタデータ）の一覧を取得するための AWS API アクションです。

### 22. wafv2:TagResource
AWS WAFv2（AWS WAF）のリソースに対してメタデータタグを付与するためのIAM権限（APIアクション）です。

### 23. wafv2:UntagResource
AWS WAFv2（AWS WAF）のリソースから付与されているタグ（Tag）を削除する操作（APIアクション）を指します。

### 24. wafv2:AssociateWebACL
AWS WAFv2のWeb ACL（アクセスコントロールリスト）を、ALBやAPI Gatewayなどの保護対象リソースに紐付ける（関連付ける）ためのAWS APIアクション（権限）です。

### 25. wafv2:DisassociateWebACL
AWS WAF v2のWeb ACL（Webアクセスコントロールリスト）と、適用対象のAWSリソース（ALBやAPI Gatewayなど）との 紐付けを解除（ディソシエイト）するために必要なAWS IAMのアクション（権限） です。

### 26. cloudtrail:ListTags
AWSのマネージドサービスである AWS CloudTrail に設定されている「タグ（Tags）」の一覧を取得する ためのIAMアクション（権限）です。
