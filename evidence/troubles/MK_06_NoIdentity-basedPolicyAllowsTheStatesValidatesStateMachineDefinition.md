## Trouble on Making_06
---
### Message

Error: validating Step Functions State Machine definition: operation error SFN: ValidateStateMachineDefinition, https response error StatusCode: 400, RequestID: 5ea9e560-7c3d-4e09-acfa-971cb0da4182, api error AccessDeniedException: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: states:ValidateStateMachineDefinition on resource: arn:aws:states:[リージョン]:[ユーザID]:stateMachine:* because no identity-based policy allows the states:ValidateStateMachineDefinition action

---

###【要約】

エラー: Step Functions ステートマシンの定義の検証中にエラーが発生しました。

操作エラー SFN: ValidateStateMachineDefinition、

HTTP レスポンスエラー (ステータスコード: 400)、

リクエスト ID: 5ea9e560-7c3d-4e09-acfa-971cb0da4182。

API エラー AccessDeniedException: ユーザー `arn:aws:iam::[ユーザID]:user/[ユーザ名]` は、

リソース `arn:aws:states:[リージョン]:[ユーザID]:stateMachine:*` に対して 

`states:ValidateStateMachineDefinition` アクションを実行する権限がありません。

これは、ID ベースのポリシーのいずれにも、`states:ValidateStateMachineDefinition` アクションを許可する設定が含まれていないためです。
---

### 【原因】

TerraformがStep FunctionsのState Machine定義をAWS側で検証しようとしています。

しかし、

```text
states:ValidateStateMachineDefinition
```

がありません。

AWS公式ドキュメントでも、このアクションはState Machine定義の検証用のサービスレベル権限として明記されています。([AWS ドキュメント][7])

---


###【対応方法】

#### 1. 

```json
{
  "Effect": "Allow",
  "Action": [
    "states:ValidateStateMachineDefinition"
  ],
  "Resource": "*"
}
```

ただし、これだけで終わらない可能性があります。

State Machineそのものを作成するなら、少なくとも、

```text
states:CreateStateMachine
states:UpdateStateMachine
states:DeleteStateMachine
states:DescribeStateMachine
states:ValidateStateMachineDefinition
```

などが必要になります。

さらにState Machineが実行ロールを使用する構成なら、

```text
iam:PassRole
```

も必要になる可能性があります。AWS公式のStep Functions権限説明でも、State Machine用のIAMロールを渡す場合の `iam:PassRole` が説明されています。([AWS ドキュメント][8])


#### 2. 修正後の作業手順

```hcl
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform show -no-color tfplan > plan.txt
terraform apply tfplan
```

