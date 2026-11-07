## Trouble on Dellition_04
---
### Message

Error: deleting ECS Task Definition (ai-content-platform-api): operation error ECS: DeregisterTaskDefinition, https response error StatusCode: 400, RequestID: 702ae9d0-8d0c-4b8f-8dd7-bf4c91601b86, AccessDeniedException: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: ecs:DeregisterTaskDefinition on resource: * because no identity-based policy allows the ecs:DeregisterTaskDefinition action

---

###【要約】

エラー: ECSタスク定義 (ai-content-platform-api) の削除に失敗しました。

操作エラー: ECS:DeregisterTaskDefinition。

HTTPレスポンスエラー (ステータスコード: 400、リクエストID: 702ae9d0-8d0c-4b8f-8dd7-bf4c91601b86)。

AccessDeniedException: ユーザー `arn:aws:iam::[ユーザID]:user/[ユーザ名]` は、リソース `*` に対して

`ecs:DeregisterTaskDefinition` アクションを実行する権限がありません。

これは、`ecs:DeregisterTaskDefinition` アクションを許可するIDベースのポリシーが存在しないためです。

---

### 【原因】

ECSタスク定義の登録解除権限不足

---

###【対応方法】

#### 1. 許可ポリシーTerraform-Apply-Computeに以下の行を追加

```hcl
        {
            "Sid": "TerraformApplyComputeEcs",
            "Effect": "Allow",
            "Action": [
                "ecs:CreateCluster",
                "ecs:DescribeClusters",
                "ecs:UpdateCluster",
                "ecs:DeleteCluster",
                "ecs:DeregisterTaskDefinition",　←この行のみ追加
                "ecs:TagResource"
            ],
            "Resource": "*"
        },
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
