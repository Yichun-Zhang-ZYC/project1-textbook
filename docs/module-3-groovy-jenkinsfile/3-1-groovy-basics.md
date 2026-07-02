# 3.1 · Groovy 基础

Groovy 是一个跑在 JVM 上的 scripting language——大概可以理解成 "语法长得有点像 Python，
底层语义是 Java 那一套"。不需要真的精通 Groovy，只要能读/写 Jenkinsfile 里常用的那一小部分
就够了。

## 跟 Python 很像的语法

| 概念 | Groovy | Python（对比一下） |
|---|---|---|
| Variable | `def name = "value"` | `name = "value"` |
| String interpolation | `"Hello, ${name}"` | `f"Hello, {name}"` |
| List | `def items = ['a', 'b', 'c']` | `items = ['a', 'b', 'c']` |
| Map | `def m = [key: 'value']` | `m = {'key': 'value'}` |
| If/else | `if (x == true) { ... } else { ... }` | `if x == True: ... else: ...` |
| For loop | `for (item in items) { ... }` | `for item in items:` |
| Function | `def greet(name) { return "hi ${name}" }` | `def greet(name): return f"hi {name}"` |

肉眼看最大的区别：Groovy 用 `{ }` 花括号，执行的时候不看缩进（不过为了可读性还是应该好好
缩进）——这点跟 [6.1](../module-6-supporting-skills/6-1-yaml-basics.md) 里那个缩进*确实*有
语义的 YAML 正好相反。

## Jenkinsfile 里实际会遇到的类型

- **String**——占了绝大多数（`params.ARTIFACT_TYPE`、credential ID、shell command）。
- **Boolean**——`true`/`false`，全小写，`when` 条件里经常用。
- **Map/List**——用来表达结构化的 config，比如
  [2.7](../module-2-cicd-jenkins/2-7-parent-child-jobs.md) 里那个 `parameters: [...]` list。

## Truthiness 的坑

```groovy
if (params.ARTIFACT_PATH) {
    // ARTIFACT_PATH 非 null 且非空字符串的时候，这里是 true
}
```

Groovy 把空字符串、`null`、`0` 都当成 falsy——跟 Python 挺像，比某些语言的规则宽松。这个特性
拿来判断 "这个 parameter 到底有没有被真正设置过" 挺方便。

!!! warning "常见坑：`==` vs. `.equals()` vs. 类型不匹配"
    `params.PUSH_TO_ARTIFACTORY == true` 的结果，取决于这个 parameter 到手的时候是真的
    boolean，还是字符串 `"true"`（见 [2.5 · Build Parameters](../module-2-cicd-jenkins/2-5-build-parameters.md)
    里那个 warning）。拿不准的时候，debug 阶段先把这个 parameter 的值和类型都打印出来看看
    （见 [3.4 · Debugging](3-4-debugging.md)）。

## Try it yourself

1. 写一个 Groovy function，`def uploadCommandFor(String artifactType)`，用 `if`/`else if`
   链，针对不同的 artifact 类型 return 不同的字符串。
2. 用 [0.3](../module-0-orientation/0-3-artifact-types.md) 里那四种 artifact 类型分别调用
   一遍，检查输出对不对。

!!! tip "How this applies to the capstone"
    给 Option C 写的每一个条件判断——"只有 flag 是 true 才 upload"、"根据 artifact 类型挑对
    的 command"——都是这种几行的 Groovy 代码，包在一个 `script { }` block 里（下一页），
    嵌在某个 pipeline stage 里面。
