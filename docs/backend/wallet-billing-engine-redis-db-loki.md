---
title: 钱包扣费引擎：Redis 管余额，DB 管账单，日志管追凶
date: 2026-05-15
slug: wallet-billing-engine-redis-db-loki
tags:
  - backend
  - redis
  - billing
  - concurrency
  - architecture
status: published
visibility: public
summary: 多连接时长计费不应该靠 tick 流水硬扛，而应该把实时余额、最终账单和审计日志拆成三套职责。
---

# 钱包扣费引擎：Redis 管余额，DB 管账单，日志管追凶

我之前看时长计费，脑子里很容易蹦出一个朴素方案：

每隔几秒扣一次，然后每次都写一条流水。

听起来很踏实，像是在账本上认真记账。但真正放到多连接场景里，这个方案很快就开始露出疲态：10 台机器同时扣、每个在线用户每 5 秒或 10 秒 tick 一次、前端还要实时展示剩余时长。DB 一边要扛高频写入，一边还要保证余额展示及时，最后谁都不舒服。

这次我真正想明白的是：钱包扣费系统里，不能让一个存储同时承担所有职责。

核心逻辑应该拆开：

```text
Redis = 真钱，负责实时余额
DB    = 账单，负责事后对账
日志  = 黑匣子，负责排查和审计
```

这句话看起来很短，但它基本决定了整个扣费引擎的设计方向。

## 问题不是扣费，而是多连接实时扣费

时长计费的业务规则其实不复杂。

系统在 account 级别维护唯一余额：

```text
balance_seconds
```

所有客户端看到同一个账户余额：

```text
total_remaining_time = balance_seconds
```

如果一个账户同时开了多个连接，预计可用时长要按当前消耗速度折算：

```text
estimated_remaining_time = balance_seconds / active_sessions
```

如果不同 session 有不同倍率，就不能只除以连接数，而要除以当前总消耗速率：

```text
estimated_remaining_time = balance_seconds / sum(session_rate)
```

当没有连接时：

```text
estimated_remaining_time = total_remaining_time
```

前端展示也很明确：

- 剩余总时长：账户里还剩多少秒
- 预计可用时长：按当前连接数和倍率还能撑多久
- 当前连接数量：当前在线使用机器数

真正麻烦的是系统行为：

- 余额用完必须立刻踢人，不能等 DB
- 多台机器同时扣费必须原子
- 每隔几秒扣费一次，DB 不能被 tick 流水打穿
- 服务挂了以后，已经扣掉但没落 DB 的账还要能恢复

这几个约束叠在一起，就不能把 DB 当作实时余额真相。

## 第一条原则：余额强一致在 Redis

余额这种东西，在用户连接中就是实时控制阀。

余额用完，要立刻拒绝新连接，或者踢掉正在使用的连接。如果还要先写 DB、等事务提交、再回查状态，整个控制链路就变钝了。

所以余额真相应该放在 Redis：

```text
wallet:balance:{account_id} = balance_seconds
```

扣费时通过 Lua 脚本完成三件事：

1. 读取当前余额
2. 判断是否足够扣费
3. 原子扣减余额，并累计 session 的 pending 消费

一个简化版逻辑是：

```lua
local balance_key = KEYS[1]
local pending_key = KEYS[2]
local cost = tonumber(ARGV[1])

local balance = tonumber(redis.call("GET", balance_key) or "0")

if balance <= 0 then
  return {0, balance}
end

local charged = cost
if balance < cost then
  charged = balance
end

redis.call("DECRBY", balance_key, charged)
redis.call("HINCRBY", pending_key, "consumed_seconds", charged)

return {charged, balance - charged}
```

这段脚本不是完整生产代码，但它表达了关键点：扣余额和累积 pending 必须在 Redis 内部一次完成。

这样才能保证 10 台机器同时扣同一个账户时，不会出现重复消费、余额扣成不可控负数、或者扣了余额但 pending 没记录的割裂状态。

## 第二条原则：DB 不记 tick，只记 session 账单

很多计费系统一开始会想记录每一次 tick：

```text
usage_tick(account_id, session_id, cost_seconds, created_at)
```

这很直觉，但不适合这个场景。

如果每 5 秒扣一次，1000 个在线 session 一分钟就是 12000 条写入。并发继续放大以后，DB 的压力会变成系统设计自己制造出来的噪音。

更合理的做法是：DB 只保存可追溯的最终账单。

比如 session 表记录连接生命周期：

```text
session_id
account_id
start_time
end_time
status
rate
```

usage_record 表记录最终消费：

```text
usage_record_id
account_id
session_id
consumed_seconds
started_at
ended_at
settled_at
settlement_source
```

也就是说，DB 不需要知道每 5 秒发生了什么。它只需要在 session 结束时知道：

```text
这个 session 最后消耗了多少
```

账单可以从两个来源生成：

1. Redis pending 池里的累计消费
2. session.start_time / session.end_time 按倍率反推出来的消费

第一种是主路径，第二种是恢复和对账路径。

## 第三条原则：pending 是崩溃恢复的中间态

最容易被忽略的问题是：

```text
Redis 扣了钱，但 DB 还没落账，服务挂了怎么办？
```

如果架构只做了“扣 Redis，然后写 DB”，那中间一挂，钱已经扣了，但账没有记。这个时候 DB 看起来少了一笔账，用户又不能平白把钱拿回来。

所以需要一个未结算消费池：

```text
billing:pending:{session_id}
```

里面至少记录：

```text
account_id
session_id
consumed_seconds
last_charged_at
rate
```

每次扣费时：

```text
1. Redis 扣 balance_seconds
2. Redis pending += charged_seconds
```

session 正常结束时：

```text
1. 读取 billing:pending:{session_id}
2. 写入 DB usage_record
3. 标记 session ended
4. 删除 pending
```

如果服务挂了，pending 还在 Redis 里。恢复以后可以继续累计，也可以由结算任务强制收口。

这个 pending 池就是 Redis 余额和 DB 账本之间的缓冲层。

```text
Redis balance
     ↓
pending consumption
     ↓
session end / settlement job
     ↓
DB usage_record
```

它不追求每一秒都和 DB 完全一致，但它保证最终能把账补上。

## 实时展示应该从余额和活跃连接推导

获取剩余时长、时长策略、SSE 推送这些接口，不应该再只返回一个孤零零的剩余秒数。

在不破坏原有接口的前提下，可以新增这些字段：

```json
{
  "total_remaining_time": 3600,
  "estimated_remaining_time": 1800,
  "active_sessions": 2,
  "active_rate": 2,
  "summary": "当前连接数 2，预计可用剩余时间约 30 分钟"
}
```

如果 session 倍率不一致，就把 `active_rate` 当作消耗速率总和：

```text
active_rate = sum(active_session.rate)
estimated_remaining_time = balance_seconds / active_rate
```

例如：

```text
session A rate = 1.0
session B rate = 1.3
active_rate = 2.3
```

这时预计可用时长不是 `balance_seconds / 2`，而是：

```text
balance_seconds / 2.3
```

这里的重点不是前端多展示几个字段，而是让用户看到的是同一个账户余额在当前消耗速度下的真实剩余能力。

## 结算任务负责把异常 session 收口

只靠 session 正常结束来写 usage_record 还不够。真实系统里一定会有异常：

- 客户端断连，但服务没收到干净的 end 事件
- worker 挂了，pending 还留在 Redis
- session 状态仍是 active，但 heartbeat 已经超时
- DB 写入失败，需要重试结算

所以扣费引擎需要一个结算任务，定期扫描异常状态。

可以按这几类处理：

```text
active but heartbeat timeout
  -> 强制结束 session
  -> 读取 pending
  -> 写 usage_record
  -> 清理 pending

ended but pending exists
  -> 重试写 usage_record
  -> 成功后清理 pending

usage_record exists but pending exists
  -> 幂等检查
  -> 清理或补偿
```

这里最重要的是幂等。

`usage_record` 应该对 `session_id` 做唯一约束，避免结算任务重试时重复写账单。结算任务宁可多跑几次，也不能多记几笔。

## 日志不是账本，但必须能还原现场

Loki 或 Elasticsearch 不应该承担余额真相，也不应该成为主账本。

但日志必须能帮助排查这些问题：

- 某个 session 为什么被踢
- 某次扣费扣了多少
- Redis 余额变化前后是多少
- pending 是否成功累计
- DB 结算是否失败
- 某个 account 的活跃 session 当时有哪些

建议每次扣费至少打结构化日志：

```json
{
  "event": "billing.tick",
  "account_id": "acc_xxx",
  "session_id": "sess_xxx",
  "charged_seconds": 10,
  "balance_after": 3590,
  "pending_consumed_seconds": 120,
  "rate": 1,
  "worker_id": "billing-worker-01"
}
```

日志的定位是黑匣子。平时不靠它跑主流程，但真的出现账单争议或者恢复边界问题时，它要能帮我们把现场拼回来。

## 最后形成的扣费引擎

最后，这套设计可以收敛成四条链路。

### 1. 查询余额

```text
读取 Redis balance
  -> 如果不存在，从 DB 初始化
  -> 计算 active_sessions / active_rate
  -> 返回 total_remaining_time / estimated_remaining_time / summary
```

如果 Redis 里没有余额，不代表余额为 0，而是需要从 DB 的账户余额或最近一次结算状态初始化。初始化之后，Redis 才继续承担实时余额。

### 2. 发起连接

```text
检查 Redis balance
  -> 创建 session
  -> 注册 active session
  -> 开始 billing tick
```

余额不足时直接拒绝。余额足够时才分配连接。

### 3. 在线扣费

```text
每 5s 或 10s
  -> Lua 原子扣 Redis balance
  -> pending += charged_seconds
  -> balance 用完则触发踢人
  -> 推送余额变化
```

这一层只做实时控制，不把每次 tick 写成 DB 流水。

### 4. 结束和恢复

```text
session end
  -> pending 写入 usage_record
  -> 删除 pending

settlement job
  -> 扫描超时 session
  -> 强制结算
  -> 对账修复
```

这套链路背后的判断是：

```text
余额要强一致，所以放 Redis。
流水要可追溯，所以放 DB。
事故要能复盘，所以进日志。
```

## 我真正学到的

以前我会本能地想：账单系统是不是每一步都要落 DB，才算安全？

现在反过来看，这种想法有点像“为了安心，把所有压力都塞给数据库”。真正的安全不是每 5 秒写一条流水，而是把实时控制、最终结算和异常恢复拆清楚。

Redis 负责把人及时踢下线。

DB 负责让账最后能对上。

Loki 负责在系统出事时告诉我：当时到底发生了什么。

这才是一个时长钱包扣费引擎该有的样子。不是每个 tick 都写进账本，而是每一笔钱都能被控制、结算和追溯。
