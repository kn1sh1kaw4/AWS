## Trouble on Dellition_08
---
### Message

Error: emptying S3 Bucket (ai-content-platform-audit-8624ffafdb958ac5bf6f77dee7): listing S3 bucket (ai-content-platform-audit-8624ffafdb958ac5bf6f77dee7) object versions: operation error S3: ListObjectVersions, https response error StatusCode: 403, RequestID: 47YTXYRQQJ4HT70B, HostID: Ofx1/p13uQg9JjdevoHbyTG4giNqq1ZK+GedzjoQ4OqItFjtEXog9RExQbJa4fP1Ox/Gt1o6C1M=, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: s3:ListBucketVersions on resource: "arn:[ ARN ]" because no identity-based policy allows the s3:ListBucketVersions action.

---

###【要約】

エラー: S3バケット (ai-content-platform-audit-8624ffafdb958ac5bf6f77dee7) の空き化処理中にエラーが発生しました。

S3バケット (ai-content-platform-audit-8624ffafdb958ac5bf6f77dee7) 内のオブジェクトバージョン一覧を取得しようとした際、S3のListObjectVersions操作でエラーとなりました。

HTTPレスポンスエラー（ステータスコード: 403、RequestID: 47YTXYRQQJ4HT70B、HostID: Ofx1/p13uQg9JjdevoHbyTG4giNqq1ZK+GedzjoQ4OqItFjtEXog9RExQbJa4fP1Ox/Gt1o6C1M=）が発生し、APIエラー「AccessDenied」が返されました。

ユーザー「arn:aws:iam::[ユーザID]:user/[ユーザ名]」には、リソース「arn:aws:s3:::ai-content-platform-audit-8624ffafdb958ac5bf6f77dee7」に対して「s3:ListBucketVersions」アクションを実行する権限がありません。

これは、当該アクションを許可するIDベースのポリシーが存在しないためです。╷

---

### 【原因】

S3バージョン一覧取得権限不足

---

###【対応方法】

#### 1. Terraform-Apply-Storage-S3に以下のものを追加する。

```hcl
{
  "Sid": "TerraformDeleteStorageS3",
  "Effect": "Allow",
  "Action": [
    "s3:ListBucketVersions",
    "s3:DeleteObject",
    "s3:DeleteObjectVersion"
  ],
  "Resource": [
    "arn:[ ARN ]",
    "arn:[ ARN ]",
    "arn:[ ARN ]",
    "arn:[ ARN ]"
  ]
}

```

ここで3つのアクションの役割は次のとおりです。

| IAMアクション | 役割 |
|---|---|
| s3:ListBucketVersions | オブジェクトのバージョン一覧取得 |
| s3:DeleteObject | オブジェクト削除 |
| s3:DeleteObjectVersion | 特定バージョンの完全削除 |

バージョニングが有効なバケットでは、通常の削除操作だけでは旧バージョンが残る場合があります。Terraformでバケットを空にして削除するには、バージョン削除権限も必要になるため、併せて追加する。

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

