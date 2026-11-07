## Trouble on Dellition_15
---
### Message

Error: deleting ELBv2 Load Balancer (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:loadbalancer/app/ai-content-platform-alb/d8edb03ceac706aa): operation error Elastic Load Balancing v2: DeleteLoadBalancer, https response error StatusCode: 403, RequestID: 2d6abeb8-3aa0-4c97-af6b-10ca572db969, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: elasticloadbalancing:DeleteLoadBalancer on resource: arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:loadbalancer/app/ai-content-platform-alb/d8edb03ceac706aa because no identity-based policy allows the elasticloadbalancing:DeleteLoadBalancer action

---

###【要約】

エラー: ELBv2ロードバランサー (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:loadbalancer/app/ai-content-platform-alb/d8edb03ceac706aa) の削除に失敗しました。

Elastic Load Balancing v2: DeleteLoadBalancer 操作でエラーが発生しました。

HTTPレスポンスエラー (ステータスコード: 403、リクエストID: 2d6abeb8-3aa0-4c97-af6b-10ca572db969)。

APIエラー (AccessDenied): ユーザー (arn:aws:iam::[ユーザID]:user/[ユーザ名]) には、

リソース (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:loadbalancer/app/ai-content-platform-alb/d8edb03ceac706aa) に対して

`elasticloadbalancing:DeleteLoadBalancer` アクションを実行する権限がありません。

これは、このアクションを許可するIDベースのポリシーが存在しないためです。

---

### 【原因】

DeleteLoadBalancer の権限不足


###【対応方法】

#### 1. 許可ポリシーを以下のように変更

旧）
```
		{
			"Sid": "CreateAlbListener",
			"Effect": "Allow",
			"Action": "elasticloadbalancing:CreateListener",
			"Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:loadbalancer/app/ai-content-platform-alb/d8edb03ceac706aa"
		},
```


新）
```
{
    "Sid": "CreateAlbListener",
    "Effect": "Allow",
    "Action": "elasticloadbalancing:CreateListener",
    "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:loadbalancer/app/ai-content-platform-alb/d8edb03ceac706aa"
},
{
    "Sid": "DeleteAlb",
    "Effect": "Allow",
    "Action": "elasticloadbalancing:DeleteLoadBalancer",
    "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:loadbalancer/app/ai-content-platform-alb/d8edb03ceac706aa"
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
   terraform apply destroy-retry[x].tfplan
```

※[x]:任意の数字（これまで使ったもの以外）

