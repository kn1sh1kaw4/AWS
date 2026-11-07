## Trouble on Dellition_09
---
### Message

Error: deleting IAM Role (ai-content-platform-backup-role): reading IAM Instance Profiles for Role (ai-content-platform-backup-role): operation error IAM: ListInstanceProfilesForRole, https response error StatusCode: 403, RequestID: d701156b-ea39-4355-b8af-2db9273e261c, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: iam:ListInstanceProfilesForRole on resource: role ai-content-platform-backup-role because no identity-based policy allows the iam:ListInstanceProfilesForRole action. Go to https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/authorization-details/9ayb2for43um34ilvaf6pimx4 for complete details, or call the GetRequestAuthorizationDetails API with the following authorization id: 9ayb2for43um34ilvaf6pimx4

---

###【要約】

IAMロール (ai-content-platform-lambda) の削除中に問題が発生しました。

ロール (ai-content-platform-lambda) に関連付けられたIAMインスタンスプロファイルを取得しようとした際、

IAM API `ListInstanceProfilesForRole` の呼び出しでエラーが発生しました。

HTTPステータスコード: 403、リクエストID: 51946189-a87a-4129-8456-11f391d0dd59。

APIエラー `AccessDenied`: ユーザー `arn:aws:iam::[ユーザID]:user/[ユーザ名]` には、IAMロール `ai-content-platform-lambda` に対して `iam:ListInstanceProfilesForRole` アクションを実行する権限がありません。

これは、このアクションを許可するIDベースのポリシーが存在しないためです。

詳細については、https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/authorization-details/4d4q4eqel37vtg0sf5gc79l7g を参照するか、認可ID `4d4q4eqel37vtg0sf5gc79l7g` を指定して `GetRequestAuthorizationDetails` APIを呼び出してください。╷

---

###【対応方法】

#### 1. 許可ポリシー "Terraform-Apply-Iam"に、（"EFFECT": "Allow", "Resource": "*"のものに、）"iam:ListInstanceProfilesForRole"を追記

```hcl
{
  "Sid": "TerraformListInstanceProfilesForRole",
  "Effect": "Allow",
  "Action": "iam:ListInstanceProfilesForRole",
  "Resource": "arn:aws:iam::[アカウントID]:role/*"
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

