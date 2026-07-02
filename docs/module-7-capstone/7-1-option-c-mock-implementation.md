# 7.1 · Mock Implementation

这一页把 capstone 场景（[0.4](../module-0-orientation/0-4-project-requirements.md)）里的
**Option C** 完整 mock 实现一遍：main build 上加一个 boolean parameter，一旦是 true，
就 trigger 一个还没实现的 downstream upload job。这里用到的每一块，Module 2-6 都单独讲过——
这一页把它们拼到一起。

## Step 1 —— 给 main build 加 parameter

```groovy
pipeline {
    agent any

    parameters {
        booleanParam(name: 'PUSH_TO_ARTIFACTORY', defaultValue: false,
                      description: 'If true, trigger the downstream upload job after a successful scan')
        string(name: 'ARTIFACT_TYPE', defaultValue: '', description: 'binary | docker | helm | terraform')
        string(name: 'ARTIFACT_PATH', defaultValue: '', description: 'Path to the built artifact on this agent')
        string(name: 'DESTINATION_PATH', defaultValue: '', description: 'Target path/repo in Artifactory')
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build') {
            steps { echo "Building artifact..." }
        }

        stage('Security Scan') {
            steps { echo "Running security scan..." }
        }

        stage('Trigger Upload') {
            when {
                allOf {
                    expression { params.PUSH_TO_ARTIFACTORY == true }
                    expression { params.ARTIFACT_TYPE != '' }
                }
            }
            steps {
                build job: 'artifact-upload-job',
                      parameters: [
                          string(name: 'ARTIFACT_TYPE', value: params.ARTIFACT_TYPE),
                          string(name: 'ARTIFACT_PATH', value: params.ARTIFACT_PATH),
                          string(name: 'DESTINATION_PATH', value: params.DESTINATION_PATH)
                      ],
                      wait: true
            }
        }
    }
}
```

这就是 [2.4 的骨架](../module-2-cicd-jenkins/2-4-declarative-pipeline-structure.md)，
加上 [3.3](../module-3-groovy-jenkinsfile/3-3-conditional-stages.md) 里那个
`when { allOf { ... } }` 的 guard，再加上
[2.7](../module-2-cicd-jenkins/2-7-parent-child-jobs.md) 里那个 `build job:` 调用。

## Step 2 —— Downstream job（`artifact-upload-job`）

这是**另外一个** Jenkins job/Jenkinsfile，也就是 capstone 场景里提到的那个 "还没实现" 的
部分。完整的分支逻辑在
[7.2 · Downstream Job Branching](7-2-downstream-job-branching.md) 里搭出来——下面这段 stub
先看个大概形状：

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
                script {
                    // 完整的按类型分支逻辑在 7.2
                    echo "Uploading ${params.ARTIFACT_PATH} (${params.ARTIFACT_TYPE}) to ${params.DESTINATION_PATH}"
                }
            }
        }
    }
}
```

## 为什么 `when { allOf { ... } } }` 这个 guard 很重要

回想一下 [0.4](../module-0-orientation/0-4-project-requirements.md) 里那个悬而未决的设计
问题：*应不应该支持 "只 scan 不 upload"？* `allOf` 这个 guard 就是给出的具体答案：只有
flag 是 true **而且**确实提供了 artifact 类型，upload 才会 trigger——这样一次纯 scan 的
run（flag 是 false，或者压根没填 artifact metadata），就永远不会不小心触发一次半吊子/
出错的 upload 调用。

## Try it yourself

1. 用 `PUSH_TO_ARTIFACTORY=false` 顺一遍这条 pipeline——确认哪些 stage 会跑。
2. 用 `PUSH_TO_ARTIFACTORY=true` 但 `ARTIFACT_TYPE=''` 顺一遍——确认这个 guard 依然能挡住
   Trigger Upload stage 不让它跑，并解释一下为什么这才是对的行为。

!!! tip "How this applies to the capstone"
    这一页*就是* capstone 核心交付物的 mock 版本。要把它变成真正的实现，缺的只是真实的
    `Build`/`Security Scan` stage 逻辑，加上一个真实的 downstream job——其他的部分已经是
    生产环境该有的样子了。
