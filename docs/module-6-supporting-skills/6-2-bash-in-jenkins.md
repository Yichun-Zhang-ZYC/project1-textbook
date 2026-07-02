# 6.2 · Bash in Jenkins

`sh` step 会在 pipeline 当前跑的那个 agent 上（见
[2.2 · Jenkins 架构](../module-2-cicd-jenkins/2-2-jenkins-architecture.md)）跑一条
shell command——Linux agent 上就是 Bash。一条 pipeline 里大部分 "真正干活" 的逻辑
（跑 `jf`、`docker`、`helm`、`terraform` 这些 command）都是在 `sh` step 里发生的，
不是在 Groovy 本身。

## 基本用法

```groovy
steps {
    sh 'echo hello'
    sh '''
        echo "multi-line block"
        echo "second line"
    '''
}
```

多行 block 用单引号（`'''...'''`），能避开 Groovy 的字符串插值——如果这段 shell script
自己也用 `$` 表示自己的变量，又不想让 Groovy 先把它们插值掉，这个写法很有用。

## 把 Groovy 变量传进 shell command

```groovy
steps {
    sh "jf rt upload ${params.ARTIFACT_PATH} ${params.DESTINATION_PATH}"
}
```

双引号字符串（`"..."`）*会*在这段字符串交给 shell 之前，先插值 Groovy 变量
（`${...}`）——pipeline 的 parameter 就是这么被塞进真实 shell command 里的。

## 把 shell 的输出捕获回 Groovy

```groovy
script {
    def version = sh(script: 'cat VERSION', returnStdout: true).trim()
    echo "Detected version: ${version}"
}
```

`returnStdout: true` 把 `sh` 从 "跑一下就完了" 变成 "跑一下，把输出当字符串给我"——
只要后面某个 stage 需要一个只有 shell 才能算出来的值，就得这么用。

## Exit code 和失败

默认情况下，`sh` step 里的 shell command 只要 exit non-zero，整个 stage 会立刻失败。
想在不让 build 失败的前提下检查一个 command 成功没有：

```groovy
def exitCode = sh(script: 'jf rt search "generic-local/tools/x/*"', returnStatus: true)
if (exitCode != 0) {
    echo "Upload verification failed"
}
```

`returnStatus: true` 会把 exit code 返回给你，而不是自动让 build 失败。

!!! warning "常见坑：单引号 vs. 双引号，插值的时机不一样"
    `sh "echo ${env.PATH}"` 会在 shell *还没看到*这条 command 之前，先把 Groovy 变量
    `env.PATH` 插值进去。`sh 'echo $PATH'`（单引号）会把 `$PATH` 原样传给 shell，
    是*shell 自己*在 runtime 把它展开的。如果某个变量在 Groovy 和 shell 自己的
    environment 里都存在，这两种写法很可能给出完全不一样的结果。

## Try it yourself

1. 写一个 `sh` step，用两个 Groovy parameter 跑 `jf rt upload`，command 分多行写。
2. 写一个 `script { }` block，捕获一次 `jf rt search` 的 exit code，根据结果
   `echo` 一句 pass/fail 的消息。

!!! tip "How this applies to the capstone"
    [Module 5](../module-5-artifactory/5-1-repository-types.md) 里每一条 upload
    command，跑起来都是在一个 `sh` step 里，跟上面这些例子一模一样——这一页就是
    Groovy 层的分支逻辑（[3.2](../module-3-groovy-jenkinsfile/3-2-script-blocks.md)）
    和真正的 upload command 之间的那层胶水。
