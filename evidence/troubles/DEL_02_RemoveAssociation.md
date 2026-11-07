## Trouble on Dellition_02
---
### Message

Error: disassociating EC2 EIP (eipassoc-0a13da5fea10a2960): operation error EC2: DisassociateAddress, https response error StatusCode: 403, RequestID: a12f3d46-c152-4c39-ba75-8d28a7ae5e0d, api error UnauthorizedOperation: You are not authorized to perform this operation. User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: ec2:DisassociateAddress on resource: arn:aws:ec2:[リージョン]:[ユーザID]:*/* because no identity-based policy allows the ec2:DisassociateAddress action. Encoded authorization failure message: dZv6f3Vm3_XUCW-9bvrPCE-Ed74GLnu5_Mk5s4f443FzgoeO0gt7Ur0wa9iafv0rsDOevexjUpMZAx4Fk8nUMsgoT-YU5oVTWgtm9RCBwsYeDMxJs9E6OOr_MwNS7fKx66_UZH1MVf38Yz8IhK-emgS6A_aA0W_WEiZ0m-t14sceN8gu3u8jq9nk6fUQjqZvv0_BxvHUj6OQZxVpSANQKNviIZpgLSbEqdCZYT7hbhGYHcVgrvpwRd5LnLL458fKoDVMycpAH0olKisyFsFAo9OXaKnJtqlhNbzt-N_-iP4VQgx_k_z-9fZid3GnCuG3qjiRU5A2JYg2WeznGEYmwMfW7Mk79wNesLdImQh50RSxIGP2Sz8C3Vj-68zvtG7xbO5rZ5jhSz8VtGH6h3jv9ArhS4Irx8XapUzzB_bM2xTFdhd2ByExuAMRWaSiDgabpnkp-NIMOXrgdLsdAG2uEXI2wAmmG_tJNaftwZ0mBvftGU6ea83IstlwIqOqUmVurT8zfMucNw

---

###【要約】

エラー: EC2 EIP (eipassoc-0a13da5fea10a2960) の関連付け解除に失敗しました。

操作エラー EC2: DisassociateAddress。

HTTP レスポンスエラー (ステータスコード: 403、リクエストID: a12f3d46-c152-4c39-ba75-8d28a7ae5e0d)。

API エラー UnauthorizedOperation: この操作を実行する権限がありません。

ユーザー `arn:aws:iam::[ユーザID]:user/[ユーザ名]` には、リソース `arn:aws:ec2:[リージョン]:[ユーザID]:*/*` に対して

`ec2:DisassociateAddress` アクションを実行する権限がありません。

これは、`ec2:DisassociateAddress` アクションを許可するアイデンティティベースのポリシーが存在しないためです。

エンコードされた認証失敗メッセージ: dZv6f3Vm3_XUCW-9bvrPCE-Ed74GLnu5_Mk5s4f443FzgoeO0gt7Ur0wa9iafv0rsDOevexjUpMZAx4Fk8nUMsgoT-YU5oVTWgtm9RCBwsYeDMxJs9E6OOr_MwNS7fKx66_UZH1MVf38Yz8IhK-emgS6A_aA0W_WEiZ0m-t14sceN8gu3u8jq9nk6fUQjqZvv0_BxvHUj6OQZxVpSANQKNviIZpgLSbEqdCZYT7hbhGYHcVgrvpwRd5LnLL458fKoDVMycpAH0olKisyFsFAo9OXaKnJtqlhNbzt-N_-iP4VQgx_k_z-9fZid3GnCuG3qjiRU5A2JYg2WeznGEYmwMfW7Mk79wNesLdImQh50RSxIGP2Sz8C3Vj-68zvtG7xbO5rZ5jhSz8VtGH6h3jv9ArhS4Irx8XapUzzB_bM2xTFdhd2ByExuAMRWaSiDgabpnkp-NIMOXrgdLsdAG2uEXI2wAmmG_tJNaftwZ0mBvftGU6ea83IstlwIqOqUmVurT8zfMucNw

---

###【原因】

EIPの関連付け解除権限不足

---

###【対応方法】

#### 1. AWSの認可リファレンスでは、ec2:DisassociateAddress はElastic IPとネットワークインターフェイスを対象リソースとして扱います。

Service Authorization Reference

まずは動作確認を優先する場合、次のようにします。

```hcl
{
  "Sid": "TerraformDisassociateAddress",
  "Effect": "Allow",
  "Action": "ec2:DisassociateAddress",
  "Resource": "*"
}
```

より厳密に制限する場合は、対象EIPのAllocation IDを確認し、Elastic IPのARNに絞る方法があります。ただし、今回の操作でEC2側が評価する関連リソースも考慮する必要があります。

補足： 今回のエラー⑤にもつながっている可能性が高いです。EIPの関連付け解除に失敗したため、VPCにパブリックアドレスが残り、インターネットゲートウェイを切り離せなくなったと考えられます。

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


