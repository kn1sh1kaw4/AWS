## Trouble on Making_15
---
### Message

Error: creating ELBv2 application Load Balancer (ai-content-platform-alb): operation error Elastic Load Balancing v2: CreateLoadBalancer, https response error StatusCode: 403, RequestID: 4dd9d192-a0bd-4cbb-8ac2-c58429cec8ed, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: iam:CreateServiceLinkedRole on resource: arn:aws:iam::[ユーザID]:role/aws-service-role/elasticloadbalancing.amazonaws.com/AWSServiceRoleForElasticLoadBalancing because no identity-based policy allows the iam:CreateServiceLinkedRole action

---

###【要約】

エラー: ELBv2 アプリケーションロードバランサー (ai-content-platform-alb) の作成中にエラーが発生しました。

Elastic Load Balancing v2: CreateLoadBalancer の操作エラーです。

HTTPS レスポンスエラー (ステータスコード: 403、リクエストID: 4dd9d192-a0bd-4cbb-8ac2-c58429cec8ed)。

API エラー (AccessDenied): ユーザー `arn:aws:iam::[ユーザID]:user/[ユーザ名]` は、

リソース `arn:aws:iam::[ユーザID]:role/aws-service-role/elasticloadbalancing.amazonaws.com/AWSServiceRoleForElasticLoadBalancing` に対して 

`iam:CreateServiceLinkedRole` アクションを実行する権限がありません。

これは、`iam:CreateServiceLinkedRole` アクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

実行権限不足

iam:CreateServiceLinkedRole

つまり、

```text
ALB作成
 ↓
ELBがService-linked roleを必要とする
 ↓
AWSServiceRoleForElasticLoadBalancingが存在しない
 ↓
ELBが作成時にroleを作ろうとする
 ↓
Terraform実行ユーザーに
iam:CreateServiceLinkedRole がない
 ↓
403 AccessDenied
```

という流れである。

---


###【対応方法】

（"terrafrorm plan -out=tfplan"実行時に taintになっている部分があれば、そこを解消してから実行する）

#### 1. IAMポリシーに、**ELBのService-linked roleに限定**する許可ポリシーを追加する。

```json
{
  "Effect": "Allow",
  "Action": "iam:CreateServiceLinkedRole",
  "Resource": "arn:aws:iam::[ユーザID]:role/aws-service-role/elasticloadbalancing.amazonaws.com/AWSServiceRoleForElasticLoadBalancing",
  "Condition": {
    "StringLike": {
      "iam:AWSServiceName": "elasticloadbalancing.amazonaws.com"
    }
  }
}
```

AWSのService-linked roleはサービス自身が管理する特殊なロールなので、ユーザー側で通常のIAM Roleのように権限を設計するものではありません。([AWS ドキュメント][1])

[1]: https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/elb-service-linked-roles.html?utm_source=chatgpt.com "Elastic Load Balancing service-linked role - Elastic Load Balancing"


#### 2. planを再作成する。

```
terraform fmt
terraform validate
terraform plan -out=tfplan
terrafrom show -no-color tfplan > plan.txt
plan.txt確認
"0 to destroy"を確認
terraform apply tfplan
```

