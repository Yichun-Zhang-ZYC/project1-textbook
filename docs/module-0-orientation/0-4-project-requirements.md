# 0.4 · Capstone 场景

这一页把整个网站贯穿始终的 running example 完整讲一遍——一个真实自动化问题的通用版本，
所有能对应到具体身份的信息都已经去掉了。后面每个 module 都会回过头来引用这个场景，
简称 "the capstone"。

## 目标

把 artifact upload 这件事从头到尾自动化：外部的 artifact 自动被 **scan**、**approve**、然后
**upload 到 artifact registry**——中间不需要人工开 ticket 交接。

## 现在被取代的人工流程

1. Requester 开一张 ticket，让 security team 帮忙 scan 一个 artifact。
2. Security team review；scan 过了之后，requester 再开**第二张** ticket 给 platform team，
   附上前面 security 的 approval。
3. Platform team 人工做 upload。具体的 upload 步骤因 artifact 类型而不同（见
   [0.3 · Artifact 类型](0-3-artifact-types.md)）。

两张 ticket，两个 team，最后一步完全靠人工。

## 已经自动化的部分

这条 pipeline 里 security-scanning 的那一半已经搭好了：scanning 已经接到 ticket 系统上，
相关 ticket 一进来就会自动 trigger scan，结果（pass/fail）会自动写回 ticket。
**缺的那一半是 upload 这一步。**

## 为什么 "scan 完了直接去读 ticket" 这个思路行不通

最直接的想法是：让 upload automation 盯着同一张 ticket，一旦看到 "scan passed"，就去读
destination，然后跑 upload。当时讨论了三种架构方案怎么把这个串起来——下面用通用的方式描述，
因为真正有价值的是这些 tradeoff 本身，而不是具体谁提的。

### Option A — Parent job wrapper

一个 parent CI job 自己去调 security-scan job，然后——根据 scan 的输出——决定要不要继续
upload。它拿一组 artifact 加一个 destination，对每个 artifact 跑一次 scan，scan 过了就把它
放到指定位置。这要求 scan job 在自己的输出里，除了 pass/fail 之外，还要多带一个
destination 字段。

### Option B — Extra intake field on the ticket

在 ticket 的 intake 表单上加一个字段，用来填 upload 的目标位置。Scan 完成之后，upload
automation 去读**同一张** ticket，检查 approval 状态和 destination 字段，然后跑 upload。

这个思路听起来很简单，但最后被否掉了——完整的技术原因见
[4.4 · 为什么 Query 很贵](../module-4-jira/4-4-why-querying-is-expensive.md)。简单说就是：
**搜 ticket 这件事本身又慢又贵**，而且 scan 一结束 ticket 往往就被**自动 close** 掉了（实际上
很难再查）——所以根本没有一个靠谱的时间窗口，能让你事后回去找它。

### Option C — Boolean parameter on the main build（最后选的方案）

给 main build 加一个 boolean parameter——就叫它 `push_to_artifactory`。如果是 `true`，build
就 trigger 一个**还没实现（NYI）的 downstream job**，直接把所需要的一切都当 build parameter
传过去：repo 位置、binary 的细节、artifact 类型，以及 destination path（这个 path 本身也是在
intake 时候多加的一个字段，跟 Option B 一样——但区别是它会被立刻当 parameter 消费掉，
之后再也不用回头去 Jira 查）。

这样就完全绕开了 Option B 的问题：不需要事后去搜任何 ticket。Destination 的数据从一开始就
跟着这次 build 一起流转，downstream job 由 *platform team 自己写*，负责真正执行 upload。
还留着一个悬而未决的设计问题：这个 "要不要跑" 的判断，是不是也应该作为 ticket 层面的
metadata 存下来——"只 scan 这个 artifact，但不 upload" 这种 case 实际上有多常见，目前还不清楚。

这个方案会在 [Module 7 · Capstone](../module-7-capstone/7-1-option-c-mock-implementation.md)
里从头到尾完整实现一遍。

## 四种 Artifact 类型

完整拆解见 [0.3 · Artifact 类型](0-3-artifact-types.md)：generic binary、Terraform module、
Docker image、Helm chart，每种 upload 的方式都不一样。

!!! tip "How this applies to the capstone"
    这一页*就是* capstone 本身。后面每个 module 要么是在教一个前置技能（Module 1-3、6），
    要么是在直接填这张图里的某一块（Module 4-5），最后在 Module 7 把 Option C 完整地
    mock 实现一遍。
