# 5.2.2 · Docker Push

Docker image 走的是 Docker 自己的 client 和 registry 协议——Artifactory 在这扮演一个
Docker-compatible 的 registry，所以 authenticate 和 push 用的是标准的 `docker` CLI，
不是 `jf`。

## Authenticate

```bash
echo "$ART_PASS" | docker login myregistry.jfrog.io --username "$ART_USER" --password-stdin
```

`--password-stdin` 能避免把密码直接写进 process 的参数列表里（不然可能会通过 shell
history 或者 `ps` 泄露出去）。

## Tag 然后 push

```bash
docker tag sample-web-app:1.4.0 myregistry.jfrog.io/docker-local/sample-web-app:1.4.0
docker push myregistry.jfrog.io/docker-local/sample-web-app:1.4.0
```

Docker image 在 push 之前必须**先 tag 上完整的 registry path**——`docker push` 不像
`jf rt upload` 那样能单独传一个 destination 参数。Registry、repo 名字、tag，全都编码在
image 的 tag 字符串里。

## Tag 的构成

```text
myregistry.jfrog.io / docker-local / sample-web-app : 1.4.0
     ^ registry host    ^ repo 名字    ^ image 名字   ^ tag
```

## 验证

```bash
docker manifest inspect myregistry.jfrog.io/docker-local/sample-web-app:1.4.0
```

确认这个 image 确实落地了，而且能被 pull，不需要真的下载整个 image。

!!! warning "常见坑：push 之前忘了重新 tag"
    `docker push sample-web-app:1.4.0`（没有 registry 前缀）默认会 push 到 Docker Hub，
    不是你的 Artifactory instance——copy-paste command 的时候很容易踩这个坑。永远 push
    带完整 `myregistry.jfrog.io/...` 前缀的 tag，不要 push 本地 build 时候用的那个短 tag。

## Try it yourself

1. 写出把 `sample-web-app:2.0.0` 发布到 `docker-local` 的 tag + push command。
2. 解释一下：如果不先 tag 就直接跑 `docker push sample-web-app:2.0.0`，会发生什么，
   为什么。

!!! tip "How this applies to the capstone"
    [7.2 · Downstream Job Branching](../module-7-capstone/7-2-downstream-job-branching.md)
    里 "Docker image" 那条分支跑的就是这条 command——四种 artifact 类型里，upload command
    长得跟其他几个真正不一样的，就是它。
