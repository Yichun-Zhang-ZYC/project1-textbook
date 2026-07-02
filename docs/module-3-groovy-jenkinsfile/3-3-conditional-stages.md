# 3.3 · Conditional Stages

`when { }` block 控制的是*整个 stage* 要不要跑——跟 `script { }` block 里的 `if` 逻辑是
两回事，后者控制的是一个已经在跑的 stage *内部*的行为。

## 基本语法

```groovy
stage('Upload') {
    when {
        expression { params.PUSH_TO_ARTIFACTORY == true }
    }
    steps {
        echo "Uploading..."
    }
}
```

如果这个 expression 算出来是 falsy，整个 stage 就会被跳过——在 Jenkins UI 里会显示成
skipped，不是 failed。

## 内置的 `when` 条件（不需要 `expression`）

| 条件 | 意思 |
|---|---|
| `branch 'main'` | 只在 `main` branch 上跑 |
| `environment name: 'DEPLOY_ENV', value: 'prod'` | 只在某个 env var 匹配的时候跑 |
| `expression { <任意 groovy boolean> }` | 万能的逃生舱，写任意逻辑 |
| `not { ... }` | 对另一个条件取反 |
| `allOf { ...; ... }` | 嵌套的条件必须全部为 true |
| `anyOf { ...; ... }` | 嵌套的条件里至少一个为 true |

## 组合条件

```groovy
when {
    allOf {
        expression { params.PUSH_TO_ARTIFACTORY == true }
        expression { params.ARTIFACT_TYPE != '' }
    }
}
```

这样只有 flag 是 true **而且**确实提供了 artifact 类型，Upload 才会跑——防住了 "flag 开了但
别的字段没填" 这种 case。

## `when` vs. `script` 里的 `if`

```groovy
stage('Upload') {
    when {
        expression { params.PUSH_TO_ARTIFACTORY == true }   // 控制：这个 stage 到底跑不跑？
    }
    steps {
        script {
            if (params.ARTIFACT_TYPE == 'docker') {          // 控制：既然这个 stage 要跑了，具体跑哪条 command？
                sh "docker push ..."
            }
        }
    }
}
```

用 `when` 来决定 "这次跑，这整个 stage 存不存在"——它会在 Jenkins UI 里显示成 skip，
对可读性/audit 很有价值。用 `script` 里的 `if` 来处理已经决定要跑的 stage *内部*更细的分支。

!!! warning "常见坑：`when` 只在 stage 进入的那一刻算一次"
    如果 `when { expression { ... } }` 里用的某个变量，在 pipeline 中途可能会变（比如在
    前面某个 stage 里被设置），一定要确保这个变量在*到达这个 stage 之前*已经设好了——`when`
    不会在 stage 跑到一半的时候重新算一遍。

## Try it yourself

1. 给 [2.4 骨架](../module-2-cicd-jenkins/2-4-declarative-pipeline-structure.md) 里的
   Upload stage 加一个 `anyOf` 条件，让它在 `params.ARTIFACT_TYPE == 'docker'` 的时候也跑，
   不管那个 boolean flag 是啥。
2. 用一句话解释一下，为什么这对这个具体的 capstone 场景来说可能是个*不太好*的主意。

!!! tip "How this applies to the capstone"
    这一行 `when { expression { params.PUSH_TO_ARTIFACTORY == true } } }`，几乎原封不动地
    就是 Option C "如果是 true → 跑 downstream job" 这条逻辑的完整实现。看它怎么接进完整
    pipeline 的，见 [Module 7](../module-7-capstone/7-1-option-c-mock-implementation.md)。
