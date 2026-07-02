# 2.5 · Build Parameters

Build parameter 是在 trigger 一次 build 的那一刻传进去的 input 值，让同一个 job 每次跑的
行为可以不一样——不用为每种情况单独写一个 job。

## 常见的 parameter 类型

| 类型 | Groovy 声明 | 用来做什么 |
|---|---|---|
| Boolean | `booleanParam(name: 'PUSH_TO_ARTIFACTORY', defaultValue: false, description: '...')` | 一个能把某个 stage 开/关的 flag |
| String | `string(name: 'ARTIFACT_TYPE', defaultValue: '', description: '...')` | 自由文本 input，比如 artifact 类型或者 path |
| Choice | `choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: '...')` | 限定范围的下拉选项 |

## 每次 build 是怎么设的

- **手动**：人在 Jenkins UI 里点 "Build with Parameters"，填一个表单。
- **程序化**：另一个 job（parent job）trigger 这个 job，直接把 parameter 值传过来——不需要
  人参与。见 [2.7 · Parent/Child Jobs](2-7-parent-child-jobs.md)。
- **通过 REST API**：外部系统（原则上甚至可以是 Jira，通过 webhook——见
  [4.3](../module-4-jira/4-3-webhooks-and-triggers.md)）可以直接 POST 到 Jenkins 的
  build-trigger endpoint，在 request 里带上 parameter 值。

## 在 Jenkinsfile 里读 parameter

在 `steps { }` 或者 `when { }` block 里，parameter 可以用 `params.<NAME>` 拿到：

```groovy
stage('Upload') {
    when {
        expression { params.PUSH_TO_ARTIFACTORY == true }
    }
    steps {
        sh "jf rt upload ${params.ARTIFACT_PATH} ${params.DESTINATION_PATH}"
    }
}
```

## 为什么这对 capstone 的 Option C 很关键

一个 boolean parameter，`push_to_artifactory`，就是把
[0.4](../module-0-orientation/0-4-project-requirements.md) 里那个架构决策变成代码的全部
机制：main build 多一个 checkbox，后面所有东西都从这一个值往下分支——不需要为 "只 scan"
vs. "scan 完还要 upload" 单独开一个 job。

!!! warning "常见坑：parameter 传过来除非特别声明，否则默认都是 string"
    通过 REST API 或者某些 trigger 方式传的 parameter，即便这个 parameter 声明的是
    boolean/choice 类型，实际到手的可能还是个 string。`params.PUSH_TO_ARTIFACTORY == 'true'`
    和 `== true` 在不同触发方式下可能会悄悄给出不一样的结果。拿不准的时候，确认一下自己
    比较的到底是什么类型。

## Try it yourself

1. 加一个 `choice` 类型的 parameter，`ARTIFACT_TYPE`，可选值是 `binary`、`docker`、`helm`、
   `terraform`。
2. 写一个 stage，用 `if`/`else` 链根据 `params.ARTIFACT_TYPE` 的值 echo 不同的消息
   （完整语法见 [3.3 · Conditional Stages](../module-3-groovy-jenkinsfile/3-3-conditional-stages.md)）。

!!! tip "How this applies to the capstone"
    这一页是 Option C 的机械核心。Capstone 设计里剩下的所有东西——"如果是 true，就 trigger
    那个 downstream job，带上这些细节"——全都是在这个 parameter 模式的基础上搭出来的。
