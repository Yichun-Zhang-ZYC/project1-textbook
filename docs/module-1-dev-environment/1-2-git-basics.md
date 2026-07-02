# 1.2 · Git 基础

Git 是一个 version control system：track 文件随时间的变化，每次变化存成一个 "commit"，
让多个人同时改同一份代码也不会互相覆盖。Jenkinsfile 是存在 Git repo 里的，所以在碰
Module 2、3 之前，得先把 Git 用顺手。

## 核心概念

| Term | 意思 |
|---|---|
| **Repository（repo）** | 一个被 Git track 的文件夹，保存完整的变更历史。 |
| **Commit** | 一次保存下来的 snapshot，附带一条说明这次改了什么的 message。 |
| **Branch** | 一条独立的开发线——可以在不影响主线代码的情况下先做点东西，等做好了再合回去。 |
| **Clone** | 把一个远程 repo 完整拷贝一份到自己电脑上。 |
| **Remote** | Repo 在别处（比如 GitHub 上）托管的那个版本，你 push 过去 / pull 下来都是跟它交互。 |
| **Staging area** | 你已经标记（`git add`）、准备进*下一个* commit 的那些改动。 |

## 日常用的核心流程

```bash
git clone <repo-url>          # 把一个已有的 repo 拷贝下来
cd <repo-folder>
git checkout -b my-branch     # 建一个新 branch 并切过去
# ...改文件...
git add path/to/file.md       # 把某个具体的改动加进 staging area
git status                    # 看看哪些 staged 了，哪些还没
git commit -m "清楚描述这次改了什么"
git push -u origin my-branch  # 把这个 branch 发布到 remote
```

## 检查自己的配置

```bash
git config --global user.name
git config --global user.email
```

如果哪个是空的，设一下：

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

!!! warning "常见坑：commit 之前忘了 git add"
    `git commit` 只会 commit **staged** 的东西。如果改了个文件但没先 `git add`，直接跑
    `git commit`，Git 会说没东西可 commit（或者 commit 了一个更早的旧 snapshot）。
    不确定的话，commit 之前先跑一下 `git status` 看看到底会包含什么。

## Try it yourself

1. 跑 `git config --global user.name` 和 `user.email`，确认两个都设好了。
2. 找个测试文件夹，跑 `git init`，建一个文件，`git add` 它，再 `git commit` 它。
3. 跑 `git log` 看看这条 commit 是不是进历史了。

!!! tip "How this applies to the capstone"
    Jenkinsfile 说白了就是 Git repo 里的一个文本文件——这就是所谓 "pipeline as code"（见
    [1.3 · GitHub 基础](1-3-github-basics.md)）。改 pipeline 就是改这个文件、commit、push，
    跟改别的代码没什么两样，不需要什么单独的 "部署 pipeline" 的步骤。
