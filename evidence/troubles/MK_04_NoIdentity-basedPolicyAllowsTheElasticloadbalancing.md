## Trouble on Making_04
---
### Message

Error: creating ELBv2 application Load Balancer (ai-content-platform-alb): operation error Elastic Load Balancing v2: CreateLoadBalancer, https response error StatusCode: 403, RequestID: a055c7f5-327a-4a2d-9786-2b4a97e19330, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: elasticloadbalancing:CreateLoadBalancer on resource: arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:loadbalancer/app/ai-content-platform-alb/* because no identity-based policy allows the elasticloadbalancing:CreateLoadBalancer action

---

###【要約】

エラー: ELBv2 アプリケーションロードバランサー (ai-content-platform-alb) の作成中にエラーが発生しました。Elastic Load Balancing v2: CreateLoadBalancer 操作において、HTTP 403 エラーが返されました (RequestID: a055c7f5-327a-4a2d-9786-2b4a97e19330)。API エラー AccessDenied: ユーザー arn:aws:iam::[ユーザID]:user/[ユーザ名] には、リソース arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:loadbalancer/app/ai-content-platform-alb/* に対して elasticloadbalancing:CreateLoadBalancer アクションを実行する権限がありません。これは、このアクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

Target Groupではなく**ALB本体の作成権限不足**です。

```text
elasticloadbalancing:CreateLoadBalancer
```

がありません。

AWSのELBv2 IAMアクションとして `CreateLoadBalancer` が定義されています。([AWS ドキュメント][5])

---


###【対応方法】

#### 1. 

```json
{
  "Effect": "Allow",
  "Action": [
    "elasticloadbalancing:CreateLoadBalancer"
  ],
  "Resource": "*"
}
```

さらに今回のTerraform構成では、少なくともELB関連として次のような権限を検討する必要があります。

```text
elasticloadbalancing:CreateLoadBalancer
elasticloadbalancing:CreateTargetGroup
elasticloadbalancing:CreateListener
elasticloadbalancing:CreateRule
elasticloadbalancing:AddTags
elasticloadbalancing:ModifyLoadBalancerAttributes
elasticloadbalancing:ModifyTargetGroup
elasticloadbalancing:Describe*
```

ただし、**いきなり `elasticloadbalancing:*` を与えるのではなく、今回のTerraformで実際に使用するアクションに絞る**のがおすすめです。


#### 2. 修正後の作業手順

```hcl
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform show -no-color tfplan > plan.txt
terraform apply tfplan
```

