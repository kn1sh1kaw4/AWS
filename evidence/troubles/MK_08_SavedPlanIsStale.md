## Trouble on Making_08
---
### Message

Error: Saved plan is stale
 The given plan file can no longer be applied because the state was changed by another operation after the plan was created.
---

###【要約】

エラー: 保存されたプランが古くなっています

指定されたプランファイルは、作成された後に別の操作によって状態が変更されたため、適用できなくなっています。

---

### 【原因】

保存されたプランが古くなっていて、現在の設定と合っていない

---


###【対応方法】

#### 1. プランの再作成

```hcl
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform show -no-color tfplan > plan.txt
terraform apply tfplan
```

