# 5.2.1 · JFrog CLI

`jf`（JFrog CLI）是跟 Artifactory 做 authenticate、然后 upload 东西的通用命令行工具。
Generic binary 和 Terraform module 这两类首选就是它（Docker 和 Helm 有自己原生的 client，
接下来两页讲）。

## 配置一个连接

```bash
jf rt config --url=https://myregistry.jfrog.io/artifactory \
              --user=$ART_USER \
              --password=$ART_PASS \
              --interactive=false
```

在 Jenkins pipeline 里，`$ART_USER`/`$ART_PASS` 应该来自 `withCredentials`
（见 [2.8 · Credentials](../module-2-cicd-jenkins/2-8-credentials.md)）——绝不能写死。

## Upload 一个 generic artifact

```bash
jf rt upload sample-cli-1.2.0.tar.gz generic-local/tools/sample-cli/1.2.0/
```

语法：`jf rt upload <本地文件 pattern> <repo>/<destination path>/`。这个 destination path
完全由你自己定义 convention——怎么设计一个统一、好查的 convention，见
[5.3](5-3-destination-path-design.md)。

## 带通配符的 upload

```bash
jf rt upload "dist/*.tar.gz" generic-local/tools/sample-cli/1.2.0/
```

一次 build 产出好几个文件（checksum、好几个平台的 build 产物）、都要落在同一个
destination 文件夹的时候很好用。

## 验证 upload 有没有成功

```bash
jf rt search "generic-local/tools/sample-cli/1.2.0/*"
```

返回一个 JSON，列出这个 path 下实际存在的东西——这是在 pipeline 报 success 之前，确认
upload 真的成功了的标准做法。

!!! warning "常见坑：结尾有没有斜杠很重要"
    `jf rt upload file.tar.gz generic-local/tools/x`（没有结尾斜杠）可能会被理解成
    "upload 成一个字面上叫 `x` 的文件"，而 `generic-local/tools/x/`（有结尾斜杠）是 upload
    *到*一个叫 `x` 的文件夹*里*。目标是个文件夹的时候，结尾斜杠一定要带上。

## Try it yourself

1. 写出把 `sample-cli-2.0.0-linux.tar.gz` 发布到
   `generic-local/tools/sample-cli/2.0.0/` 的 `jf rt upload` command。
2. 写出对应的 `jf rt search` command，验证它是不是真的落在那了。

!!! tip "How this applies to the capstone"
    [7.2 · Downstream Job Branching](../module-7-capstone/7-2-downstream-job-branching.md)
    里 "generic binary" 那条分支跑的就是这条 command；Terraform module 那条分支（见
    [5.2.4](5-2-4-terraform-modules.md)）用的也是它，只是 repo 类型的参数不一样。
