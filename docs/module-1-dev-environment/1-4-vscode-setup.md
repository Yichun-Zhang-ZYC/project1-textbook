# 1.4 · VS Code 配置

针对这套 stack 里会用到的文件类型（Groovy 写的 Jenkinsfile、YAML、Markdown），给一套
够用、不折腾的配置。

## 值得装的 Extension

| Extension | 为什么要装 |
|---|---|
| **GitLens** | 直接在 editor 里看 inline blame、历史、diff。 |
| **YAML**（Red Hat） | 给 YAML 做 schema validation 和自动补全——commit 之前就能抓到缩进错误。 |
| **Groovy** language support | 给 Jenkinsfile（`.groovy` / `Jenkinsfile`）做语法高亮。 |
| **Jenkins Pipeline Linter Connector**（可选） | 能对着真实的 Jenkins instance 的 linter 校验 Jenkinsfile，不用先 commit 才能知道对不对。 |

## 推荐设置

- **Render whitespace**（`editor.renderWhitespace: "boundary"`）——YAML 和 Groovy 都对
  缩进敏感，能看见多余/混用的空白字符能省不少 debug 时间。
- Markdown/YAML 开 **format on save**——保持 diff 干净。
- Integrated terminal 的默认 shell 手动设一下——这样随时都清楚自己在哪个 shell 的上下文里，
  见 [1.1 · Terminal 基础](1-1-terminal-basics.md)。

## 打开一个 repo

```bash
cd path/to/repo
code .
```

`code .` 会在当前目录打开 VS Code——如果 `code` 这个命令找不到，那是 PATH 的问题（见
[1.1 · Terminal 基础](1-1-terminal-basics.md)）：重装的时候勾上 "Add to PATH"，或者在
VS Code 的 command palette（`Ctrl+Shift+P`）里跑 **Shell Command: Install 'code' command
in PATH**。

## Try it yourself

1. 装上 YAML 和 Groovy 这两个 extension。
2. 打开（或者新建）一个小的 `.yml` 文件，故意把某一行的缩进写错——确认 extension 会标出来。
3. `Ctrl+Shift+P` → **Terminal: Select Default Profile**，把默认的 integrated terminal shell
   设好。

!!! tip "How this applies to the capstone"
    这个项目里所有的 Jenkinsfile、YAML config、Markdown 页面都是在 VS Code 里改的——尤其是
    YAML extension，能帮你避开整个这套 stack 里最常见的一个 bug：缩进写错（见
    [6.1 · YAML 基础](../module-6-supporting-skills/6-1-yaml-basics.md) 里那个 warning）。
