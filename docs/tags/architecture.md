---
title: 标签 - architecture
date: 2026-08-25
slug: tag-architecture
tags:
  - architecture
status: published
visibility: public
summary: 带有 architecture 标签的公开文章。
---

# architecture

共 6 篇文章。

- 2026-06-30 [Vibe Coding 半年谈：从氛围编程到指挥作战](../architecture/vibe-coding-半年谈-从氛围编程到指挥作战.md) — 半年前还在"氛围编程"，半年后已经在指挥 Agent 小队协同作战。这篇是 Claude Code、Codex 等 AI 编程工具的实战总结——从 Spec 到 AGENTS.md，从上下文管理到 Worktree 并行开发。
- 2026-05-15 [钱包扣费引擎：Redis 管余额，DB 管账单，日志管追凶](../backend/wallet-billing-engine-redis-db-loki.md) — 多连接时长计费不应该靠 tick 流水硬扛，而应该把实时余额、最终账单和审计日志拆成三套职责。
- 2026-05-15 [奖励发放中台架构：别让活动系统直接改用户资产](../architecture/reward-distribution-platform-architecture.md) — 把签到、广告、任务、活动、订单和后台补偿统一收敛到 Reward Service，用奖励包、幂等、流水、重试和对账能力守住发奖这条业务资金链。
- 2026-05-14 [别再对 AI 说“做得好看一点”了：面向 LLM 的 UI Spec 应该怎么写](../architecture/specos-target-ui-agent-ui-spec-spec-agent-template.md) — 面向 LLM 写前端，核心不是审美词，而是把页面写成状态、行为、反馈和约束，再把它固化成 UI Spec、Target 和 Agent 边界。
- 2026-05-14 [Spec 不是一份会越写越乱的需求文档](../architecture/spec-不是一份会越写越乱的需求文档.md) — 当需求会变、代码会漂、补丁会插队时，真正能帮 agent 干活的不是一份大而全文档，而是一套分层上下文系统。
- 2026-03-14 [从 Private Vault 到 Public Blog 的最小发布流水线](../architecture/private-vault-public-blog-pipeline.md) — 用 Blog/public、白名单资源和导出脚本，把私有 Obsidian 仓库改造成可发布的公开博客。
