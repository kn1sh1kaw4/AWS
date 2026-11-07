## Trouble on Dellition_06
---
### Message

Error: deleting Backup Vault (ai-content-platform-dev-backup): operation error Backup: DeleteBackupVault, https response error StatusCode: 400, RequestID: 5892b720-22a7-4526-9caf-1cf9bf2780ac, InvalidRequestException: Backup vault cannot be deleted because it contains recovery points.

---

###【要約】

エラー: バックアップコンテナ (ai-content-platform-dev-backup) の削除に失敗しました。

操作エラー: Backup: DeleteBackupVault。

HTTP レスポンスエラー (ステータスコード: 400、リクエストID: 5892b720-22a7-4526-9caf-1cf9bf2780ac)。

InvalidRequestException: バックアップコンテナ内にリカバリポイントが含まれているため、削除できません。

---

### 【原因】

Backup Vaultに復旧ポイントが残存

---

###【対応方法】

#### 1. AWS Backup > ボールト > ai-content-platform-dev-backupの復旧ポイントにある復旧ポイントすべてにチェックが入った状態で削除する。

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
