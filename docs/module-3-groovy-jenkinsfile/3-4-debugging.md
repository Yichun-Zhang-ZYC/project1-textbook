# 3.4 · Debugging

Jenkinsfile 出错的方式跟普通脚本不太一样——默认没有本地 REPL 可以用，而且错误可能来自三个
不同的层：Groovy 语法本身、declarative pipeline 的结构、或者某个 stage 跑的 shell command。
搞清楚自己在哪一层，能大大缩小排查范围。

## Layer 1：语法错误（根本跑不起来）

症状：build 一开始就直接失败，任何 stage 都还没跑，报的是 Groovy 的 parse error。

- 先检查花括号（`{ }`）有没有配对好——这是最常见的原因。
- 检查是不是在没套 `script { }` wrapper 的情况下，直接在 `steps { }` 里写了裸 Groovy 逻辑
  （见 [3.2](3-2-script-blocks.md)）。

## Layer 2：Declarative 结构错误

症状：在你自己的逻辑真正跑起来之前就很快失败，报的是类似 `Invalid parameter type` 或者
`Nothing to execute` 这样的消息。

- 确认 `parameters { }` 里声明的名字，跟每一处 `params.NAME` 引用完全一致（大小写敏感）。
- 确认 `when { }` 是直接放在 `stage { }` 里面，没有嵌套在别的什么地方。

## Layer 3：Runtime / Shell 错误

症状：某个具体的 stage 失败了，带着一个 `sh` step 的输出。

- Shell step 真实的 stdout/stderr 都在 build log 里，跟看普通的 terminal 报错一样读就行。
- 在变量真正被用之前，临时加几句 `echo` 把它们的值打出来：

```groovy
script {
    echo "DEBUG: ARTIFACT_TYPE=${params.ARTIFACT_TYPE}, PUSH_TO_ARTIFACTORY=${params.PUSH_TO_ARTIFACTORY}"
}
```

## 几个好用的技巧

| 技巧 | 怎么做 |
|---|---|
| 打印一个变量的类型，不只是值 | `echo "type: ${params.PUSH_TO_ARTIFACTORY.getClass()}"`——能直接暴露出 [2.5](../module-2-cicd-jenkins/2-5-build-parameters.md) 里说的那个 string-vs-boolean 问题 |
| 改完 pipeline code 直接重跑，不用重新 commit | Jenkins 的 "Replay" 功能——可以直接改改 Jenkinsfile 再跑一次，不需要新开一次 commit，迭代的时候很好用 |
| Commit 之前先校验语法 | Jenkins Pipeline Linter（对 `/pipeline-model-converter/validate` 发 `curl`，或者用 [1.4](../module-1-dev-environment/1-4-vscode-setup.md) 里那个 VS Code extension） |
| 确认到底跑了哪个 stage | Jenkins UI 的 stage view 会按 stage 显示 skipped / 跑了 / 失败——能确认 `when` 条件到底是不是按你预期算的 |

!!! warning "常见坑：被跳过的 stage 看起来像什么都没发生"
    如果整个 stage 因为 `when` 条件被悄悄跳过了，很容易误以为是 "pipeline 坏了"，其实它
    是按配置正常工作的。先看 stage view 再下结论——见 [3.3](3-3-conditional-stages.md)。

## Try it yourself

1. 故意把某个 parameter 名字写错（声明的是 `ARTIFACT_TYPE`，引用的时候写成
   `params.ARTIFACTTYPE`），先自己预测一下会发生什么，再实际跑一遍看看对不对。
2. 在自己练习用的 Jenkinsfile 里，Upload stage 之前加一行 debug `echo`，把每个相关 parameter
   的值和类型都打出来。

!!! tip "How this applies to the capstone"
    [7.3 · Debug Scenarios](../module-7-capstone/7-3-debug-scenarios.md) 是一组用这套
    三层排查方法、针对完整 Option C pipeline 的实战案例。
