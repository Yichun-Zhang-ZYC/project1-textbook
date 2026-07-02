# 4.2 · Jira REST API

Jira 几乎所有东西都能通过 REST API 操作——读 ticket、搜索、改字段、转 status。任何想跟 Jira
程序化交互的东西（包括 Jenkins pipeline），走的都是这条路，不是靠人点网页 UI。

## Authentication

API request 需要一个 token，通过 Bearer token 或者 basic auth header 传——存成 Jenkins
credential（见 [2.8 · Credentials](../module-2-cicd-jenkins/2-8-credentials.md)），
绝不能写死在代码里。

```bash
curl -H "Authorization: Bearer $JIRA_TOKEN" \
     -H "Content-Type: application/json" \
     "https://your-jira-instance/rest/api/2/issue/PROJ-1234"
```

## 读一张具体的 ticket

```json
GET /rest/api/2/issue/PROJ-1234

{
  "key": "PROJ-1234",
  "fields": {
    "summary": "Scan and upload artifact X",
    "status": { "name": "Approved" },
    "customfield_10050": "artifactory/generic-local/tools/artifact-x/"
  }
}
```

Custom field（见 [4.1](4-1-jira-concepts.md)）显示出来的名字是 `customfield_10050` 这种
不透明的 ID——一般得去查一下这个 project 的 field 配置，才能知道哪个 ID 对应哪个人能看懂的
field 名字，因为这个 ID 本身完全看不出是什么意思。

## 搜索多张 ticket（JQL）

Jira Query Language（JQL）用来一次搜一批 ticket：

```bash
curl -H "Authorization: Bearer $JIRA_TOKEN" \
     -G "https://your-jira-instance/rest/api/2/search" \
     --data-urlencode 'jql=project = PROJ AND status = "Approved" AND updated >= -1h'
```

这种调用，ticket 量不大的时候很便宜，但**规模一大就会变贵**——具体为什么这对 capstone 的
架构决策很重要，见 [4.4](4-4-why-querying-is-expensive.md)。

## 更新一张 ticket / 加评论

```bash
curl -X POST -H "Authorization: Bearer $JIRA_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"body": "Upload completed successfully"}' \
     "https://your-jira-instance/rest/api/2/issue/PROJ-1234/comment"
```

!!! warning "常见坑：rate limit"
    Jira instance 会对 API 有 rate limit（每分钟多少次 request，search endpoint 一般比
    单条 issue 的 GET 卡得更严）。如果一条 pipeline 太频繁地 poll 或者 search，很容易撞上
    这个限制——这也是为什么事件驱动的做法（webhook，或者把数据往前传而不是事后回查）比
    反复 polling 更好的一个原因。

!!! tip "How this applies to the capstone"
    这就是（已经被否掉的）Option B 设计里，本来打算在每次 scan 完成之后调的那个 API——
    去搜那张 ticket，读它的 destination 字段。搞清楚这次调用到底要花多少代价，正是
    [4.4](4-4-why-querying-is-expensive.md) 要讲的铺垫。
