## Trouble on Making_07
---
### Message

Error: listing tags for CloudTrail Trail (arn:aws:cloudtrail:[リージョン]:[ユーザID]:trail/ai-content-platform-trail): operation error CloudTrail: ListTags, https response error StatusCode: 400, RequestID: 3581a5f3-6d0f-4435-b5e7-c4a1d7a28866, AccessDeniedException: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: cloudtrail:ListTags on resource: arn:aws:cloudtrail:[リージョン]:[ユーザID]:trail/ai-content-platform-trail because no identity-based policy allows the cloudtrail:ListTags action

---

###【要約】

エラー: CloudTrail Trail のタグのリスト (arn:aws:cloudtrail:[リージョン]:[ユーザID]:trail/ai-content-platform-trail): 

操作エラー CloudTrail: ListTags、https 応答エラー StatusCode: 400、RequestID: 3581a5f3-6d0f-4435-b5e7-c4a1d7a28866、

AccessDeniedException:ユーザー: arn:aws:iam::[ユーザID]:user/[ユーザ名] は、アイデンティティベースのポリシーが

Cloudtrail:ListTags アクションを許可していないため、

リソース: arn:aws:cloudtrail:[リージョン]:[ユーザID]:trail/ai-content-platform-trail で Cloudtrail:ListTags を実行する権限がありません。

---

### 【原因】

タグ情報を読み取る権限不足

TerraformがCloudTrail Trailの状態・タグ情報を確認しようとしています。

---


###【対応方法】

#### 1. 

```json
{
  "Effect": "Allow",
  "Action": [
    "cloudtrail:ListTags"
  ],
  "Resource": "arn:aws:cloudtrail:[リージョン]:[VPC-ID]:trail/ai-content-platform-trail"
}
```

こちらもTerraformでTrailの作成・更新を行うのであれば、`CreateTrail`、`UpdateTrail`、`StartLogging`、`StopLogging`、`DeleteTrail` など、実際に使用するアクションの確認が必要です。


#### 2. 修正後の作業手順

```hcl
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform show -no-color tfplan > plan.txt
terraform apply tfplan
```

