## Trouble on Dellition_17
---
### Message

Error: deleting ELBv2 Target Group (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17): operation error Elastic Load Balancing v2: DeleteTargetGroup, https response error StatusCode: 403, RequestID: 03d27af7-59e2-403c-b9fe-de53baca2a97, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: elasticloadbalancing:DeleteTargetGroup on resource: arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17 because no identity-based policy allows the elasticloadbalancing:DeleteTargetGroup action.

---

###【要約】

エラー: ELBv2 ターゲットグループ (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17) の削除に失敗しました。Elastic Load Balancing v2: DeleteTargetGroup 操作でエラーが発生しました。HTTP レスポンスエラー (ステータスコード: 403、リクエストID: 03d27af7-59e2-403c-b9fe-de53baca2a97)。

API エラー AccessDenied: ユーザー arn:aws:iam::[ユーザID]:user/[ユーザ名] には、

リソース arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17 に対して 

elasticloadbalancing:DeleteTargetGroup アクションを実行する権限がありません。

これは、このアクションを許可するアイデンティティベースのポリシーが存在しないためです。


---

### 【原因】

DeleteTargetGroup の権限不足


###【対応方法】

#### 1. IAMポリシーの修正

許可ポリシーに以下の権限を追記する。

```
{
    "Sid": "DeleteAlbAppTargetGroup",
    "Effect": "Allow",
    "Action": "elasticloadbalancing:DeleteTargetGroup",
    "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[VPC-ID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17"
}
```

#### 2. 残りのAWS環境を削除

全体のdestroy planを作り直します。

```
terraform plan -destroy -out="destroy-final.tfplan"

terraform show -no-color destroy-final.tfplan > destroy-final-plan.txt
```

内容を確認し、問題がなければ実行します。

```
terraform apply destroy-final.tfplan
```

これで、Internet Gatewayの切り離し時にALB管理のパブリックIPが残っているという問題を回避できる可能性が高くなります。


#### 3. 削除計画を再作成

```hcl
   terraform plan -destroy -out="destroy-retry.tfplan"
```
   
   内容をテキスト化します。

```hcl
   terraform show -no-color destroy-retry.tfplan > destroy-retry-plan.txt
```


#### 4. 新しい削除計画を確認

   - 既に削除されたリソースが対象から外れているか。
   - 残存するリソースがすべて削除対象になっているか。
   - 意図しないリソースが対象になっていないか。
   - 追加・変更が発生していないか。


#### 5. 新しい計画を適用
   
   問題がなければ実行します。

```hcl
   terraform apply destroy-retry[x].tfplan
```

※[x]:任意の数字（これまで使ったもの以外）


