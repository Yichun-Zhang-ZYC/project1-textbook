# CI/CD Artifact 上传自动化学习笔记

## 这是个啥

这是一份纯个人的 study reference，记录怎么搭一条 pipeline，让 artifact 先过 **security scan**，
scan 过了之后自动 **upload 到 artifact registry**——用来取代那种全靠人工开 ticket 走流程的老办法。

写这份笔记的时候假设读者（也就是我自己）有还不错的通用 coding 背景，但对 Jenkins、CI/CD
pipeline、Groovy、Jira REST API、Artifactory/JFrog 这些具体的 packaging convention **完全没碰过**。
所以像 variable、function、loop 这种通用编程概念不会展开讲，但凡是这个 CI/CD/DevOps stack
特有的东西，都会从零开始解释。

## 怎么用这个网站

按顺序读，因为后面的 module 都会默认你已经知道 [Module 0](module-0-orientation/0-1-glossary.md)
里引入的 vocabulary 和贯穿全站的 running example。每个 module 的结构都差不多：

- 短段落 + 表格 + code block，尽量不写没用的长篇大论。
- 每个 hands-on 的 chapter 结尾都有一个 `Try it yourself` 小练习。
- 每个技术性 chapter 结尾都有一个 `How this applies to the capstone` 提示框，把这个 concept 和
  贯穿全站的 running example 对应起来。

## 贯穿全站的 running example

整个网站用同一个场景当 running example：有个内部工具需要先做 security scan 排查安全问题，
scan 过了之后 approve，然后自动 upload 到 artifact registry——全程自动化，不需要人工在中间开
ticket 交接。这个场景（已经完全 anonymize 过——不涉及任何真实公司、team 或者人名）在
[0.4 · Capstone 场景](module-0-orientation/0-4-project-requirements.md) 里完整展开，
[Module 7](module-7-capstone/7-1-option-c-mock-implementation.md) 会把它当成一个完整的
mock implementation 走一遍。

## Module 一览

| # | Module | 讲什么 |
|---|--------|-------|
| 0 | 入门须知 | Vocabulary、架构图、artifact 类型、running example |
| 1 | 开发环境 | Terminal、Git、GitHub、VS Code |
| 2 | CI/CD 与 Jenkins | Jenkins 架构、job、pipeline、parameter、trigger |
| 3 | Groovy 与 Jenkinsfile | 怎么读/写 pipeline code |
| 4 | Jira | Ticket、REST API、webhook，以及这些东西的局限 |
| 5 | Artifactory | 怎么把每种 artifact 类型 upload 到 registry |
| 6 | 辅助技能 | YAML、Bash in Jenkins、REST API 概念 |
| 7 | Capstone | 完整地把一个 mock implementation 从头走到尾 |
