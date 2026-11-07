## Trouble on Making_12
---
### Message

Error: creating Step Functions State Machine (ai-content-platform-generation): operation error SFN: CreateStateMachine, https response error StatusCode: 400, RequestID: 9f80abda-c765-4903-b943-95330f02fdc2, api error AccessDeniedException: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: iam:PassRole on resource: arn:aws:iam::[ユーザID]:role/ai-content-platform-sfn because no identity-based policy allows the iam:PassRole action.

---

###【要約】

エラー: Step Functions ステートマシン (ai-content-platform-generation) の作成中にエラーが発生しました。

操作エラー SFN: CreateStateMachine、HTTP レスポンスエラー (ステータスコード: 400、

リクエスト ID: 9f80abda-c765-4903-b943-95330f02fdc2)、API エラー AccessDeniedException: 

ユーザー arn:aws:iam::[ユーザID]:user/[ユーザ名] は、リソース arn:aws:iam::[ユーザID]:role/ai-content-platform-sfn に対して 

iam:PassRole アクションを実行する権限がありません。

これは、iam:PassRole アクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

アクションを実行する権限が不足している

iam:PassRole

---


###【対応方法】

#### 1. 許可ポリシーに以下の記述を追加する

```hcl
        {
            "Sid": "TerraformApplyStepFunctionsPassRole",
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "arn:aws:iam::[ユーザID]:role/ai-content-platform-sfn"
        },
```


#### 2. プランの再作成

```hcl
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform show -no-color tfplan > plan.txt
terraform apply tfplan
```

