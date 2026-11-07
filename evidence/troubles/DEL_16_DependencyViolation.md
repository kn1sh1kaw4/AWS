## Trouble on Dellition_16
---
### Message

Error: deleting EC2 Internet Gateway (igw-[ゲートウェイID]): detaching EC2 Internet Gateway (igw-[ゲートウェイID]) from VPC (vpc-[VPC-ID]): operation error EC2: DetachInternetGateway, https response error StatusCode: 400, RequestID: 9f4ec8f6-d483-4b1f-9f7e-aefe138ebc93, api error DependencyViolation: Network vpc-[VPC-ID] has some mapped public address(es). Please unmap those public address(es) before detaching the gateway.

---

###【要約】

エラー: EC2インターネットゲートウェイ (igw-[ゲートウェイID]) の削除中にエラーが発生しました。VPC (vpc-[VPC-ID]) から当該インターネットゲートウェイを切り離そうとしましたが、失敗しました。操作エラー (EC2: DetachInternetGateway)、HTTPレスポンスエラー (ステータスコード: 400、リクエストID: 9f4ec8f6-d483-4b1f-9f7e-aefe138ebc93)、APIエラー (DependencyViolation): 

VPC (vpc-[VPC-ID]) にパブリックアドレスが割り当てられたままになっています。

ゲートウェイを切り離す前に、これらのパブリックアドレスの割り当てを解除してください。


---

### 【原因】

VPCがパブリックアドレスが割り当てられたままになっている。

ALBそのものの削除がIAM権限不足で失敗しているということです。

ALBが残存しているため

ALB管理のENI・EIP

eni-02ffd484ce97588c2 / 15.168.156.35

残存している可能性が高い

パブリックアドレスが残存

Internet Gateway

切り離し失敗：DependencyViolation

したがって、前回と同じエラー②が発生したのは、ALBの削除が完了していないことが有力な理由です。

ただし、もう一つ考慮すべき点があります。

Terraformは、依存関係のないリソースの削除を並行して実行することがあります。ALBとInternet Gatewayの削除が同じapply内で並行して実行されると、ALBの削除が完了する前にInternet Gatewayの切り離しが試みられる可能性もあります。

そのため、今回はALB関連リソースを先に削除し、その後に残りのAWS環境を削除する二段階方式を推奨します。


###【対応方法】

#### 1. IAMポリシーの修正

まず、`Terraform-Apply-Balancer` に以下の権限を追加・確認してください。

- `elasticloadbalancing:DeleteLoadBalancer`
- `elasticloadbalancing:DeleteTargetGroup`（API用）
- `elasticloadbalancing:DeleteTargetGroup`（APP用）
- `elasticloadbalancing:DeleteListener`
- `elasticloadbalancing:DeleteRule`

ポリシーのJSON構文が正しいこと、IAMユーザーにアタッチされていることも確認してください。


#### 2. ALB関連リソースだけを先に削除

今回は、通常の全体destroyではなく、ALB関連リソースを対象にしたdestroy planを作成する方法を提案します。

まず、Terraform stateに残っているALB関連リソースを確認してください。

```
terraform state list | Select-String "aws_lb"
```

現在のstateに残っているリソースを確認したうえで、以下のコマンドを実行します。

```
terraform state pull > terraform-state-before-alb-destroy.json

terraform plan -destroy `
  -target=aws_lb_listener_rule.api `
  -target=aws_lb_listener.http `
  -target=aws_lb_target_group.api `
  -target=aws_lb_target_group.app `
  -target=aws_lb.app `
  -out="alb-destroy.tfplan"
```

※すでにstateから削除されているリソースがあれば、その対象は除いてください。

続いて、内容をテキスト化します。

```
terraform show -no-color alb-destroy.tfplan > alb-destroy-plan.txt
```

`alb-destroy-plan.txt` を開き、削除対象がALB関連に限定されていることを確認してください。

問題がなければ実行します。

```
terraform apply alb-destroy.tfplan
```

`-target` は通常のTerraform運用では常用しない方がよいオプションですが、今回のように依存関係のない削除処理を段階的に進めたい場合には、対象を限定して実行する方法として利用できます。


#### 3. ALB・EIP・ENIの削除確認

ALBの削除後、以下を実行してください。

```
aws elbv2 describe-load-balancers `
  --region ap-northeast-3
```

```
aws ec2 describe-network-interfaces `
  --filters "Name=vpc-id,Values=vpc-[VPC-ID]" `
  --region ap-northeast-3
```

```
aws ec2 describe-addresses `
  --region ap-northeast-3
```

確認したいのは次の3点です。

| 確認対象                    | 期待する結果                  |
| ----------------------- | ----------------------- |
| ai-content-platform-alb | 削除されている                 |
| ALB管理のENI               | 削除されている                 |
| EIP `15.168.156.35`     | 関連付けがなくなり、不要であれば解放されている |

AWS側の削除処理には少し時間がかかる場合があります。ALBの削除直後にENIが残っていても、少し待ってから再確認してください。


#### 4. 残りのAWS環境を削除

ALB関連リソースが正常に削除されたことを確認した後、全体のdestroy planを作り直します。

```
terraform plan -destroy -out="destroy-final.tfplan"

terraform show -no-color destroy-final.tfplan > destroy-final-plan.txt
```

内容を確認し、問題がなければ実行します。

```
terraform apply destroy-final.tfplan
```

これで、Internet Gatewayの切り離し時にALB管理のパブリックIPが残っているという問題を回避できる可能性が高くなります。


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

