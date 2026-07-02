# 7.3 · Debug Scenarios

用 [3.4 · Debugging](../module-3-groovy-jenkinsfile/3-4-debugging.md) 里那套三层排查方法，
套到 [7.1](7-1-option-c-mock-implementation.md) 和 [7.2](7-2-downstream-job-branching.md)
搭出来的完整 pipeline 上，走几个实战案例。

## 场景 1："我把 `PUSH_TO_ARTIFACTORY` 设成 true 了，但 upload 就是没 trigger"

**先查 Layer 2（declarative 结构）/ 那个 `when` guard。**

回想一下 7.1 里那个 guard：

```groovy
when {
    allOf {
        expression { params.PUSH_TO_ARTIFACTORY == true }
        expression { params.ARTIFACT_TYPE != '' }
    }
}
```

最可能的原因：`ARTIFACT_TYPE` 是空的。`allOf` 里两个条件都得成立。可以在这个 stage 之前
加一个 debug stage，把每个 parameter 的值都 echo 出来，确认这次 build 到底收到了什么——
注意 `when` 条件是在任何 `steps` 之前跑的，所以不能直接在这个 stage 里面加 echo，得放
更早的地方。

第二可能的原因：`PUSH_TO_ARTIFACTORY` 是从某个外部 trigger（比如一次 REST API 调用）传过来
的，到手的其实是字符串 `"false"`，不是真的 boolean `false`——见
[2.5](../module-2-cicd-jenkins/2-5-build-parameters.md) 里那个类型不匹配的 warning。
打印 `params.PUSH_TO_ARTIFACTORY.getClass()` 确认一下。

## 场景 2："Downstream job 跑起来了，但 upload command 失败了"

**先查 Layer 3（runtime/shell）。**

去看 downstream job build log 里 `sh` step 真实的输出——会显示出参数已经插值完的完整
command，以及 shell 的报错内容。这一层最常见的根因，按可能性从高到低排：

1. Credential 问题——`withCredentials` block 没写，或者 credential ID 打错了。看看是不是
   `401`/`403` 那种形状的错误（见 [6.3](../module-6-supporting-skills/6-3-rest-api-concepts.md)）。
2. Destination path 的形状跟 artifact 类型对不上——比如 Docker 的 `ARTIFACT_PATH` 不是一个
   完整合法的 registry tag（见 [5.2.2](../module-5-artifactory/5-2-2-docker-push.md) 里
   那个坑）。
3. `switch` 语句走到了 `default` 分支——意味着 `ARTIFACT_TYPE` 跟四个预期值都对不上
   （大小写、多余的空格）。

## 场景 3："Pipeline 报成功了，但 Artifactory 里啥都没看到"

**先查 Layer 3，但要看*验证*那一步，不是 upload 那一步。**

如果 `Verify Upload`（见 [7.2](7-2-downstream-job-branching.md)）根本没真的 assert 什么——
比如 `jf rt search` 跑了，但结果是不是空的从没被检查过——pipeline 就可能报成功，尽管
upload 在更前面的地方已经悄悄失败了（`docker`/`helm` 这两条分支尤其容易中招，因为它们没有
自动的、基于 search 的验证）。这跟 [7.2](7-2-downstream-job-branching.md#the-default-branch-is-not-optional)
里提到的那种 "成功掩盖失败" 的失败模式是一回事。

解法：让验证步骤在结果为空/不符合预期的时候真的让 build 失败，而不只是 log 一下了事。

## 场景 4："昨天还好好的，今天就挂了，Jenkinsfile 一行都没改"

**先别查上面那三层——查外部状态。**

- Jira custom field 的 ID 是不是变了（见 [4.2](../module-4-jira/4-2-jira-rest-api.md) 里
  那种 `customfield_10050` 式的不透明 ID）？Field ID 在 Jira project 重新配置之后，
  不保证一定还是原来那个。
- Artifactory 的 token 是不是过期了？Credential 会 rotate；先直接确认这个 credential
  本身还有没有效，不要一上来就怀疑是 pipeline 的问题。
- Destination path 的 convention 是不是变了，但没通知到所有发布方（见
  [5.3](../module-5-artifactory/5-3-destination-path-design.md)）？

## Try it yourself

挑上面任意一个场景，写出你会加的具体 `echo`/debug 语句，在改任何实际逻辑*之前*先确认根因——
这正是 [3.4](../module-3-groovy-jenkinsfile/3-4-debugging.md) 里 "先 debug，后修" 的那套
纪律。

!!! tip "How this applies to the capstone"
    这四个场景覆盖的是 Option C 这套设计里最容易出问题的地方——parameter 传递、
    按类型分支、以及验证环节里那种悄悄成功的漏洞。这套 stack 里真实的 debug 过程，
    几乎都是在判断 "这几层里到底是哪一层出了问题"，而不是什么玄学。
