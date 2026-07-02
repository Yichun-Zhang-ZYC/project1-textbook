# 5.2.4 · Terraform Modules

Terraform module 是可复用、带版本号的 infrastructure-as-code 单元。Artifactory 能托管一个
私有的 Terraform module registry，让 `terraform` 能像从公共 registry 拉一样，从这里拉
module。

## 打包一个 module

Terraform module 本质上就是一堆 `.tf` 文件的目录。要发布它，先把这个目录 zip 起来：

```bash
zip -r terraform-aws-network-module-1.0.0.zip ./terraform-aws-network-module
```

## 用 JFrog CLI 发布

```bash
jf rt upload terraform-aws-network-module-1.0.0.zip \
     terraform-local/aws-network-module/1.0.0/aws-network-module-1.0.0.zip
```

Artifactory 的 Terraform repo 类型要求一个特定的 path convention
（`<module 名字>/<版本>/<module 名字>-<版本>.zip`），这样它内置的 Terraform Registry
Protocol endpoint 才能正确地把这个 module serve 回去。

## 消费一个已发布的 module（只是给点背景，跟 upload 这一步无关）

```hcl
module "network" {
  source  = "myregistry.jfrog.io/terraform-local/aws-network-module/generic"
  version = "1.0.0"
}
```

这是*消费者*会写的代码——之所以贴出来，是为了帮你理解为什么 publish 时候的 path
convention 这么重要：如果 upload 的 path 跟 registry protocol 期望的对不上，即便文件
确实存在 Artifactory 里，`terraform init` 也照样找不到这个 module。

!!! warning "常见坑：Terraform module 的 path convention 比 generic upload 严格得多"
    跟完全自由的 generic upload（[5.2.1](5-2-1-jfrog-cli.md)）不一样，Terraform repo 会
    解析 destination path 来 serve Registry Protocol——一个不符合 convention 的 path
    可能 upload 成功了，但对 `terraform init` 来说完全不可见。一定要用真实的
    `terraform init` 去验证这个 module，光看 `jf rt upload` 成功了不够。

## Try it yourself

1. 写出发布一个叫 `terraform-gcp-storage-module`、版本 `2.1.0` 的 module 的
   `jf rt upload` command。
2. 写出消费者会用来拉这个 module 的 `module "..." { source = ... }` block。

!!! tip "How this applies to the capstone"
    [7.2 · Downstream Job Branching](../module-7-capstone/7-2-downstream-job-branching.md)
    里 "Terraform module" 那条分支跑的就是这条 command——跟 generic binary 那个 case
    机制上很像，只是 path 要求更严格。
