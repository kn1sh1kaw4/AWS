## Trouble on Dellition_05
---
### Message

Error: deleting EC2 Internet Gateway (igw-0a0f40ba4f4acfebc): detaching EC2 Internet Gateway (igw-[ Internet Gateway ID ]) from VPC (vpc-[VPC-ID]): operation error EC2: DetachInternetGateway, https response error StatusCode: 400, RequestID: fc260748-bb00-4ca8-aa73-30b38ab3d22b, api error DependencyViolation: Network vpc-[VPC-ID] has some mapped public address(es). Please unmap those public address(es) before detaching the gateway.

---

###【要約】

エラー: EC2インターネットゲートウェイ (igw-[ Internet Gateway ID ]) の削除中に問題が発生しました。

VPC (vpc-[VPC-ID]) から当該インターネットゲートウェイを切り離そうとしましたが、以下のエラーが返されました: 

EC2: DetachInternetGateway 操作エラー。HTTPレスポンスステータスコード: 400、RequestID: fc260748-bb00-4ca8-aa73-30b38ab3d22b。

APIエラー: DependencyViolation。VPC (vpc-[VPC-ID]) にパブリックアドレスが割り当てられたままになっています。

ゲートウェイを切り離す前に、これらのパブリックアドレスの割り当てを解除してください。╷

---

### 【原因】

VPCにパブリックアドレスが残存

---

###【対応方法】

#### 1. 許可ポリシーTerraform-Apply-Networkに以下の行を追加

```hcl
{
  "Sid": "TerraformDisassociateAddress",
  "Effect": "Allow",
  "Action": "ec2:DisassociateAddress",
  "Resource": "*"
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

