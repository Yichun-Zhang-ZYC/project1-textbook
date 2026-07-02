# 1.1 · Terminal 基础

Windows 上会遇到三种不同的 shell。搞清楚哪个是哪个很重要，因为 command、path、environment
variable 的写法在这三者之间**不能互换**。

| Shell | 是啥 | Path 写法 | 什么时候会用到 |
|---|---|---|---|
| **PowerShell** | Windows 自带的 shell | `C:\Users\you\project` | 默认的 Windows terminal，`winget`，大部分 Windows 专属的工具 |
| **Git Bash** | 跟着 Git for Windows 一起装的类 Unix shell | `/c/Users/you/project` | 跑 Unix 风格的 script、`curl`，或者从 Linux/macOS 文档里 copy 过来的 command |
| **WSL（Windows Subsystem for Linux）** | 一个真正跑在 Windows 里的 Linux 环境 | `/home/you/project` 或 `/mnt/c/Users/you/project` | 需要完整 Linux 兼容性的时候（Docker、某些 CLI 在这里表现更正常） |

这个项目其实不需要三个都装，但网上抄命令的时候经常会遇到某个是给别的 shell 写的，得自己
翻译一下。举几个例子：

| 要做的事 | PowerShell | Git Bash / WSL |
|---|---|---|
| List 文件 | `Get-ChildItem`（alias `ls`） | `ls` |
| 打印当前目录 | `Get-Location`（alias `pwd`） | `pwd` |
| 给一条 command 临时设个 env var | `$env:FOO="bar"` | `FOO=bar command` |
| 看一个文件的内容 | `Get-Content file`（alias `cat`） | `cat file` |
| "成功了才继续" | `A; if ($?) { B }` | `A && B` |

## PATH——最容易踩坑的地方

`PATH` 是一个 environment variable，列了一堆文件夹，shell 输命令的时候会去这些文件夹里找。
装一个新的 CLI 工具（比如 `git`、`jf`、`gh`）的时候，installer 通常会自动把它的文件夹加到
`PATH` 里——但**已经开着的 terminal 窗口看不到这个更新**。这是最常见的 "我明明刚装完啊！"
类型的 bug。

!!! warning "PATH 在已经打开的 terminal 里不会自动更新"
    装完东西之后如果 command 还是提示找不到，先别急着以为装失败了——关掉重开一个 terminal
    （用的是 editor 自带的 integrated terminal 的话，可能得把整个 editor 都重开）再试。

## 检查一个工具装没装

```powershell
# PowerShell
git --version
where.exe git      # 显示这个 executable 的完整路径
```

```bash
# Git Bash / WSL
git --version
which git
```

## Try it yourself

1. 打开一个 terminal（三种任选）。
2. 跑 `git --version`。如果报错了，说明是 PATH 的问题——先记下来，
   [1.2 · Git 基础](1-2-git-basics.md) 里会解决。
3. 打印当前目录，然后 list 出里面的内容，用你打开的那个 shell 对应的写法。

!!! tip "How this applies to the capstone"
    后面的 module 会给一堆能直接 copy-paste 的 command——有些是 `bash` block（这其实就是
    Jenkins pipeline 在自己的 build agent 上真正跑的东西，见
    [6.2 · Bash in Jenkins](../module-6-supporting-skills/6-2-bash-in-jenkins.md)），有些是
    通用的 CLI 示例。搞清楚自己在哪个 shell 里，能省掉不少 "command not found" 的困惑。
