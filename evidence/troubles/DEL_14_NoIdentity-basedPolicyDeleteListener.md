## Trouble on Dellition_14
---
### Message

Error: deleting ELBv2 Listener (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c): operation error Elastic Load Balancing v2: DeleteListener, https response error StatusCode: 403, RequestID: e41646f4-c4de-423d-9031-ebaac09d6f0d, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: elasticloadbalancing:DeleteListener on resource: arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c because no identity-based policy allows the elasticloadbalancing:DeleteListener action.

---

###【要約】

エラー: ELBv2リスナー (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c) の削除に失敗しました。

操作エラー (Elastic Load Balancing v2: DeleteListener)、HTTPレスポンスエラー (ステータスコード: 403)、リクエストID: e41646f4-c4de-423d-9031-ebaac09d6f0d。

APIエラー (AccessDenied): ユーザー (arn:aws:iam::[ユーザID]:user/[ユーザ名]) には、リソース (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c) に対して `elasticloadbalancing:DeleteListener` アクションを実行する権限がありません。

これは、`elasticloadbalancing:DeleteListener` アクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

IAMポリシーの削除権限不足

---

###【対応方法】

#### 1. Terraform-Apply-Balancerに以下の行を追加

```hcl
        {
            "Sid": "TerraformApplyDelListner",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:DeleteListener"
            ],
            "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c"
        },
```
