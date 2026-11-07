## Trouble on Making_19
---
### Message

Error: reading ELBv2 Target Group (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17) attributes: operation error Elastic Load Balancing v2: DescribeTargetGroupAttributes, https response error StatusCode: 403, RequestID: 50a3f70f-ce59-4dcf-bb03-d167777266a0, api error AccessDenied: User: arn:aws:iam::[ユーザID]:user/[ユーザ名] is not authorized to perform: elasticloadbalancing:DescribeTargetGroupAttributes because no identity-based policy allows the elasticloadbalancing:DescribeTargetGroupAttributes action.
（再度、同じエラーが出た）

---

###【要約】

エラー: ELBv2 ターゲットグループ (arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17) の属性を読み込み中にエラーが発生しました。

Elastic Load Balancing v2 の操作「DescribeTargetGroupAttributes」の実行に失敗しました。

HTTP レスポンスエラー (ステータスコード: 403、リクエストID: 50a3f70f-ce59-4dcf-bb03-d167777266a0)。

API エラー「AccessDenied」: ユーザー (arn:aws:iam::[ユーザID]:user/[ユーザ名]) は「elasticloadbalancing:DescribeTargetGroupAttributes」アクションを実行する権限がありません。

このアクションを許可するアイデンティティベースのポリシーが存在しないためです。

---

### 【原因】

elasticloadbalancing:DescribeTargetGroupAttributes」アクションを実行する権限が不足している。

再度同じエラーが出た原因の核心は、

```text
not authorized to perform:
elasticloadbalancing:DescribeTargetGroupAttributes
```

つまり、

```text
Terraform
  ↓
AWS Provider
  ↓
ELBv2 DescribeTargetGroupAttributes
  ↓
AWS IAM
  ↓
Kn1sh1kaw4
  ↓
許可されていない
  ↓
403 AccessDenied
```

という状態である。

```text
not authorized to perform:
elasticloadbalancing:DescribeTargetGroupAttributes
```

つまり、

```text
Terraform
  ↓
AWS Provider
  ↓
ELBv2 DescribeTargetGroupAttributes
  ↓
AWS IAM
  ↓
Kn1sh1kaw4
  ↓
許可されていない
  ↓
403 AccessDenied
```

という状態です。

これは、

```text
Terraformコード
        ×
AWSネットワーク
        ×
Target Groupそのもの
        ×
```

ではなく、**IAM認可の問題**です。

---

# 2. 「ポリシーに追記したのに、なぜまだ403なのか」

ここが今回一番重要です。

「Terraform-Apply-Networkに」

```json
"elasticloadbalancing:DescribeTargetGroupAttributes"
```

を追加しただけでは、

**そのポリシーが現在Terraformを実行しているIAMユーザーに実際に適用されていなければ効果がありません。**

例えば、

```text
Terraform-Apply-Network
        ↓
IAMポリシーを編集
        ↓
保存
```

まで行っていても、

```text
[IAMユーザ]
        ↑
Terraform-Apply-Network
```

というアタッチ状態でなければ、Terraformからは403になります。

---


###【対応方法】

#### 1. まずAWS CLIで「本当に許可されているか」を確認

PowerShellで以下を実行してください。

```
powershell aws iam simulate-principal-policy --policy-source-arn arn:aws:iam::[ユーザID]:user/[ユーザ名] --action-names elasticloadbalancing:DescribeTargetGroupAttributes --resource-arns "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17"
```

結果の中に、

```text
EvalDecision
```

がある。

### 理想

```text
allowed
```

### 問題あり

```text
implicitDeny
```

または

```text
explicitDeny
```

である。

これによって、

> 「ポリシーを書き間違えた」のか
> 「ポリシーは正しいがユーザーに付いていない」のか
> 「別のDenyに阻まれている」のか

を切り分けられる。

#### 2. PowerShellで以下を実行する。

```powershell
aws iam simulate-principal-policy `
  --policy-source-arn arn:aws:iam::[ユーザID]:user/[ユーザ名] --action-names elasticloadbalancing:DescribeTargetGroupAttributes --resource-arns "arn:aws:elasticloadbalancing:[リージョン]:[ユーザID]:targetgroup/ai-content-platform-app-tg/2997253728b1cf17"
```

結果の中に、

```text
EvalDecision
```

がある。

### 理想

```text
allowed
```

### 問題あり

```text
implicitDeny
```

または

```text
explicitDeny
```

である。

これによって、

> 「ポリシーを書き間違えた」のか
> 「ポリシーは正しいがユーザーに付いていない」のか
> 「別のDenyに阻まれている」のか

を切り分けられ


elasticloadbalancing:DescribeTargetGroupAttributes

```
terraform fmt
terraform validate
terraform plan -out=tfplan
terrafrom show -no-color tfplan > plan.txt
plan.txt確認
"0 to destroy"を確認
terraform apply tfplan
```

2. 再度、terraform plan -out=tfplanを実行しても同一のエラーが発生

IAMポリシーへ追記したにもかかわらず、AWS側でその権限が実際に有効になっていない**ことを意味すると推察。
（IAMユーザに、追加した権限がまだ届いていない可能性がある）

以下4点を確認する。

・ `DescribeTargetGroupAttributes` を追加したポリシーが本当に保存されたか
・ そのポリシーが [IAMユーザ] にアタッチされているか
・ [IAMユーザ] が実際に使用している認証情報がそのユーザーのものか
・ 明示的DenyやPermissions Boundaryなどで拒否されていないか



#### 3. それでも403なら、次に疑うもの

もしポリシーにも存在し、ユーザーにもアタッチされているのに、

```text
AccessDenied
```

が続く場合は、次の段階に進む。

### ① Permissions Boundary

IAMユーザーにPermissions Boundaryが設定されている場合、通常のAllowを追加しても、そのBoundaryによって制限される可能性がある。

確認：

```powershell
aws iam get-user --user-name [IAMユーザ名]
```

出力に、

```text
PermissionsBoundary
```

があるか確認する。

---

### ② AWS Organizations / SCP

AWS Organizationsを利用している場合は、Service Control Policy（SCP）による明示的な制限も候補になります。

ただし、今回の個人AWSアカウント環境でOrganizationsを利用していなければ、この可能性は低いです。

---

### ③ 別のAWS認証情報でTerraformを実行している

これも非常に重要である。

Terraformが本当に、

```text
arn:aws:iam::[ユーザID]:user/[IAMユーザ名]
```

としてAWSへアクセスしているか確認する。

```powershell
aws sts get-caller-identity
```

期待する結果は、

```text
Arn:
arn:aws:iam::[ユーザID]:user/[IAMユーザ名]
```

である。

**ここが違っていたら、IAMポリシーを修正してもTerraformには効かない。**

そしてAWSコンソールで、

### `Terraform-Apply-Network`（インライン）

と

### `Terraform-Apply-Network`（グループ経由）

の**両方のJSON**を確認する。

さらに重要な確認：Permissions Boundary

AWSコンソールの

```text
IAM
→ Users
→ [IAMユーザ名]
```

で、

**Permissions boundary**

が設定されていないか確認してください。

もし設定されていたら、今回の403の有力候補になります。

例えば、

```text
Identity-based policy
    ↓
Allow DescribeTargetGroupAttributes
    ↓
Permissions Boundary
    ↓
許可されていない
```

という状態があり得る。





### ④

```powershell
aws iam list-attached-user-policies --user-name [IAMユーザ名]
```

→ `Terraform-Apply-Network` があるか確認。


### ⑤

`Terraform-Apply-Network` の実体を確認。

→ `DescribeTargetGroupAttributes` が入っているか確認。


### ⑥

可能なら、

```powershell
aws iam simulate-principal-policy ...
```

→ `allowed` を確認。


### ⑦

それから、

```powershell
terraform plan -out=tfplan
```

