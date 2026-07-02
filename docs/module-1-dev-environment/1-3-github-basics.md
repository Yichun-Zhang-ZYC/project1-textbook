# 1.3 · GitHub 基础

GitHub 在远程托管 Git repository 之上，又加了一层协作工具：pull request、code review、
issue tracking，还有——跟这里相关的——Actions（CI）和 Pages（静态网站托管）。这个网站本身
就是照下面讲的这套方式 build 和 deploy 出来的。

## 核心概念

| Term | 意思 |
|---|---|
| **Repository** | GitHub 上托管的一个项目，底层是 Git。 |
| **Pull Request（PR）** | 某个 branch 上一组提议中的改动，merge 到主 branch 之前先拿出来给人 review。 |
| **Review / approval** | 队友读一遍 diff，要么提改动意见，要么 approve。 |
| **Merge** | 把一个已经 approve 的 branch 的改动合并进主 branch。 |
| **Issue** | 一个被 track 的任务/bug，跟 PR 是两码事（不过一个 PR 可以 "close" 掉一个 issue）。 |
| **GitHub Actions** | GitHub 自带的 CI/CD 系统——碰到 "push to main" 这类事件的时候，跑一个 workflow（YAML 文件）。 |
| **GitHub Pages** | 免费的静态网站托管，从 repo 的某个 branch（常见的是 `gh-pages`）直接serve 文件。 |

## 为什么 Jenkinsfile 要放在 repo 里："pipeline as code"

以前 CI pipeline 的配置都是靠在网页 UI 上点来点去搞定的——很难 review，很难 track 谁改了什么，
不同环境之间也容易跑偏。**Pipeline as code** 的意思是：整条 pipeline 的定义（一个 Jenkinsfile）
就是个文本文件，跟它要 build 的东西存在同一个 repo 里：

- 改 pipeline 走的是跟改代码一样的 PR/review 流程。
- Pipeline 的历史就是 Git 历史——对着 Jenkinsfile 跑 `git log`，能看到每一次改动。
- 不同 branch 可以有不同的 pipeline 行为，就跟不同 branch 可以有不同的应用代码一样。

这也是为什么 Module 3（Groovy/Jenkinsfile）很重要，即便你从来不直接碰 Jenkins 的网页 UI：
pipeline *本身就是一个文件*，跟改别的文件没区别。

## PR 的流程

```bash
git checkout -b add-upload-stage
# ...改 Jenkinsfile...
git add Jenkinsfile
git commit -m "Add artifact upload stage"
git push -u origin add-upload-stage
```

然后在 GitHub 上开一个 PR（网页 UI，或者用 CLI 跑 `gh pr create`），拿去给人 review，
review 过了就 merge。Merge 到 `main` 这件事，很多时候本身就是一个 trigger——见
[2.6 · Triggers](../module-2-cicd-jenkins/2-6-triggers.md)。

## Try it yourself

1. 在 GitHub 上随便找一个你有权限的 repo，打开它的 **Actions** tab——看一个最近的 workflow
   run，看看它的 log。
2. 在这个 repo 里找一个 `.github/workflows/*.yml` 文件，读一遍——等你读完
   [6.1 · YAML 基础](../module-6-supporting-skills/6-1-yaml-basics.md) 之后，会发现这个结构
   很眼熟。

!!! tip "How this applies to the capstone"
    这个学习网站本身就是这么 deploy 的：push 到 `main` 会 trigger 一个 GitHub Actions
    workflow，负责 build 这个网站然后发布到 GitHub Pages。这是一个活生生、就在眼前的
    "pipeline as code" 例子。
