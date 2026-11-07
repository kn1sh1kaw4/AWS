## Trouble on Making_01
---
### Message

Error: deleting IAM Role (ai-content-platform-vpc-flowlogs): reading IAM Instance Profiles for Role (ai-content-platform-vpc-flowlogs): operation error IAM: ListInstanceProfilesForRole, https response error StatusCode: 403, RequestID: cb125218-9abe-4a5c-8016-078cb3755633, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: iam:ListInstanceProfilesForRole on resource: role ai-content-platform-vpc-flowlogs because no identity-based policy allows the iam:ListInstanceProfilesForRole action. Go to https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/authorization-details/9cl85o471rsbt1t8vzh5ii6r3 for complete details, or call the GetRequestAuthorizationDetails API with the following authorization id: 9cl85o471rsbt1t8vzh5ii6r3

---

###【要約】
エラー: ELBv2 ターゲットグループ (ai-content-platform-app-tg) の作成に失敗しました。

Elastic Load Balancing v2: CreateTargetGroup 操作でエラーが発生しました。

HTTP レスポンスステータスコード: 403、リクエスト ID: 44c209e4-676a-4b45-a528-099b40dcdfc5。

API エラー AccessDenied: ユーザー arn:aws:iam::[User-ID]:user/[ユーザ名] には、リソース arn:aws:elasticloadbalancing:[リージョン]:[User-ID]:targetgroup/ai-content-platform-app-tg/* に対して elasticloadbalancing:CreateTargetGroup アクションを実行する権限がありません。

これは、elasticloadbalancing:CreateTargetGroup アクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

Terraform実行ユーザーに以下の権限がない。

```text
elasticloadbalancing:CreateTargetGroup
```

---

###【対応方法】

#### 1. Terraform実行ユーザの許可ポリシーに、下記を追記

```json
{
  "Effect": "Allow",
  "Action": [
    "elasticloadbalancing:CreateTargetGroup"
  ],
  "Resource": "*"
}
```

ただし、今回のTerraformではTarget Groupだけでなく、

* ALB
* Listener
* Listener Rule
* Target Group
* タグ付与
* Target登録
* Health Check確認

なども行うため、**`CreateTargetGroup` だけ追加して終わりとは考えない方がよい**です。

特にリソース作成時にタグを付ける場合、`elasticloadbalancing:AddTags` も必要になる場合があります。AWSも、作成時タグ付けには作成アクションに加えて `AddTags` の権限が必要となる。

#### 2. 修正後の作業手順

```hcl
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform show -no-color tfplan > plan.txt
terraform apply tfplan
```

