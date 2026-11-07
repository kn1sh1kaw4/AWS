## Trouble on Making_05
---
### Message

Error: creating Lambda Function (ai-content-platform-worker): operation error Lambda: CreateFunction, https response error StatusCode: 403, RequestID: 870b8ed0-b06b-4af5-a2e3-f3240f8d8cba, api error AccessDeniedException: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: iam:PassRole on resource: arn:aws:iam::[ユーザID]:role/ai-content-platform-lambda because no identity-based policy allows the iam:PassRole action

---

###【要約】

エラー: Lambda関数 (ai-content-platform-worker) の作成中にエラーが発生しました。

操作: Lambda: CreateFunction。HTTPレスポンスエラー (ステータスコード: 403)。

リクエストID: 870b8ed0-b06b-4af5-a2e3-f3240f8d8cba。

APIエラー: AccessDeniedException。

ユーザー (arn:aws:iam::[ユーザID]:user/[ユーザ名]) には、

リソース (arn:aws:iam::[ユーザID]:role/ai-content-platform-lambda) に対して `iam:PassRole` アクションを実行する権限がありません。

これは、`iam:PassRole` アクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

TerraformはLambda関数を作る際、

```text
Lambda Function
        ↓
ai-content-platform-lambda
```

というIAM実行ロールをLambdaに渡そうとしています。

Lambdaの作成APIでは、実行ロールのARNが必要です。AWS公式APIでもLambda Function作成時にexecution roleが必要とされています。([AWS ドキュメント][6])

つまり、

> IAMユーザーがそのロールを「自分で使う」権限

ではなく、

> **Lambdaサービスにそのロールを渡す権限**

が必要です。

---


###【対応方法】

#### 1. 

```json
{
  "Effect": "Allow",
  "Action": [
    "iam:PassRole"
  ],
  "Resource": "arn:aws:iam::[ユーザID]:role/ai-content-platform-lambda"
}
```

ここは非常に重要なので、

```json
"Resource": "*"
```

にはせず、**今回のLambda実行ロールだけに限定**することを強く推奨します。


#### 2. 修正後の作業手順

```hcl
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform show -no-color tfplan > plan.txt
terraform apply tfplan
```

