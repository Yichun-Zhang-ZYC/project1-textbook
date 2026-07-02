# 0.3 · Artifact 类型

Upload 这一步具体怎么做，取决于 upload 的到底是什么。这个网站的 running example 里一共有四种
artifact 类型。这一页先过一遍大概，具体到每条 command 的细节留到
[Module 5](../module-5-artifactory/5-1-repository-types.md) 再展开。

| Artifact 类型 | 举例 | 用什么 upload | Artifactory repo 类型 |
|---|---|---|---|
| Generic binary / tarball | `sample-cli`（一个独立的 CLI 工具） | JFrog CLI（`jf rt upload`） | Generic |
| Terraform module | `terraform-aws-network-module` | JFrog CLI 或者 `terraform login` + `publish` | Terraform |
| Docker image | `sample-web-app:1.4.0` | `docker push`（Artifactory 当 registry 用） | Docker |
| Helm chart | `sample-chart-2.13.6.tgz` | `helm push`（或者 `jf rt upload`） | Helm |

## 为什么这对自动化很重要

一条 "upload 这个 artifact" 的 pipeline 不能只跑一条 command 了事——得先按 artifact 类型分支，
因为每种类型：

- authenticate 的方式不一样（Docker 用 registry login；JFrog CLI 用自己的 token），
- destination path 的 convention 不一样（见
  [5.3 · Destination Path 设计](../module-5-artifactory/5-3-destination-path-design.md)），
- 判断 "成功了没" 的方式也不一样（Docker 看 manifest digest，Helm 看 chart index 里有没有条目，
  其他的看 file checksum）。

```bash
# Generic binary
jf rt upload sample-cli-1.2.0.tar.gz generic-local/tools/sample-cli/1.2.0/

# Docker image
docker push myregistry.jfrog.io/docker-local/sample-web-app:1.4.0

# Helm chart
helm push sample-chart-2.13.6.tgz oci://myregistry.jfrog.io/helm-local/

# Terraform module
jf rt upload terraform-aws-network-module-1.0.0.zip terraform-local/aws-network/1.0.0/
```

!!! tip "How this applies to the capstone"
    [Module 7](../module-7-capstone/7-2-downstream-job-branching.md) 里那个 downstream upload
    job，本质上就是照着这张表写的一个 `switch` 语句：给一个 artifact 类型的 parameter，挑出对的
    upload command 和 destination convention。
