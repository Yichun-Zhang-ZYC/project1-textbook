# 5.2.3 · Helm Push

Helm chart 是打包好（`.tgz`）的一堆 Kubernetes manifest + template。Artifactory 可以把它
当成经典的 Helm chart repository（index-based）来托管，也可以走 OCI（新版本 Helm push
chart 的方式，跟 Docker push image 几乎一模一样）。

## 打包一个 chart

```bash
helm package ./sample-chart --version 2.13.6
# 产出: sample-chart-2.13.6.tgz
```

## Push 到经典 Helm repo

```bash
curl -u "$ART_USER:$ART_PASS" -T sample-chart-2.13.6.tgz \
     "https://myregistry.jfrog.io/artifactory/helm-local/sample-chart-2.13.6.tgz"
```

Artifactory 里经典的 Helm repo，upload 的方式跟普通文件存储差不多（就是一个 HTTP PUT），
因为 Helm client 真正读的那个 "repo index" 是 Artifactory 自己根据 repo 里现有的内容生成/
维护的——不需要自己手动去 build 这个 index。

## 走 OCI Push（新版本 Helm，`helm push`）

```bash
helm registry login myregistry.jfrog.io --username "$ART_USER" --password "$ART_PASS"
helm push sample-chart-2.13.6.tgz oci://myregistry.jfrog.io/helm-local/
```

这是更现代的做法，跟 Docker 的 login/push 流程几乎一模一样——如果你的 Artifactory
instance 和 Helm 版本都支持 OCI，值得优先用这个。

## 验证

```bash
helm show chart oci://myregistry.jfrog.io/helm-local/sample-chart --version 2.13.6
```

!!! warning "常见坑：文件名里的版本号得跟 `Chart.yaml` 对上"
    `helm package` 的 `-2.13.6.tgz` 这个版本后缀是从 `--version` 来的，但如果
    `Chart.yaml` 自己的 `version:` 字段跟这个对不上，有些 Helm client 之后会拒绝或者警告
    这个已经发布的 chart。`--version` 和 `Chart.yaml` 要保持同步。

## Try it yourself

1. 写出把 `./sample-chart` 打包成 `3.0.0` 版本的 `helm package` command。
2. 写出对应的 OCI push command。

!!! tip "How this applies to the capstone"
    [7.2 · Downstream Job Branching](../module-7-capstone/7-2-downstream-job-branching.md)
    里 "Helm chart" 那条分支跑的就是这条 command。
