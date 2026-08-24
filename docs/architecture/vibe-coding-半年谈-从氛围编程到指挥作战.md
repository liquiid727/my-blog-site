---
title: Vibe Coding 半年谈：从氛围编程到指挥作战
date: 2026-06-30
slug: vibe-coding-half-year-review
tags:
  - agent
  - vibe-coding
  - architecture
  - workflow
  - spec
status: published
visibility: public
summary: 半年前还在"氛围编程"，半年后已经在指挥 Agent 小队协同作战。这篇是 Claude Code、Codex 等 AI 编程工具的实战总结——从 Spec 到 AGENTS.md，从上下文管理到 Worktree 并行开发。
---

# Vibe Coding 半年谈：从氛围编程到指挥作战

> 半年前，我还只是在"氛围编程"；半年后，我已经在指挥一支 Agent 小队协同作战了。这篇文章是我这半年使用 Claude Code、Codex 等 AI 编程工具的实战总结，希望能帮到同样在摸索的你。

---

## 一、Vibe Coding 的蜕变

Vibe coding 的内涵已经发生了根本性的变化。

最初它意味着一种模糊的"氛围编程"——你给 AI 一句话，闭眼等结果，心里默念"差不多得了"。而现在，它已经演化为**指挥一个或多个 Agent 协同作战**。

换句话说，你手底下有一批小兵了。你得当将军——负责发号施令、制定战术、审查战果。

而不会发号施令的将军，只会把整个战场带进沟里。

### 如何指挥好 Agent？做到指哪打哪？

**1. 写清楚 Spec**

不要只告诉 Agent "做个登录"，而是按系统设计的规范把功能描述罗列清楚：

> "增设 `auth/` 下的 QQ、微信登录模块，采用 xxx 方案，使用 unionid 进行存储……"

模糊的指令产出模糊的代码。Spec 越精确，Agent 的输出越可控。

**2. 输入合适的上下文**

除了功能需求，Agent 还需要大量的背景信息：业务逻辑、团队规范、技术约束。这些通过 spec 文件、`agents.md`、文件路径等方式来引导。Agent 不是读心术大师，它只是一个勤快但需要明确上下文的执行者。

**3. 人工审查、引导、修正**

几个实用原则：

- **可以不看代码，但一定要看 diff**：关注 Agent 改了什么、改动是否合理
- **亲手验证**：跑一下，看这一步能不能正常工作
- **追问设计**：问 Agent "你打算怎么做？为什么这样做？还有其他方案吗？"甚至可以注入不同的视角让它们 battle：
  - A：你是乐观派
  - B：你是悲观、严谨派
  - "你们都对这个方案考虑一下"

**4. 善用 Plan 模式**

尤其是对不熟悉的领域，让 Agent 动手之前先规划：

> 我想做 X。在你动手之前，先告诉我：
> 1. 你打算怎么做（分几步、改哪些文件）
> 2. 你需要我决定什么（选 A 还是 B 方案）
> 3. 你看到了什么我没看到的问题或边界 case

这不是形式主义。Plan 模式的核心价值是**让你在 Agent 动手之前就发现方向性错误**。

**5. 止损机制**

给了不够全面的需求就让 Agent 跑下去，一连串改了很多东西，还没做 git 记录——这种场景太常见了。

Claude Code 的 **rewind 命令**可以回滚到之前的状态。但更好的习惯是：每完成一个阶段就 commit，把"可回滚点"主动留下来。

**6. 适时拆解任务、压缩上下文、切换 Agent**

随着主流 code agent 不断进化，上下文管理的问题未来大概率会被模型本身解决。但目前，人工干预仍然不可少。

---

## 二、Spec：把模糊的愿望写成可执行的规范

Spec 的本质是：**将模糊的愿望，写成背景、目标、方案和验收标准。**

三条基本要求：

- **意图清晰**——你到底要什么
- **约束明确**——什么不能碰
- **验收可测**——怎么算做完了

市面上比较常见的是 OpenSpec，但 IDE 与 spec 的适配还不够成熟。后续我会结合 Agent 的特点，单独写一篇 spec 实践（也可以参考我之前写的 [Spec 不是一份会越写越乱的需求文档](spec-不是一份会越写越乱的需求文档.md)）。

常见的 spec 工具：

- [brainstorming.skill](https://github.com/obra/superpowers/blob/main/skills/brainstorming/SKILL.md)
- OpenSpec
- [writing-plans.skill](https://github.com/obra/superpowers/blob/main/skills/writing-plans/SKILL.md)

---

## 三、AGENTS.md / CLAUDE.md：给 Agent 看的 README

人类读 `README.md`，Agent 读 `AGENTS.md`。

需要注意：Claude Code 不会默认读 `AGENTS.md`。官方推荐在 `CLAUDE.md` 首行写 `@AGENTS.md` 来引用。如果你是模型混用的，统一配置会省很多事。

### 这个文件应该放什么？

经验值：**200-400 行**最佳。太短没信息量，太长 Agent 抓不到重点。每隔几周回顾一次，把"Agent 又踩的坑"补进去。

推荐六大模块：

**1. 项目身份**

```
## 项目简介
这是一个 B2B SaaS 的后端，Go 1.22 + PostgreSQL + Redis，
部署在 AWS ECS Fargate，前端在另一个仓库。
```

**2. 目录地图**

这是最重要的部分。Agent 对项目结构的理解直接决定了它的代码质量：

```
## 代码结构
- cmd/api/           HTTP 入口
- internal/auth/     认证，改这里要看 SECURITY.md
- internal/billing/  计费，所有金额用 decimal，不要用 float
- pkg/               可被外部引用的工具，慎改
- migrations/        数据库迁移，只增不改，用 goose
```

**3. 代码风格与约定**

```
## 编码规范
- 错误处理：用 fmt.Errorf("doing X: %w", err) 包装
- 日志：用 slog，不要用 fmt.Println
- 测试：表驱动，文件名 _test.go
```

**4. 命令清单**

让 Agent 不用猜：

```
## 常用命令
- 运行测试: make test
- 启动本地: make dev（会起 docker compose）
- Lint: make lint（必须通过才能提 PR）
```

**5. 红线**

```
## 禁止事项
- 不要直接改 migrations/ 下已存在的文件
- 不要在生产代码里加 TODO，要么做要么开 issue
- 涉及 PII 的字段必须走 internal/crypto/pii.go
```

**6. 历史教训**

```
## 容易踩的坑
- Postgres timezone 默认是 UTC，但 API 返回要用用户时区
- Redis key 必须带 namespace 前缀，见 internal/cache/keys.go
```

### 一个重要提醒

初版不要追求完美，先保证**有**，后面边用边迭代。好用的 AGENTS.md 是养出来的，不是一次写好的。

---

## 四、常见场景

### 创建新项目

先用脚手架搭好项目骨架，再根据脚手架规范让 Agent 生成 agents.md。

### 接手别人的项目

先用 `/init` 初始化项目，生成最基础的 agents.md。然后让 Agent 做深度探索：

> 你刚才生成的 AGENTS.md 是基于配置文件推断的。现在做更深的探索，不要修改任何代码，但要回答：
> 1. 画一张模块依赖图（用 mermaid）
> 2. 找出 5 个最复杂的文件，告诉我它们做什么
> 3. 翻 git log 最近 50 个 commit，看有没有反复修同一个 bug
> 4. 找出测试覆盖薄弱的地方
> 5. 列出代码里你看不懂或者觉得可疑的地方
> 6. 找出"约定但没写下来"的隐性知识：命名风格、错误处理模式、日志格式、目录潜规则、commit 约定

接手项目最大的成本不在代码本身，而在那些**隐性知识**——每个人的规范和实现逻辑完全不同。把 Agent 的考古发现合并回 AGENTS.md，让这些知识变成可复用的上下文。

---

## 五、上下文管理

上下文管理不好会带来一系列问题：

- Agent 频繁犯错
- 忘记 spec 约束或业务背景
- 调用变慢，输出质量明显下降

更危险的是：**当你发觉 Agent 变傻的时候，往往已经很糟了——因为你不知道它在"犯蠢"的过程中到底改了什么。**

### 什么时候该切换上下文？

- 某个 feature/bug 的阶段性任务完成时
- 切换话题时（debug → 部署，前端 → 后端）——可以生成 handoff 文件进行切换
- 上下文快到极限了
- 对话内容已经过于沉重

### 两种常见操作

**方式一：/compact 压缩**

将对话压缩成摘要继续聊。实践下来，压缩后容易丢失中间过程的信息（开头和结尾保留较好，取决于供应商的压缩策略）。

**方式二：阶段交接（推荐）**

完成阶段任务 → 让 Agent 写 handoff 文件 → 开新 session。或者复制会话 ID，在新会话中让模型自行压缩和提取旧会话内容。

### 容易被忽略的上下文消耗

工具的 schema 本身就在消耗上下文。一个工具 schema 约 200-800 token，而 Claude Code 和 Codex 内置了几十个工具 + MCP + Plugin，这些固定开销每次都会发送。

**解法**：关掉不需要的 MCP 和工具，保持精简。

---

## 六、MCP / Skill / SubAgent / Workflow

### 常用的 MCP

| MCP | 用途 |
|-----|------|
| Sentry MCP | 生产环境错误堆栈 |
| Github MCP | 仓库操作 |
| Postgres MCP | 数据库操作 |
| Playwright MCP | 端到端测试 |

MCP server 启动后，会向 Agent 注册一组工具（每个工具有 name、description、参数 schema）。Agent 在需要时调用它们，和调用内置的 Read、Bash 没有区别。配置一般存储在 `.mcp.json` 文件中。

---

## 七、并行开发：Spec + Worktree + SubAgent

### SubAgent：隔离上下文的利器

SubAgent 的核心价值是**控制上下文膨胀**。读仓库代码、扫描文件、读取日志这类场景，开启 SubAgent 去处理，只把结论带回来，而不是把原始信息塞入主 Agent。

> 原则：只有涉及到过长上下文才开 SubAgent，能一个 Agent 处理的就不要分兵。

### SubAgent 的三种实战模式

- **扇出（fan-out）**：主 Agent 发现需要并行做 5 件事，同时派 5 个 SubAgent，等都返回后汇总
- **深度委托**：主 Agent 保持高层视角，把具体实现委派给 SubAgent，只接收结论
- **专家 SubAgent**：预定义角色（code-reviewer、security-auditor、test-writer），每个有自己的系统提示。审 PR 时自动调 code-reviewer，专注审查不分心

我个人偏好专家 SubAgent 模式，结合 spec 设置增设 Agent 注册列表来插拔使用。

### Skill：把重复的事情交给 AI 脚本化

Skill 的本质就是 workflow 的自然语言版——以前把重复的事情写成脚本很麻烦，现在只需要把流程描述给 AI。

几个经验：

1. **不要盲目追 Skill**。用不上的 Skill 只会占上下文，拉低 Agent 的表现。
2. **一件事做过 3 次以上，且每次都给 Agent 讲同样的话**，就该做成 Skill。
3. 常见 Skill 场景：重构老项目（同一套改法）、写周报、Review / CI / CD / Deploy。

推荐用 [skill-creator.skill](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md) 来写 Skill——用 Skill 写 Skill。

**一个实用技巧**：让 Agent 审查过去的 50 次 session，将重复的过程和模式提炼成 Skill。

---

## 八、工程约束：Git + CI/CD + Hooks

### CI 在 Vibe Coding 里要拦什么？

CI 是"机器审查官"——它看不懂业务，但能死死盯住格式和规范问题。建议在 agents.md 中增设 CI 说明，对应的检查用脚本形式（比如 make）：

```
## CI 规则

1. 提交前必须本地跑过 make ci-local
   - 这等价于 CI 上跑的所有检查
   - 本地 30 秒内能跑完，比等 CI 5 分钟快得多

2. 如果 CI 挂了：
   - 先看哪一步挂的，贴出失败日志
   - 不要"猜测式修复"——先复现再修
   - 修完之后本地跑一次 ci-local 再 push
```

### AGENTS.md / Hooks / CI 的关系

三者是叠加关系：

- `AGENTS.md` 告诉 Agent **应该怎么做**
- Hooks 拦住 Agent **不该做的**
- CI 验证 Agent **做完后对不对**

### Git Worktree 并行开发

现在一个人可以做到类似团队并行开发的效果。对于互相隔离的模块，完全可以并行推进。

**适用场景：**

- 有 2 个以上互不相关的任务想并行做（auth + payment + dashboard）
- A/B 实验："用两种方案各做一遍，比较结果"
- reviewer Agent 和 coder Agent 同时工作

**注意事项**：worktree 里被 `.gitignore` 忽略的文件不会自动带过来。尤其是 `.env` 文件，会导致新的 worktree 跑不起来。解法是准备一个 setup 脚本：

```bash
# scripts/setup-worktree.sh
cp /path/to/main-repo/.env .
ln -s /path/to/main-repo/node_modules .
```

### 并行 Worktree 工作流

1. **准备**：把任务拆成 N 个独立单元，检查是否会改同一个文件。为每个任务写好 spec
2. **派发**：创建 N 个 worktree + 分支，每个跑 setup 脚本，启动各自的 Agent
3. **监督**：不用盯着每一个，偶尔切过去看进度、回答问题
4. **收割**：review diff → 跑测试 → 合并到 main → 删 worktree

---

## 九、测试：让 Agent 助力可交付的代码

推荐测试驱动的流程：

1. 从 spec 提取对应的测试标准
2. 让 Agent 生成测试计划（happy path、边界、失败 case）
3. 审核测试计划——检查覆盖度，关注 Agent 常见错误（权限、幂等、空输入）
4. Agent 执行写测试
5. 跑测试

Prompt 模板：

> 读 `specs/003-data-export.md`。先不要写实现。
>
> 任务：
> 1. 从 spec 提取所有验收标准，列成 checklist
> 2. 为每个标准设计测试 case（输入 → 期望输出）
> 3. 列出 Agent 在这个项目里容易犯的错，各加一个防呆 test
> 4. 输出测试计划给我审；我同意后再写测试代码

### 抓住 Agent 的错误

Agent 的错误经常"看起来很合理"——结构正确、命名合理、注释到位，但跑起来是坏的。

对抗这个的唯一办法是测试驱动：要么你写测试 Agent 实现，要么 Agent 写测试你审，然后跑出来看绿。

这些测试不是仪式感——它们是在把 Agent 最容易糊弄过去的项目知识写成**可执行的约束**。

---

## 十、一些实用技巧

**"让 Agent 先说"原则**

任何重要任务，第一步永远是让 Agent 复述和规划，不要直接动手：

```
// 不推荐
给我加一个用户搜索 API

// 推荐
我想加一个用户搜索 API。先告诉我：
- 你打算怎么实现
- 会改哪些文件
- 有哪些边界 case
- 你需要我决定什么
```

**拒绝"看起来对"的代码**

Agent 会写出结构正确但逻辑错误的代码——看起来很专业，但跑起来是坏的。对抗这个的唯一办法是：测试驱动。

---

> **参考来源**：[Vibe Coding Guide](https://lling0000.github.io/Vibe_coding_guide/?lang=zh&day=1&view=reader&chapter=1)，结合 Codex / Claude 使用经验及个人实践总结。
