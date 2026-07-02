# 5.3 · Destination Path 设计

对于那些 path 比较灵活的 repo 类型（generic，Terraform 也算是一定程度上——见
[5.1](5-1-repository-types.md)），文件夹结构*你说了算*。一个设计得不好的 convention 会让
artifact 很难找、很难清理，也很难自动化。一个好的 convention，应该好到 pipeline 不用人填
表单，自己就能算出来。

## 一个还算合理的 convention

```text
<repo>/<category>/<artifact-name>/<version>/<filename>
```

套到四种 artifact 类型上：

```text
generic-local/tools/sample-cli/1.2.0/sample-cli-1.2.0-linux.tar.gz
terraform-local/aws-network-module/1.0.0/aws-network-module-1.0.0.zip
docker-local/sample-web-app:1.4.0                      # （基于 tag，不是文件夹）
helm-local/sample-chart-2.13.6.tgz                     # （扁平的，基于 index）
```

Docker 和 Helm 之所以不走这套文件夹 convention，是因为它们的 client 有自己的寻址方式
（tag，或者 chart 名字+版本）——见 [5.2.2](5-2-2-docker-push.md) 和
[5.2.3](5-2-3-helm-push.md)。

## 一个好的 convention 应该有的特性

| 特性 | 为什么重要 |
|---|---|
| **光靠 metadata 就能算出来** | Pipeline 应该能光靠 `(artifact_name, version, type)` 就算出完整的 destination path，不用人手动输入——这正是 Option C 里那个 `DESTINATION_PATH` parameter 一路带着走的东西。 |
| **可排序/可浏览** | 人偶尔也需要手动去 repo 里翻一翻；`name/version/` 这种嵌套结构好翻，一堆文件平铺在一起就不好翻。 |
| **不会互相覆盖** | Path 里带上版本号，意味着重新发布一个新版本，永远不会悄悄覆盖掉旧版本。 |
| **不同 artifact 类型之间保持一致** | 就算底层协议不一样（Docker/Helm vs. generic），*逻辑上*的寻址方式（名字 + 版本）也应该能对应得上，这样分支逻辑才能保持简单。 |

## Destination 到底是从哪来的，走一遍全流程

```mermaid
flowchart LR
    A["Intake：destination path\n作为 ticket 字段填进去"] --> B["传给 main build\n当一个 parameter"]
    B --> C["传给 downstream job\n当一个 parameter（2.7）"]
    C --> D["直接用在\nupload command 里（5.2.x）"]
```

这条链路上没有任何一步重新推导或者重新 query 这个 destination——它在 intake 的时候被决定
一次，然后一路带着走。这跟 [4.4](../module-4-jira/4-4-why-querying-is-expensive.md) 里
"把数据往前抓，别事后往回查" 是同一个原则，只是这里用在 path 数据上，而不是 approval
状态上。

!!! warning "常见坑：大小写或者分隔符不统一"
    同一个 artifact 的历史，如果一会用 `sample-cli`、一会用 `sample_cli`，或者一会用
    `1.2.0`、一会用 `v1.2.0`，就会在 registry 里被拆成好几条不一样的 path。定好一套
    convention，靠 pipeline 强制执行，不要只靠大家自觉遵守。

## Try it yourself

1. 给一个叫 `sample-tool`、版本 `4.5.1`、平台 `windows` 的 generic binary 设计
   destination path。
2. 想一下这个 path 里哪部分应该来自 build parameter，哪部分应该是 pipeline 自己写死的。

!!! tip "How this applies to the capstone"
    贯穿 [Module 2](../module-2-cicd-jenkins/2-5-build-parameters.md) 和
    [Module 7](../module-7-capstone/7-1-option-c-mock-implementation.md) 的那个
    `DESTINATION_PATH` parameter，正是这一页这套 convention 算出来的结果，原样传下去，
    没有中途被改动过。
