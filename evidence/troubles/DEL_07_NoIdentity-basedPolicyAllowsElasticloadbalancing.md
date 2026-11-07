## Trouble on Dellition_07
---
### Message

Error: deleting ELBv2 Listener Rule (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener-rule/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c/2ebc0afc54520d02): operation error Elastic Load Balancing v2: DeleteRule, https response error StatusCode: 403, RequestID: a33caaf8-6ff2-4f77-a494-37cb181939a3, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: elasticloadbalancing:DeleteRule on resource: arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener-rule/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c/2ebc0afc54520d02 because no identity-based policy allows the elasticloadbalancing:DeleteRule action.

---

###【要約】

エラー: ELBv2 リスナールール (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener-rule/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c/2ebc0afc54520d02) の削除に失敗しました。

Elastic Load Balancing v2: DeleteRule 操作においてエラーが発生しました。

HTTP ステータスコード: 403、リクエスト ID: a33caaf8-6ff2-4f77-a494-37cb181939a3。

API エラー AccessDenied: ユーザー `arn:aws:iam::[ユーザID]:user/[ユーザ名]` には、リソース `arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener-rule/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c/2ebc0afc54520d02` に対して `elasticloadbalancing:DeleteRule` アクションを実行する権限がありません。

これは、`elasticloadbalancing:DeleteRule` アクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

ELBリスナールール削除権限不足

---

###【対応方法】

#### 1. Terraform-Aplly-Iamに以下の行を追加する。

```hcl
        {
            "Sid": "TerraformDeleteElbV2",
            "Effect": "Allow",
            "Action": "elasticloadbalancing:DeleteRule",
            "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener-rule/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c/2ebc0afc54520d02"
        }
```

#### 2. 修正後の作業手順

ここからは、権限を追加してすぐに再度 terraform apply destroy.tfplan を実行するのではなく、次の順序で進める。

特に今回は、最初の削除処理で一部のリソースが削除されている可能性があります。最初に作成した destroy.tfplan をそのまま再利用しない。

#### 3. AWS Backupの復旧ポイントを削除
   
   保持する必要がないことを確認したうえで、対象ボールトの復旧ポイントを削除します。

#### 4. Terraformの状態を退避
   
   作業ディレクトリで、念のため現在の状態を保存します。

```hcl
   terraform state pull > terraform-state-before-retry.json
```

#### 5. 削除計画を再作成

```hcl
   terraform plan -destroy -out="destroy-retry.tfplan"
```
   
   内容をテキスト化します。

```hcl
   terraform show -no-color destroy-retry.tfplan > destroy-retry-plan.txt
```


#### 6. 新しい削除計画を確認

   - 既に削除されたリソースが対象から外れているか。
   - 残存するリソースがすべて削除対象になっているか。
   - 意図しないリソースが対象になっていないか。
   - 追加・変更が発生していないか。


#### 7. 新しい計画を適用
   
   問題がなければ実行します。

```hcl
   terraform apply destroy-retry.tfplan
```


