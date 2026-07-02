# 2.3 · Job 类型

Jenkins 支持好几种 job "类型"——指的是这个 job 怎么被配置出来的，不是它做什么事。

| Job 类型 | 什么意思 | 什么时候用 |
|---|---|---|
| **Freestyle** | 完全靠网页 UI 配置——build 步骤、trigger，所有东西都是 UI 驱动的 config，没有 Jenkinsfile。 | 遗留下来的老 job / 非常简单的 job。不算 "pipeline as code"，一般能避免就避免，尤其是需要 review/历史记录的场景。 |
| **Pipeline** | Config 指向一个 Jenkinsfile（通常在被 build 的 repo 里）。所有 stage 逻辑都写在这个文件里。 | 只要不是特别简单的场景，默认都用这个——也是 Module 2-3 在教你怎么写的东西。 |
| **Multibranch Pipeline** | 对 repo 里的每个 branch 自动建一个子 job，每个子 job 跑自己 branch 上的 Jenkinsfile。 | 不同 branch 需要各自独立跑 pipeline 的场景（比如 PR 校验）。 |
| **Folder / Organization Folder** | 本身不是一个 job——是用来分组/namespace 其他 job 的方式。 | Job 数量很多的时候用来保持整洁。 |

## Parameterized vs. 非 Parameterized

上面任何一种类型都可以是 **parameterized** 的——意思是每次 build 都能接受不同的 input 值，
而不是每次都跑得一模一样。这跟 job 类型是两码事；一般是 config 里的一个勾选项，或者
declarative Jenkinsfile 里的一个 `parameters {}` block。见
[2.5 · Build Parameters](2-5-build-parameters.md)。

## 实际怎么选

这个项目里几乎任何场景，答案基本都是：一个 parameterized 的 **Pipeline** job，背后跟着一个
放在对应 repo 里的 Jenkinsfile。Freestyle job 没法给 pipeline 逻辑本身做 code review 和
留历史——跟 [1.3 · GitHub 基础](../module-1-dev-environment/1-3-github-basics.md) 里
"pipeline as code" 那套论证是一个道理。

!!! tip "How this applies to the capstone"
    Main build 和那个还没实现的 downstream upload job（Module 7）都是普通的、
    parameterized 的 Pipeline job——[2.7 · Parent/Child Jobs](2-7-parent-child-jobs.md) 里
    要实现这个 parent/child 关系，不需要什么特殊的 job 类型。
