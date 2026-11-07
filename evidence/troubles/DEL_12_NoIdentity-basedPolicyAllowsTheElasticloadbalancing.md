## Trouble on Dellition_12
---
### Message

Error: deleting ELBv2 Target Group (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-api-tg/971db7989cc7da6b): operation error Elastic Load Balancing v2: DeleteTargetGroup, https response error StatusCode: 403, RequestID: 2a3db0ef-d034-42fb-8cdd-fd92e2ac191e, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: elasticloadbalancing:DeleteTargetGroup on resource: arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-api-tg/971db7989cc7da6b because no identity-based policy allows the elasticloadbalancing:DeleteTargetGroup action.

---

###【要約】

エラー: ELBv2 ターゲットグループ (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-api-tg/971db7989cc7da6b) の削除に失敗しました。

Elastic Load Balancing v2: DeleteTargetGroup 操作でエラーが発生しました。

HTTP レスポンスエラー (ステータスコード: 403、リクエストID: 2a3db0ef-d034-42fb-8cdd-fd92e2ac191e)。

API エラー AccessDenied: ユーザー (arn:aws:iam::[ユーザID]:user/[ユーザ名]) には、リソース (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-api-tg/971db7989cc7da6b) に対して `elasticloadbalancing:DeleteTargetGroup` アクションを実行する権限がありません。

これは、このアクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

アイデンティティベースのポリシー削除権限不足

---

###【対応方法】

#### 1. Terraform-Aplly-Networkから、"Action"が"elasticloadbalancing:〇〇"の許可ポリシーのすべてを別の許可ポリシー（Terraform-Apply-Balancer）を作成・転記する。

　↓

Terraform-Apply-Balancerに以下の行を追加

```hcl
        {
            "Sid": "TerraformApplyDelTgg",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:DeleteTargetGroup"
            ],
            "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-api-tg/971db7989cc7da6b"
        },
```

ただし、次の2点に注意してください。

1. `Resource` は、エラーメッセージに表示されたターゲットグループのARNと完全に一致させること。

2. `Terraform-Apply-Balancer` をIAMユーザーにアタッチすること。ポリシーJSONを作成しただけでは権限は付与されません。

また、今回の削除対象にはAPI用以外のターゲットグループも含まれていたため、今後別のターゲットグループで同様のエラーが発生する可能性があります。必要に応じて、削除対象のターゲットグループARNを追加してください。


#### 2. 修正後の作業手順

ここからは、権限を追加してすぐに再度 terraform apply destroy.tfplan を実行するのではなく、次の順序で進める。

特に今回は、最初の削除処理で一部のリソースが削除されている可能性があります。最初に作成した destroy.tfplan をそのまま再利用しない。


#### 3. AWS Backupの復旧ポイントを削除
   
   保持する必要がないことを確認したうえで、対象ボールトの復旧ポイントを削除します。

#### 4. Terraformの状態を退避
   
   作業ディレクトリで、念のため現在の状態を保存します。

```hcl
   terraform state pull > terraform-state-before-retry2.json
```

#### 5. 削除計画を再作成

```hcl
   terraform plan -destroy -out="destroy-retry2.tfplan"
```
   
   内容をテキスト化します。

```hcl
   terraform show -no-color destroy-retry2.tfplan > destroy-retry-plan2.txt
```


#### 6. 新しい削除計画を確認

   - 既に削除されたリソースが対象から外れているか。
   - 残存するリソースがすべて削除対象になっているか。
   - 意図しないリソースが対象になっていないか。
   - 追加・変更が発生していないか。


#### 7. 新しい計画を適用
   
   問題がなければ実行します。

```hcl
   terraform apply destroy-retry2.tfplan
```

