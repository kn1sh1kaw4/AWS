## Trouble on Making_17
---
### Message

Error: Resource instance is not tainted

Resource instance aws_cloudtrail.main is not currently tainted, and so it cannot be untainted.

---

###【要約】

エラー: リソースインスタンスは tainted 状態ではありません

リソースインスタンス `aws_cloudtrail.main` は現在 tainted 状態ではないため、tainted 状態を解除することはできません。

---

### 【原因】

untaintする対象がすでにtaint状態ではなかった

今回、

```text
Error: Resource instance is not tainted
Resource instance aws_cloudtrail.main is not currently tainted,
and so it cannot be untainted.
```

となっています。

つまり、

```text
aws_cloudtrail.main
        ↓
現在のTerraform state
        ↓
taintedではない
```

ということです。

---


###【対応方法】

#### 1. 現在のTerraform stateを基準に新しいplanを作り直す

```
terraform fmt
terraform validate
terraform plan -out=tfplan
terrafrom show -no-color tfplan > plan.txt
plan.txt確認
"0 to destroy"を確認
terraform apply tfplan
```

