## Trouble on Dellition_13
---
### Message

Error: deleting EC2 Internet Gateway (igw-[Internet Gateway ID]): detaching EC2 Internet Gateway (igw-0a0f40ba4f4acfebc) from VPC (vpc-[VPC-ID]): operation error EC2: DetachInternetGateway, https response error StatusCode: 400, RequestID: a6f01068-0b9a-4d5d-943b-1309d5f61d86, api error DependencyViolation: Network vpc-[VPC-ID] has some mapped public address(es). Please unmap those public address(es) before detaching the gateway.

---

###【要約】

エラー: EC2インターネットゲートウェイ (igw-0a0f40ba4f4acfebc) の削除中に問題が発生しました。

VPC (vpc-[VPC-ID]) から当該インターネットゲートウェイを切り離そうとしましたが、以下のエラーが返されました。

操作エラー: EC2: DetachInternetGateway、HTTPレスポンスエラー (ステータスコード: 400、リクエストID: a6f01068-0b9a-4d5d-943b-1309d5f61d86)。

APIエラー: DependencyViolation - VPC (vpc-[VPC-ID]) にパブリックアドレスが割り当てられています。

ゲートウェイを切り離す前に、これらのパブリックアドレスの割り当てを解除してください。

---

### 【原因】

削除対象がパブリックアドレスが割り当てられている。
　↓
今回のTerraform環境では、過去の構成から考えると、主に次の可能性があります。

| 原因候補            | 内容                                 | 対応                    |
| --------------- | ---------------------------------- | --------------------- |
| Elastic IP（EIP） | EIPがVPC内のEC2などに関連付けられている           | EIPの関連付け解除            |
| NAT Gateway     | NAT GatewayにEIPが割り当てられている          | NAT Gatewayの削除・EIPの解放 |
| EC2等のパブリックIP    | EC2インスタンス等にパブリックIPv4アドレスが割り当てられている | 対象リソースの削除・IPの解除       |

---

###【対応方法】

#### 1. AWSマネジメントコンソールで、以下の手順で現状を確認する。

① Elastic IP アドレス

AWSコンソールから、

`EC2 → ネットワーク＆セキュリティ → Elastic IP`

を開きます。

対象VPCに関連するEIPが残っていないか確認してください。

特に、以下の情報を確認します。

- パブリックIPv4アドレス
- 割り当てID（Allocation ID）
- 関連付けID（Association ID）
- 関連付け先のインスタンスIDまたはネットワークインターフェイスID

② NAT Gateway

`VPC → NAT Gateway`

を開き、対象VPCにNAT Gatewayが残っていないか確認します。

NAT Gatewayが残っている場合、削除が完了するまでEIPが保持されることがあります。

③ ネットワークインターフェイス

`EC2 → ネットワーク＆セキュリティ → ネットワークインターフェイス`

を開き、対象VPCに残っているネットワークインターフェイスを確認してください。

特に、パブリックIPアドレスが割り当てられているものがないか確認します。

現時点では、手動でEIPを解除・解放する前に、Terraformの管理状態とAWS上の実リソースを照合することを推奨します。


#### 2. 対象VPCに残っているNAT Gatewayを確認する。

```
aws ec2 describe-nat-gateways --filter "Name=vpc-id,Values=vpc-[VPC ID]" --region [リージョン]
```

確認する項目：

- `NatGatewayId`
- `State`
- `NatGatewayAddresses`
- `AllocationId`
- `PublicIp`

　↓
```
{
    "NatGateways": []
}
```

（※確認する項目はなし）


```
aws ec2 describe-network-interfaces --filters "Name=vpc-id,Values=vpc-[VPC ID]" --region [リージョン]

{
    "NetworkInterfaces": [
        {
            "Association": {
                "AllocationId": "eipalloc-0a8b0f01e89a2fb3f",
                "AssociationId": "eipassoc-0576d2dc1dfba96e4",
                "IpOwnerId": "[ユーザID]",
                "PublicDnsName": "ec2-15-168-156-35.[リージョン].compute.amazonaws.com",
                "PublicIp": "[IPアドレス]"
            },
            "Attachment": {
                "AttachTime": "2026-10-29T11:49:35+00:00",
                "AttachmentId": "eni-attach-08b69f1a24ed2b139",
                "DeleteOnTermination": false,
                "DeviceIndex": 1,
                "NetworkCardIndex": 0,
                "InstanceOwnerId": "amazon-elb",
                "Status": "attached"
            },
            "AvailabilityZone": "[リージョン]a",
            "Description": "ELB app/ai-content-platform-alb/d8edb03ceac706aa",
            "Groups": [
                {
                    "GroupId": "sg-[Security Group ID]",
                    "GroupName": "ai-content-platform-alb-sg"
                }
            ],
            "InterfaceType": "interface",
            "Ipv6Addresses": [],
            "MacAddress": "0e:ff:e6:05:93:df",
            "NetworkInterfaceId": "eni-02ffd484ce97588c2",
            "OwnerId": "[ユーザID]",
            "PrivateDnsName": "ip-10-0-1-77.[リージョン].compute.internal",
            "PublicDnsName": "ec2-15-168-156-35.[リージョン].compute.amazonaws.com",
            "PublicIpDnsNameOptions": {
                "DnsHostnameType": "public-ipv4-dns-name",
                "PublicIpv4DnsName": "ec2-15-168-156-35.[リージョン].compute.amazonaws.com"
            },
            "PrivateIpAddress": "10.0.1.77",
            "PrivateIpAddresses": [
                {
                    "Association": {
                        "AllocationId": "eipalloc-0a8b0f01e89a2fb3f",
                        "AssociationId": "eipassoc-0576d2dc1dfba96e4",
                        "IpOwnerId": "[ユーザID]",
                        "PublicDnsName": "ec2-15-168-156-35.[リージョン].compute.amazonaws.com",
                        "PublicIp": "15.168.156.35"
                    },
                    "Primary": true,
                    "PrivateDnsName": "ip-10-0-1-77.[リージョン].compute.internal",
                    "PrivateIpAddress": "10.0.1.77"
                }
            ],
            "RequesterId": "amazon-elb",
            "RequesterManaged": true,
            "SourceDestCheck": true,
            "Status": "in-use",
            "SubnetId": "subnet-[サブネットID]",
            "TagSet": [],
            "VpcId": "vpc-[VPC-ID]",
            "Operator": {
                "Managed": false,
                "HiddenByDefault": false
            },
            "AvailabilityZoneId": "apne3-az3"
        }
    ]
}
```


#### 3. Elastic IPの確認

```
aws ec2 describe-addresses --region [リージョン]

{
    "Addresses": [
        {
            "AllocationId": "eipalloc-0a8b0f01e89a2fb3f",
            "AssociationId": "eipassoc-0576d2dc1dfba96e4",
            "NetworkInterfaceId": "eni-02ffd484ce97588c2",
            "ServiceManaged": "alb",
            "PublicIp": "[IPアドレス]"
        }
    ]
}
```

確認する項目：

- `PublicIp`
- `AllocationId`
- `AssociationId`
- `NetworkInterfaceId`
- `InstanceId`

※アカウント全体のEIPが表示されます。対象VPCに関連するものを確認してください。

　↓

調査結果によって対応が変わります。

- NAT Gatewayが残っている場合：NAT Gatewayの削除状態を確認します。
- EIPが関連付けられている場合：関連付け先を確認して解除します。
- EC2等のネットワークインターフェイスにパブリックIPが残っている場合：そのリソースの所有者・用途を確認して対処します。

　↓

ネットワークインターフェイスの情報に、次の記載があります。

```
"Description": "ELB app/ai-content-platform-alb/d8edb03ceac706aa",
"InstanceOwnerId": "amazon-elb",
"RequesterId": "amazon-elb",
"RequesterManaged": true
```

これは、AWSが管理するALBのネットワークインターフェイスであることを示しています。

さらに、次の情報があります。

| 項目                 | 確認結果                       |
| ------------------ | -------------------------- |
| ALB名               | ai-content-platform-alb    |
| ALBのネットワークインターフェイス | eni-02ffd484ce97588c2      |
| EIP                | 15.168.156.35              |
| EIP Allocation ID  | eipalloc-0a8b0f01e89a2fb3f |
| EIP Association ID | eipassoc-0576d2dc1dfba96e4 |
| VPC                | vpc-[VPC-ID]      |
| NAT Gateway        | なし                         |

特に重要なのは、EIPの情報にある次の項目です。

```
"ServiceManaged": "alb"
```

これは、EIPがALBによって管理されていることを示しています。

Application Load Balancer（ALB）に関連付けられたElastic IP（EIP）が残っていることが原因だと判明。

したがって、EIPを直接解除・解放するのではなく、ALBを正常に削除することが先決です。


#### 4. IAM権限を追加

   `DeleteTargetGroup` と `DeleteListener` を許可するポリシーを設定・アタッチします。

```
{
    "Sid": "TerraformApplyDelTgg",
    "Effect": "Allow",
    "Action": [
        "elasticloadbalancing:DeleteTargetGroup"
    ],
    "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-api-tg/971db7989cc7da6b"
},
{
    "Sid": "TerraformApplyDelListener",
    "Effect": "Allow",
    "Action": [
        "elasticloadbalancing:DeleteListener"
    ],
    "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c"
}
```

IAMユーザーにポリシーがアタッチされていることも確認してください。

#### 5. 最新のTerraform destroy planを作成

   現在のstateとAWS環境を反映した削除計画を作成します。

```
{
    "Sid": "TerraformApplyDelTgg",
    "Effect": "Allow",
    "Action": [
        "elasticloadbalancing:DeleteTargetGroup"
    ],
    "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-api-tg/971db7989cc7da6b"
},
{
    "Sid": "TerraformApplyDelListener",
    "Effect": "Allow",
    "Action": [
        "elasticloadbalancing:DeleteListener"
    ],
    "Resource": "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:listener/app/ai-content-platform-alb/d8edb03ceac706aa/2187a020d8629d3c"
}
```

IAMユーザーにポリシーがアタッチされていることも確認してください。


#### 6. planの確認とapply

以下を確認してください。

- ALB関連リソースが削除対象になっているか
- 予期しないリソースの追加・変更がないか
- S3バケットやDynamoDBなど、削除してよいデータリソースだけが削除対象になっているか


#### 6. ALB関連リソースを削除

   問題がなければリスナー、ターゲットグループ、ALBの順に依存関係に従って削除します。

```
terraform apply destroy-retry-3.tfplan
```


#### 7. EIPとネットワークインターフェイスを再確認

   ALBの削除完了後、関連するEIP・ENIが消えているか確認します。

apply後、以下を再実行してください。

```
aws ec2 describe-network-interfaces `
  --filters "Name=vpc-id,Values=vpc-[VPC-ID]" `
  --region [リージョン]
```

```
aws ec2 describe-addresses `
  --region [リージョン]
```

ALBの削除が完了し、今回のEIPとENIが消えていることを確認します。


#### 8. Internet Gatewayの削除を確認

   パブリックIPの関連付けが解消されれば、IGWの切り離しが可能になると考えられます。
