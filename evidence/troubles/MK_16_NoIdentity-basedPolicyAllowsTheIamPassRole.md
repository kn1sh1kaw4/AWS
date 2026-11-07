## Trouble on Making_16
---
### Message

Error: creating Step Functions State Machine (ai-content-platform-generation): operation error SFN: CreateStateMachine, https response error StatusCode: 400, RequestID: 9f80abda-c765-4903-b943-95330f02fdc2, api error AccessDeniedException: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: iam:PassRole on resource: arn:aws:iam::[ユーザ名]:role/ai-content-platform-sfn because no identity-based policy allows the iam:PassRole action

---

###【要約】

エラー: Step Functions ステートマシン (ai-content-platform-generation) の作成に失敗しました。

操作エラー SFN: CreateStateMachine、HTTP レスポンスエラー (ステータスコード: 400、

リクエスト ID: 9f80abda-c765-4903-b943-95330f02fdc2)、API エラー AccessDeniedException が発生しました。

ユーザー `arn:aws:iam::[ユーザID]:user/[ユーザ名]` には、リソース `arn:aws:iam::[ユーザ名]:role/ai-content-platform-sfn` に対して 

`iam:PassRole` アクションを実行する権限がありません。

これは、アイデンティティベースのポリシーのいずれにも、`iam:PassRole` アクションを許可する設定が含まれていないためです。

---

### 【原因】

実行権限不足

iam:PassRole

---


###【対応方法】

（"terrafrorm plan -out=tfplan"実行時に taintになっている部分があれば、そこを解消してから実行する）

#### 1. IAMポリシーに、**ELBのService-linked roleに限定**する許可ポリシーを追加する。

```json
        {
            "Sid": "TerraformApplyBackupPassRole",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::[ユーザID]:role/ai-content-platform-backup-role"
        },
```

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

