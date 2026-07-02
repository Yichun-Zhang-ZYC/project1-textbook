# 6.1 · YAML 基础

YAML 在这套 stack 里到处都是：GitHub Actions workflow
（[1.3](../module-1-dev-environment/1-3-github-basics.md)）、Helm chart 的 metadata
（[5.2.3](../module-5-artifactory/5-2-3-helm-push.md)），还有各种 Jenkins job 的 config。
跟 Groovy 不一样，YAML 的**缩进是有语义的**——不是风格选择，它直接定义了结构。

## 基本语法

```yaml
name: example
enabled: true
count: 3
tags:
  - one
  - two
nested:
  key: value
  another_key: another_value
```

| YAML | 对应的 Python/JSON 结构 |
|---|---|
| `key: value` | `{"key": "value"}` |
| `key:` 后面跟缩进的 `- item` 行 | `{"key": ["item1", "item2"]}` |
| `key:` 后面跟缩进的 `subkey: value` | `{"key": {"subkey": "value"}}` |
| `true` / `false`（不加引号） | Boolean |
| 不加引号的数字 | Number |
| 其他的一切 | String（除非包含特殊字符，一般引号可加可不加） |

## 最容易出 bug 的那条缩进规则

**同级的 key 必须缩进在完全一样的层级。** 混用 tab 和 space，或者少一个空格多一个空格，
要么直接解析失败，要么——更糟——悄悄改变了谁嵌套在谁下面。

```yaml
# 错的 —— 'push' 嵌套的层级不对
on:
  push:
   branches:
      - main

# 对的 —— 全程保持一致的 2 格缩进
on:
  push:
    branches:
      - main
```

!!! warning "YAML 缩进错误是 '我明明写对了怎么不生效' 的头号原因"
    一个缩进不一致的 YAML 文件，很多时候还是能*解析成功*的——只是解析出来的结构跟你想的
    不一样（某个 key 嵌套深了一层或者浅了一层）。这种失败是悄悄的，不是报错的。用
    [1.4](../module-1-dev-environment/1-4-vscode-setup.md) 里那个 VS Code YAML
    extension——它会在你保存之前就把这种问题标出来。

## 什么时候要给字符串加引号

引号一般可加可不加，但有些情况不加就会被解析错：

```yaml
version: "1.0"     # 不加引号，有些 parser 会读成数字 1.0，不是字符串
status: "true"      # 不加引号，会读成 boolean true，不是字符串 "true"
```

## Try it yourself

1. 找一下这个网站自己用来 deploy 的 `.github/workflows/deploy.yml`，找出：trigger 是什么、
   job 叫什么名字、step 列表是什么。
2. 故意把某一行的缩进写错，先自己预测一下会解析出什么结构，再对照检查。

!!! tip "How this applies to the capstone"
    这个网站自己的 deploy workflow、以及任何你会碰到的 Jenkins job config，都是 YAML。
    这一页里价值最高的一个习惯：深层嵌套的东西，永远别在没有 linter 盯着的情况下手打——
    见上面那个 warning。
