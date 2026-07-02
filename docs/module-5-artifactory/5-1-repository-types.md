# 5.1 · Repository 类型

Artifactory 把一切都组织成 **repository**——命名好的 bucket，每一个都对应一种 artifact
类型。Docker image 塞不进 Terraform 的 repo；repo 类型决定了它期望收到什么协议/格式的东西。

## 三种 repository 角色

| 角色 | 用途 |
|---|---|
| **Local** | 你（或者你的组织）直接发布的 artifact——这个项目里 upload 的东西最终就落在这。 |
| **Remote** | 一个挡在外部 registry（比如 Docker Hub）前面的缓存代理——跟 upload 自己的 artifact 没关系。 |
| **Virtual** | 一个把好几个 local/remote repo 聚合起来的单一 URL——方便消费者用，不是拿来直接 upload 的。 |

## 按格式分的 repo 类型

| Repo 类型 | 期望的格式 | 对应哪种 artifact 类型（[0.3](../module-0-orientation/0-3-artifact-types.md)） |
|---|---|---|
| Generic | 任何你自己定义的文件/文件夹结构 | Binary、tarball |
| Docker | Docker registry API（image layer + manifest） | Docker image |
| Helm | Helm chart repo index（或者 OCI，新版本 Helm 支持） | Helm chart |
| Terraform | Terraform Registry Protocol（modules API） | Terraform module |

上面每种 repo 类型对应的命名 convention 大概是 `generic-local`、`docker-local`、
`helm-local`、`terraform-local` 这样——`-local` 这个后缀标记它是前面角色表里说的
local（可以往里 publish 的）repo。

## 为什么要这么拆

Docker 和 Helm 的 client 说的是各自特定的 wire protocol（Docker 的 registry API、Helm 的
chart index/OCI 格式）——Artifactory 必须*就是*那个协议本身，这些 client 才能正常工作。
Generic 和 Terraform repo 更接近 "带着已知 path convention 的结构化文件存储"，这也是为什么
generic upload 走的是灵活的 destination path（见
[5.3](5-3-destination-path-design.md)），而 Docker/Helm push 走的是各自 client 原生的
command。

!!! tip "How this applies to the capstone"
    Downstream upload job（[7.2](../module-7-capstone/7-2-downstream-job-branching.md)）
    里每一次 "该 upload 到哪个 repo" 的判断，本质上都可以归结成：查一下 artifact 类型，
    照这张表挑出对应的 repo 类型，然后按后面几页讲的 convention 去 upload。
