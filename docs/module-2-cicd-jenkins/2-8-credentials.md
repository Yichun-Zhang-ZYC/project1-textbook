# 2.8 · Credentials

Pipeline 需要用到各种 secret——API token、密码、registry credential——但这些 secret **绝不能**
直接写死在 Jenkinsfile 里（这是纯文本，存在 Git 里，只要有 repo 权限的人都能看到）。

## Jenkins 怎么处理这个问题

Secret 只在 Jenkins 的 **credential store** 里存一份，每个都有个 ID。Pipeline 只引用这个
ID，永远不直接接触真实的 secret 值：

```groovy
stage('Upload') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'artifactory-upload-creds',
            usernameVariable: 'ART_USER',
            passwordVariable: 'ART_PASS'
        )]) {
            sh 'jf rt config --url=$REGISTRY_URL --user=$ART_USER --password=$ART_PASS'
        }
    }
}
```

`withCredentials` **只在这个 block 执行期间**把 secret 注入到 environment variable 里，
而且 Jenkins 会自动在 build log 里把这些值 mask 掉（显示成 `****`）。

## 常见的 credential 类型

| 类型 | 用来做什么 |
|---|---|
| Username/password | Basic auth（registry、部分 API） |
| Secret text | 单个 token（比如 Jira 或 Artifactory 的 API token） |
| SSH key | 通过 SSH 做 Git 操作 |
| Certificate | mTLS 或者签过名的 artifact workflow |

## 引用一个 secret token

```groovy
withCredentials([string(credentialsId: 'jira-api-token', variable: 'JIRA_TOKEN')]) {
    sh 'curl -H "Authorization: Bearer $JIRA_TOKEN" https://your-jira-instance/rest/api/2/issue/...'
}
```

!!! warning "常见坑：不小心把 credential 打印出来了"
    `withCredentials` 只会 mask *那个具体的 variable*——但如果你直接 `echo $ART_PASS`，
    或者这个 credential 被拼进一个更大的字符串，然后这个字符串又通过别的方式被 log 出来了，
    mask 机制可能就漏掉了。永远不要为了 "调试一下" 就手动 `echo` 一个 credential variable。

## Try it yourself

1. 看看 [2.4](2-4-declarative-pipeline-structure.md) 那份骨架里，哪些 stage 需要
   credential（提示：至少 Upload stage 需要）。
2. 把那个 stage 的 `steps` block 改写一遍，用 `withCredentials` 包住一个假想的
   `artifactory-upload-creds` credential。

!!! tip "How this applies to the capstone"
    [Module 5](../module-5-artifactory/5-2-1-jfrog-cli.md) 里每一条 upload command——
    `jf`、`docker push`、`helm push`——都需要跟 registry 做 authenticate。真实的 pipeline
    里，这些 command 每一条都应该包在一个 `withCredentials` block 里，而不是写死一个 token。
