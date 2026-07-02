# 0.1 · 术语表 Glossary

后面所有 chapter 会用到的 term，先在这里过一遍。现在可以先扫一眼，之后哪个词看着眼熟又想不起来
具体意思，再回来查。

| Term | 解释 |
|---|---|
| **CI/CD** | Continuous Integration / Continuous Delivery——每次改动都自动 build、test、ship，而不是靠人工手动做。 |
| **Jenkins** | 这套 stack 里用的 CI/CD server/orchestrator，负责跑 "job"，job 执行的是 pipeline。 |
| **Job** | Jenkins 里一个配置好、有名字的工作单元（比如"build 这个 artifact"）。可以手动触发、定时触发，或者被别的 job 触发。 |
| **Build** | Job 的一次执行（一次 run）。Job 是定义，build 是具体跑的那一次。 |
| **Pipeline** | 一次 build 要经过的一系列 stage，写在 Jenkinsfile 这个 code 文件里定义。 |
| **Jenkinsfile** | 一个 Git repo 里存的文本文件（Groovy 语法），定义整条 pipeline——这就是所谓 "pipeline as code"。 |
| **Groovy** | Jenkinsfile 用的那个 JVM scripting language。 |
| **Declarative pipeline** | Jenkinsfile 里那种结构化、关键字驱动的写法（`pipeline { stages { ... } }`），跟自由格式的 "scripted" pipeline 相对。 |
| **Build parameter** | 传给某一次具体 build 的 input 值（比如一个 boolean flag、一个 string）。让同一个 job 每次跑的行为可以不一样。 |
| **Trigger** | 什么东西导致 build 开始跑：一个 webhook、一个 schedule（cron），或者被别的 job 调用。 |
| **Parent/child job** | 一个 job（parent）在自己的 pipeline 里调用另一个 job（child），通常还会传 parameter 过去。 |
| **Downstream job** | 跟 child job 是一个意思——被别的 job 触发、且是在它*之后*跑的 job。 |
| **Credential** | 存在 Jenkins 里的一个 secret（API token、密码、key），pipeline 用 ID 引用它，绝不会直接写死在代码里。 |
| **Artifact** | 真正要 build 出来、要 ship 的东西：一个 binary、一个 Docker image、一个 Helm chart、一个 Terraform module，等等。 |
| **Artifact registry** | Artifact 最终 upload 进去、存放的地方（这个网站里用 **Artifactory/JFrog** 举例）。 |
| **Repository（Artifactory 里的）** | Artifactory 里一个专门存某一类 artifact 的命名 bucket（比如一个 Docker repo、一个 generic repo、一个 Helm repo）。 |
| **`jf`（JFrog CLI）** | 用来跟 Artifactory 做 authenticate、然后 upload 东西的命令行工具。 |
| **Jira** | 用来提需求、track 工作进度的 ticketing/issue-tracking 系统（在这里主要用来 track security scan 的请求）。 |
| **Jira ticket / issue** | Jira 里一个具体被 track 的工作单元，用一个类似 `PROJ-1234` 这样的 key 标识。 |
| **Webhook** | 一种 HTTP callback——一个系统（比如 Jira）在某件事发生的时候，自动去调另一个系统（比如 Jenkins）的 URL。 |
| **REST API** | 通过标准 HTTP 方法（GET/POST/PUT）访问、返回结构化数据（通常是 JSON）的 web API。Jira 和 Artifactory 都是走这套。 |
| **YAML** | Jenkins job config、Helm chart、GitHub Actions 用的那种靠缩进表达结构的配置文件格式。 |
| **Security scan** | 一个自动化检查，在 artifact 被允许 ship 之前，先扫一遍有没有安全漏洞。 |

!!! tip "How this applies to the capstone"
    这些 term 会在整个网站的 running example 里反复出现：一个 artifact 先被 **scan**（security
    scan），结果被 track 成一个 **Jira ticket**，approve 之后，一条 **Jenkins pipeline**——写在
    **Jenkinsfile** 里——负责把它 upload 到 **artifact registry**。后面哪个 term 看着模糊，回来
    翻这张表就行。
