# 6.3 · REST API 概念

Jira（[4.2](../module-4-jira/4-2-jira-rest-api.md)）和 Artifactory 都对外暴露 REST API。
这一页讲的是这套 stack 里任何 REST API 都通用的一些概念。

## HTTP 方法，翻译成这个项目里的说法

| 方法 | 意思 | 举例 |
|---|---|---|
| `GET` | 读数据，没有副作用 | 拿一张 Jira ticket 的字段 |
| `POST` | 创建东西，或者触发一个动作 | 给 ticket 加评论；trigger 一次 Jenkins build |
| `PUT` | 替换/更新，upload 场景经常用 | 用原始 PUT upload 一个 Helm chart（[5.2.3](../module-5-artifactory/5-2-3-helm-push.md)） |
| `DELETE` | 删掉某个东西 | 这个项目的流程里很少用到 |

## 值得认识的 status code

| Code 范围 | 意思 | 该怎么办 |
|---|---|---|
| `2xx` | 成功 | 继续往下走 |
| `401` | 没 authenticate | 检查 credential/token 在不在、有没有过期 |
| `403` | Authenticate 了，但没权限 | 检查权限，不是 token 有没有效 |
| `404` | 这个资源不存在 | 检查 URL/path/key 对不对 |
| `429` | 被 rate limit 了 | 退避一下——见 [4.2](../module-4-jira/4-2-jira-rest-api.md) 里那个 rate-limit 的 warning |
| `5xx` | Server 那边出错了 | 一般是暂时的，加个 backoff 重试通常没问题 |

## 会遇到的几种 authentication 方式

```bash
# Bearer token（Jira Cloud 常见，很多现代 API 都这样）
curl -H "Authorization: Bearer $TOKEN" https://api.example.com/resource

# Basic auth（Artifactory 常见）
curl -u "$USER:$PASS" https://api.example.com/resource
```

这两种方式的 credential，都得来自 Jenkins 的 credential store
（[2.8](../module-2-cicd-jenkins/2-8-credentials.md)），不能写死在脚本里。

## 在 pipeline 里读一个 JSON response

```groovy
script {
    def response = sh(script: 'curl -s -H "Authorization: Bearer $JIRA_TOKEN" https://.../issue/PROJ-1234',
                       returnStdout: true).trim()
    def json = readJSON text: response
    echo "Status: ${json.fields.status.name}"
}
```

`readJSON`（来自 Pipeline Utility Steps 这个 plugin）把一个 JSON 字符串解析成 Groovy 的
map/list，之后读字段的方式，跟在 Python 里操作嵌套 dictionary 差不多。

## Idempotency —— 为什么这对重试很重要

一个操作是 **idempotent** 的，意味着跑两遍跟跑一遍的效果是一样的。`GET` 永远是
idempotent 的。`PUT`（替换）一般也是。`POST`（创建/触发）一般**不是**——盲目重试一个
失败的 `POST`，可能会造成重复的动作（比如两条评论，或者两次 trigger 的 build）。
只要给某个 API 调用加重试逻辑，这一点就得考虑进去。

!!! warning "常见坑：重试一个非 idempotent 的 POST"
    如果一个 build-trigger 的 `POST` 超时了，但其实 server 那边已经成功执行了，简单粗暴地
    重试可能会导致重复 trigger。尽量用 idempotent 的操作，或者在确认第一次真的失败了之后，
    才有条件地重试。

## Try it yourself

1. 假设一个 `curl` 调用返回 `403`，列出两个可能的根因，以及怎么区分它们。
2. 写一段用 `readJSON` 从 Jira API response 里取出某个 custom field 值的代码片段。

!!! tip "How this applies to the capstone"
    这个项目里每一个集成点——读一张 Jira ticket、upload 到 Artifactory、trigger 一个
    downstream 的 Jenkins job——底层都是一次 REST 调用。这一页的这套 vocabulary
    （status code、authentication 方式、idempotency）在所有这些场景里都是通用的。
