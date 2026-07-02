# 3.2 · Script Blocks

Declarative pipeline（[2.4](../module-2-cicd-jenkins/2-4-declarative-pipeline-structure.md)
里那个 `pipeline { stages { ... } }` 的样子）故意限制了 `steps { }` block 里能直接写的东西——
这是为了保持结构化和简单。真的需要写点复杂逻辑（loop、复杂的条件判断、跨 step 的变量赋值）
的时候，就得用 `script { }` block，里面可以写任意的 Groovy。

## 语法

```groovy
stage('Upload') {
    steps {
        script {
            def uploadCmd = ''
            if (params.ARTIFACT_TYPE == 'docker') {
                uploadCmd = "docker push ${params.ARTIFACT_PATH}"
            } else if (params.ARTIFACT_TYPE == 'helm') {
                uploadCmd = "helm push ${params.ARTIFACT_PATH} oci://myregistry.jfrog.io/helm-local/"
            } else {
                uploadCmd = "jf rt upload ${params.ARTIFACT_PATH} ${params.DESTINATION_PATH}"
            }
            sh uploadCmd
        }
    }
}
```

在 `script { }` block 外面，declarative 语法只允许那一套固定的关键字（`when`、`steps`、
`environment` 等等）——不能在 stage 这一层直接写 `def uploadCmd = ...` 这种裸的 Groovy。
在 `script { }` block 里面，就是纯 Groovy，规则跟 [3.1](3-1-groovy-basics.md) 一样。

## 什么时候该用 `script { }`

| 场景 | 需要 `script { }` 吗？ |
|---|---|
| 跑一条 shell command | 不需要——`steps` 里直接写 `sh '...'` 就行 |
| 就一个简单的 true/false 判断某个 *stage* 要不要跑 | 不需要——用 `when { expression { ... } }`，见 [3.3](3-3-conditional-stages.md) |
| 根据好几个 parameter 拼一条 command 字符串 | 需要 |
| 循环一堆 artifact，一个个 upload | 需要 |
| 一个 step 里赋值一个变量，同一个 stage 后面的 step 要用 | 需要 |

## Try it yourself

1. 把 [3.1 练习](3-1-groovy-basics.md#try-it-yourself) 里写的那条 `if`/`else if` 链，
   包进一个 `steps { }` block 里的 `script { }` block。
2. 加一个 `for` 循环，遍历一个存着几个 artifact path 的 Groovy list，每个都调一次 `sh`。

!!! tip "How this applies to the capstone"
    [7.2 · Downstream Job Branching](../module-7-capstone/7-2-downstream-job-branching.md)
    里根据 artifact 类型分支的逻辑，整个都是写在一个 `script { }` block 里的——这正是那种
    "根据 parameter 挑一条 command" 的逻辑，没法干净地塞进纯 declarative 语法里。
