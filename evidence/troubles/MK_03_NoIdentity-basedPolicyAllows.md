## Trouble on Making_03
---
### Message

Error: listing tags for SNS (Simple Notification) Topic (arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts): operation error SNS: ListTagsForResource, https response error StatusCode: 403, RequestID: 4dae14d8-3553-5635-94c2-77e73024e24c, AuthorizationError: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: SNS:ListTagsForResource on resource: arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts because no identity-based policy allows the SNS:ListTagsForResource action

---

###【要約】

エラー: SNS (簡易通知) トピックのタグのリスト (arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts): 

操作エラー SNS: ListTagsForResource、https 応答エラー StatusCode: 403、RequestID: 4dae14d8-3553-5635-94c2-77e73024e24c、

AuthorizationError: 

User: arn:aws:iam::[ユーザID]:user/[ユーザ名] には、アイデンティティベースのポリシーが SNS:ListTagsForResource アクションを許可していないため、

リソース: arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts に対する SNS:ListTagsForResource を実行する権限がありません。

---

### 【原因】

これは「SNS Topicを作成できない」というエラーではありません。

TerraformがSNS Topicについて**タグ情報を取得しようとしたところ、参照権限がなかった**というエラーです。

AWS APIにも `ListTagsForResource` は正式なSNS APIとして存在します。([AWS ドキュメント][4])
---


###【対応方法】

#### 1. 

```json
{
  "Effect": "Allow",
  "Action": [
    "sns:ListTagsForResource"
  ],
  "Resource": "arn:aws:sns:[リージョン]:[ユーザID]:ai-content-platform-dev-alerts"
}
```

または、TerraformがこのTopicを作成・更新・削除まで行うなら、SNS関連の必要権限をまとめて持たせます。

今回のエラーだけを見る限り、**Terraformコードを修正する必要はありません。**


#### 2. 修正後の作業手順

```hcl
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform show -no-color tfplan > plan.txt
terraform apply tfplan
```

