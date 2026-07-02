# 7.2 · Downstream Job Branching

[7.1](7-1-option-c-mock-implementation.md) 里那个 downstream 的 `artifact-upload-job`，
需要根据 `ARTIFACT_TYPE` 跑不同的 upload command——这一页把这个分支逻辑完整补上，
用到的每条 command 都来自 [Module 5](../module-5-artifactory/5-1-repository-types.md)。

## 完整的分支 stage

```groovy
pipeline {
    agent any

    parameters {
        string(name: 'ARTIFACT_TYPE', defaultValue: '')
        string(name: 'ARTIFACT_PATH', defaultValue: '')
        string(name: 'DESTINATION_PATH', defaultValue: '')
    }

    stages {
        stage('Upload') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'artifactory-upload-creds',
                    usernameVariable: 'ART_USER',
                    passwordVariable: 'ART_PASS'
                )]) {
                    script {
                        switch (params.ARTIFACT_TYPE) {
                            case 'binary':
                                sh "jf rt upload ${params.ARTIFACT_PATH} ${params.DESTINATION_PATH}"
                                break
                            case 'terraform':
                                sh "jf rt upload ${params.ARTIFACT_PATH} ${params.DESTINATION_PATH}"
                                break
                            case 'docker':
                                sh """
                                    echo "\$ART_PASS" | docker login myregistry.jfrog.io --username "\$ART_USER" --password-stdin
                                    docker push ${params.ARTIFACT_PATH}
                                """
                                break
                            case 'helm':
                                sh """
                                    helm registry login myregistry.jfrog.io --username "\$ART_USER" --password "\$ART_PASS"
                                    helm push ${params.ARTIFACT_PATH} oci://myregistry.jfrog.io/helm-local/
                                """
                                break
                            default:
                                error "Unknown ARTIFACT_TYPE: ${params.ARTIFACT_TYPE}"
                        }
                    }
                }
            }
        }

        stage('Verify Upload') {
            steps {
                script {
                    if (params.ARTIFACT_TYPE in ['binary', 'terraform']) {
                        sh "jf rt search '${params.DESTINATION_PATH}*'"
                    }
                    // docker/helm 的验证：见 5.2.2 / 5.2.3
                }
            }
        }
    }
}
```

## 为什么这里用 `switch`，不是继续用 `if`/`else if`

Groovy 对字符串用 `switch`，跟 [3.1 练习](../module-3-groovy-jenkinsfile/3-1-groovy-basics.md#try-it-yourself)
里那条 `if`/`else if` 链在功能上是等价的——这里用 `switch` 是因为按一组已知、固定的
artifact 类型分支（还带一个明确的 `default` 兜底任何意外情况），读起来比一长串
`if`/`else if` 更清楚。两种写法在 Groovy 里都是对的；这只是个可读性上的选择，不是功能上
必须的。

## The `default` branch is not optional

`error "Unknown ARTIFACT_TYPE: ..."` 是故意设计成，一旦 `ARTIFACT_TYPE` 是四个已知值之外
的任何东西，就**大张旗鼓地让 build 失败**。没有这一句，一个没被识别的值会悄悄从这里穿过去，
什么都不做——一个看起来*像*成功、实际上却是 upload 失败的 build。这正是
[3.4 · Debugging](../module-3-groovy-jenkinsfile/3-4-debugging.md) 里提到的那种失败模式：
一个 "成功"，其实藏着上一层的问题。

## Try it yourself

1. 给 `switch` 语句加一个第五种 artifact 类型（自己编一个），包括它自己的、用
   `withCredentials` 包好的 upload command。
2. 解释一下：为什么 `docker` 和 `helm` 没有像 `binary`/`terraform` 那样自动带一个
   `jf rt search` 验证步骤（提示：回头看看
   [5.2.2](../module-5-artifactory/5-2-2-docker-push.md) 和
   [5.2.3](../module-5-artifactory/5-2-3-helm-push.md) 里各自的验证部分）。

!!! tip "How this applies to the capstone"
    这一页就是 [0.2](../module-0-orientation/0-2-architecture-diagram.md) 架构图里
    "按 artifact 类型分支" 那个 box 的真实实现——目标状态那条 pipeline 里最后一块
    没实现的部分。
