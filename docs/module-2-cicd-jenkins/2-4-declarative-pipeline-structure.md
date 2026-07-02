# 2.4 · Declarative Pipeline 结构

这是这个项目里每个 Jenkinsfile 的骨架。从头到尾读一遍——下面会逐个解释每个关键字。

```groovy
pipeline {
    agent any

    parameters {
        booleanParam(name: 'PUSH_TO_ARTIFACTORY', defaultValue: false,
                      description: 'Trigger the downstream upload job after a successful scan')
        string(name: 'ARTIFACT_TYPE', defaultValue: '', description: 'binary | docker | helm | terraform')
    }

    environment {
        REGISTRY_URL = 'https://myregistry.jfrog.io'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Security Scan') {
            steps {
                echo "Running security scan..."
            }
        }
        stage('Upload') {
            when {
                expression { params.PUSH_TO_ARTIFACTORY == true }
            }
            steps {
                echo "Uploading artifact of type ${params.ARTIFACT_TYPE}..."
            }
        }
    }

    post {
        success {
            echo "Pipeline succeeded"
        }
        failure {
            echo "Pipeline failed — check logs above"
        }
    }
}
```

## 一块一块拆开看

| Block | 作用 |
|---|---|
| `pipeline { }` | 最外层的 wrapper——所有 declarative Jenkinsfile 都从这里开始。 |
| `agent` | 这条 pipeline 在哪跑（见 [2.2 · Jenkins 架构](2-2-jenkins-architecture.md)）。`any` = 随便哪个空闲 agent；`{ label '...' }` = 指定某种 agent。 |
| `parameters { }` | 声明这个 job 接受哪些 build parameter——见 [2.5](2-5-build-parameters.md)。 |
| `environment { }` | 每个 stage 都能用的 environment variable。 |
| `stages { }` | 组成这条 pipeline 的一串有顺序的 stage。 |
| `stage('name') { }` | 一个有名字的阶段。在 Jenkins UI 里会单独显示成一段。 |
| `steps { }` | Stage 里真正执行的 command/动作。 |
| `when { }` | 一个条件——只有满足这个条件，这个 stage 才会跑。见 [3.3 · Conditional Stages](../module-3-groovy-jenkinsfile/3-3-conditional-stages.md)。 |
| `post { }` | 所有 stage 跑完之后执行，不管结果如何——常用来做 `success`/`failure`/`always` 的通知或者清理工作。 |

## Try it yourself

1. 把上面这段骨架 copy 到一个叫 `Jenkinsfile` 的文件里。
2. 加第三个 parameter，`string(name: 'DESTINATION_PATH', ...)`，然后在 Upload stage 的
   `echo` 里用 `${params.DESTINATION_PATH}` 引用它。
3. 顺一遍逻辑：如果 `PUSH_TO_ARTIFACTORY` 是 `false`，到底哪些 stage 会真正跑？

!!! tip "How this applies to the capstone"
    这套骨架——parameters block 加一个有条件的 Upload stage——正是 capstone 场景里 Option C
    的骨干。Module 7 会把这个 `when` 条件和这个 stub `echo` 背后真正要调的 downstream job
    补全。
