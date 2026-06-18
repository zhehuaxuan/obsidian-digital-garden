---
{"dg-publish":true,"permalink":"/src/site/notes/public/src/site/notes/src/site/notes/Learning（学习）/03-技能学习/02-Harness工程/01-harness工程学习资料/","dg-note-properties":{"permalink":"/public/src/site/notes/src/site/notes/Learning（学习）/03-技能学习/02-Harness工程/01-harness工程学习资料/"}}
---





# AI Agent Harness 工程资料

> 最后更新：2026-06-16
> 整理者：浙华 🦞
> 主题：AI Agent Harness 工程——给 LLM 建造"操作系统"


---

## 目录

- [一、核心概念：什么是 Harness？](#一核心概念什么是-harness)
- [二、核心理论框架](#二核心理论框架)
- [三、主流框架与工具](#三主流框架与工具)
- [四、工程实践模式](#四工程实践模式)
- [五、具象化场景：如何用 Harness 工程确保 Agent 可控](#五具象化场景如何用-harness-工程确保-agent-可控)
  - [5.1 场景一：Stripe Minions — "无人值守编码 Agent"的全链路可控](#51-场景一stripe-minions--无人值守编码-agent的全链路可控)
  - [5.2 场景二：ByteDance DeerFlow — "执行优先"的 SuperAgent Harness](#52-场景二bytedance-deerflow--执行优先的-superagent-harness)
  - [5.3 场景三：OpenAI Codex CLI — "无状态"的生产级 Agent Loop](#53-场景三openai-codex-cli--无状态的生产级-agent-loop)
  - [5.4 横向对比：三个案例的 Harness 设计选择](#54-横向对比三个案例的-harness-设计选择)
- [六、关键工程问题](#六关键工程问题)
- [六、学习资源汇总](#六学习资源汇总)

---

## 一、核心概念：什么是 Harness？

### 1.1 一句话定义

**Harness（套具）= 围绕 LLM 的一切基础设施，把 LLM 变成可靠 Agent 产品。**

在 AI Agent 领域，"Harness" 指的是把大语言模型（LLM）包装成可信赖的生产级 Agent 所需要的全部工程组件——包括工具调度、上下文管理、权限控制、错误恢复、状态管理等。

LLM是引擎+Harness是规则，Harness能够确保Agent Runtime的稳定性和效果。

### 1.2 核心类比

> **Model = CPU，Context Window = RAM，Harness = 操作系统，Agent = 应用程序**

- 你不会把 CPU 直接卖给终端用户，你需要的是操作系统
- 同样，AI Agent 的核心竞争力不在于模型本身，而在于 Harness 的质量
- Meta 以约 **$2B 收购 Manus**，买的不是模型（Manus 使用 Anthropic 和 OpenAI 的基础模型），而是 **Harness**

> 来源：https://www.morphllm.com/agent-engineering

### 1.3 为什么 Harness 价值连城

| 数据 | 含义 |
|------|------|
| ~98.4% | 生产级 Agent 代码中，Harness 基础设施占比（权限、上下文、沙盒、工具路由、恢复） |
| ~1.6% | 实际 AI 决策逻辑占比 |
| $2B | Meta 为 Manus 的 Harness 支付的价格 |

> 2026年5月 MBZUAI 研究：分析 Claude Code 源码（1,884 文件，约 512k 行代码），四个独立团队（Claude Code、Codex CLI、Aider、OpenClaw）都收敛到相同的 Harness 架构模式——说明这是问题的本质约束，而非设计选择。

---

## 二、核心理论框架

### 2.1 Martin Fowler — Harness 工程框架

Martin Fowler 和 Birgitta Böckeler（ThoughtWorks）在 2026 年提出的框架，核心是**前馈（Feedforward）+ 反馈（Feedback）** 两条控制链路：

#### 前馈（Guides）— 预防控制
在 Agent 行动**之前**引导它，提高一次做对的概率：
- 代码规范（AGENTS.md、Skills）
- 项目初始化指引（bootstrap 脚本）
- 代码规范文档

#### 反馈（Sensors）— 纠正控制
在 Agent 行动**之后**观察并帮助它自我纠正：
- 计算型：Linter、类型检查、单元测试、结构测试（确定性、毫秒级）
- 推理型：AI 代码审查、"LLM as judge"（需要 GPU、较慢、非确定性）



| 类型                               | 职责                                        | 成熟度        |
| -------------------------------- | ----------------------------------------- | ---------- |
| **Maintainability Harness**      | 调控代码质量和可维护性（圈复杂度、重复码、架构漂移）                | 最成熟，工具链丰富  |
| **Architecture Fitness Harness** | 定义和检查架构特征（Fitness Functions），如性能测试、可观测性标准 | 中等         |
| **Behaviour Harness**            | 引导和感知应用功能行为（功能测试覆盖）                       | 最不成熟，是核心难题 |
|                                  |                                           |            |

> 来源：https://martinfowler.com/articles/harness-engineering.html
   关联笔记： [Learning（学习）/01-读书笔记/01-关于Martin Fowler的《harness-engineering》.md](../../01-读书笔记/01-关于Martin%20Fowler%20的《harness-engineering》的解读.md)
#### 三类 Harness
### 2.2 swyx — IMPACT 框架

swyx 在 2025 AI Engineer Summit 提出，因"LLM + Tools + Loop"定义太简化而创建。六个组件：

| 组件 | 含义 |
|------|------|
| **I**ntent | 目标定义，通过 Evals 验证 |
| **M**emory | 技能库中的连贯性记忆 |
| **P**lanning | 可编辑的多步骤计划 |
| **A**uthority | 信任和权限模型 |
| **C**ontrol Flow | LLM 驱动的执行路径 |
| **T**ools | RAG、沙盒、浏览器自动化 |

> 来源：https://www.latent.space/p/agent
   关联笔记： [Learning（学习）/01-读书笔记/02-关于swyx的《Agent Engineering》的解读](../../01-读书笔记/02-关于swyx的《Agent Engineering》解读.md)
### 2.3 Anthropic — 长周期 Agent Harness

Anthropic 工程团队提出的模式，解决跨多个上下文窗口工作的 Agent 问题：

**核心原则：每个 Session 只做一件事（One feature per session）**

- **Initializer Agent**：初始化环境（init.sh、进度文件、初始 git commit、功能列表）
- **Coding Agent**：读取进度文件和 git log 理解上下文，选择最高优先级功能，实现后更新文档
- 干净的 commit 状态确保代码在上下文窗口结束前是生产就绪的

> 来源：https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents

---

## 三、主流框架与工具

### 3.1 框架分类（不是所有"Agent 框架"都一样）

| 类别                     | 定义                       | 控制者    | 代表                                                      |
| ---------------------- | ------------------------ | ------ | ------------------------------------------------------- |
| **Agent Harness**      | 运行时容器——生命周期、治理、工具访问、策略执行 | 平台     | GitHub Copilot, Bedrock Agents, Vertex AI Agent Builder |
| **Agent Framework**    | 在代码中组合 Agent 的可编程构建块     | 开发者    | LangChain/LangGraph, CrewAI, AutoGen, Semantic Kernel   |
| **Agent SDK**          | 绑定到厂商 Harness 的轻量客户端库    | 厂商运行时  | OpenAI Agents SDK, Google ADK                           |
| **Agent Orchestrator** | 并行运行多个 Harness 的控制平面     | 编排层    | Warp Oz                                                 |
| **IDE Agent**          | 嵌入代码编辑器的 AI 助手           | IDE 厂商 | Cursor, Devin Desktop, JetBrains AI                     |

> 来源：https://htek.dev/articles/all-agent-harnesses-live-comparison

### 3.2 Agent Framework 详细对比

| 框架                      | 最适合                                     | 学习曲线 | 生产就绪   | 许可         |
| ----------------------- | --------------------------------------- | ---- | ------ | ---------- |
| **LangChain/LangGraph** | 自定义流水线，9000万月下载                         | 中等   | ⚠️ 需努力 | MIT        |
| **CrewAI**              | 多 Agent 协作，60% 财富500强采用                 | 简单   | ⚠️ 基础  | MIT        |
| **AutoGen**             | 企业多 Agent，合并入 Microsoft Agent Framework | 中等   | ✅      | MIT        |
| **LlamaIndex**          | RAG 和数据索引                               | 中等   | ✅      | Apache 2.0 |
| **Mastra**              | TypeScript 首选                           | 简单   | ✅      | MIT        |

**关键数据：**
- LangChain 月下载 9000 万次，GitHub 100k+ stars
- CrewAI 增长最快，60% 财富500强，每月 4.5 亿工作流
- 预计 2027 年 80% 的生产 Agent 部署将需要编排+可观测性集成
- 预计 2028 年 TypeScript Agent 框架将占据 30% 市场份额

> 来源：https://rywalker.com/research/agent-frameworks

### 3.3 主流 Agent 工具横比

| 工具 | Loop 模式 | 上下文策略 | 工具管理 | 多 Agent | 权限模型 |
|------|-----------|------------|----------|----------|----------|
| **Claude Code** | Initializer + coding agent | CLAUDE.md + 即时检索 | MCP + 懒加载（95%上下文压缩） | Agent Teams via MCP | 可配置级别 + hooks |
| **Cursor** | 模型特定 harness | Repo 索引 + 推理追踪 | 模型重命名工具 + 显式分发 | Subagent 系统并行 | 沙盒 + 文件系统/网络边界 |
| **Manus** | KV-cache 优化 ReAct | 文件系统即上下文 + todo.md | Logit masking 状态机 | 子 Agent 上下文隔离 | 沙盒环境 |
| **GitHub Copilot** | 云端自主 Agent | Extensions + MCP | Extensions API + MCP | CLI 多 Agent（任务工具） | Pre/post tool hooks + MXC |
| **Devin Cloud** | 完全自主云端环境 | 每个 Devin 独立云 IDE | 全开发环境工具 | 并行沙盒 Session | 完全沙盒，云端隔离 |

---

## 四、工程实践模式

### 4.1 测试优先开发（Test-First Development）

Simon Willison 提出的红/绿模式：

```
Red:  人类编写失败测试 → Agent 看到测试失败输出
Green: Agent 诊断问题 → 写代码 → 运行测试 → 通过
```

- 紧反馈循环，比"实现这个功能"的开放提示产生更好的代码
- Agent 看到测试失败，直接诊断，无需人工干预

> 来源：https://simonwillison.net/2026/Feb/23/agentic-engineering-patterns/

### 4.2 规范优先开发（Spec-Driven Development）

Addy Osmani 的工作流，从规范开始：

1. **spec.md**：包含需求、架构决策、测试策略的规范文档
2. **一次一个功能**：要求同时实现太多会产生"一团糟"
3. **每个块后提交**：提交是回滚的保存点
4. **每步质量门**：Linter、类型检查、测试套件
5. **多模型交叉验证**：一个模型卡住时尝试另一个

> 来源：https://addyosmani.com/blog/ai-coding-workflow/

### 4.3 上下文工程（Context Engineering）

上下文工程不是独立学科，而是 **Agent 工程的核心能力**。每个 Harness 决策都是上下文工程决策：什么 token、以什么顺序、在什么时间进入窗口。

**五个 Harness 级上下文策略（Manus 生产验证）：**

| 策略 | 说明 |
|------|------|
| 优化 KV-cache 命中率 | Agent 工作负载有约 100:1 的 prefill-to-decode 比，缓存效率是最重要的生产指标 |
| Mask 工具，不要删除 | 动态修改工具定义会使 KV-cache 失效，用 logit masking 和状态机约束行动选择 |
| 用文件系统做扩展上下文 | 大观察结果（网页、PDF）超出窗口，写入沙盒存储，保留可恢复引用 |
| 在上下文末尾背诵目标 | todo.md 在整个执行过程中更新，保持目标在模型最近注意力范围 |
| 保留错误轨迹 | 不要清理失败的尝试，保持错误信息帮助模型避免重复犯错 |

> 来源：https://www.morphllm.com/agent-engineering

### 4.4 多 Agent 编排

多 Agent 编排是 **Harness 工程问题**，不是模型问题。编排器协调并行专业化 Agent，每个有独立上下文。

**核心好处：上下文隔离**
- 单个 Agent 同时重构三个模块会让上下文充满三个模块的细节
- 三个并行 Agent，每个专注一个模块，保持干净的上下文
- 编排器只需要高层状态，不需要文件级细节

| 工具 | 架构 | 并行度 |
|------|------|--------|
| Claude Code | Agent Teams via MCP | 专业化角色，消息传递 |
| Cursor | Subagent 系统 | 主 Agent 的离散并行子任务 |
| Windsurf | Cascade via git worktrees | 5 Agent 同时处理 5 个 bug |
| Grok Build | 直接并行 | 8 Agent 同时工作 |
| Devin | 并行沙盒 Session | 每个 Devin 独立云 IDE |

---

---

## 五、具象化场景：如何用 Harness 工程确保 Agent 可控

> 方法论是骨架，真实案例是血肉。以下三个场景都来自可查证的真实系统（Stripe Minions、ByteDance DeerFlow、OpenAI Codex），每个场景都展示 Harness 工程在特定约束下如何实现控制。

### 5.1 场景一：Stripe Minions — "无人值守编码 Agent"的全链路可控

**背景约束：**
- 超过 **3 亿行代码**，横跨数个大型代码仓库
- 后端主要是 Ruby（非 Rails）+ Sorbet 类型系统，在 LLM 训练数据中几乎不存在
- 代码流经超过 **1 万亿美元**的支付交易量，任何错误都有现实代价
- 每周 **1,300+ 个 PR** 完全由 Minion 产生（无人编写代码，只有 human review）

**核心问题：** 如何让一个 LLM Agent 在如此大规模、如此高风险的代码库中做到"无人值守"还能保持可控？

#### 5.1.1 环境隔离（Devbox）— 物理可控

Stripe 不在开发者笔记本上跑 Minion，而是为每个任务分配一个独立的云端开发环境——**Devbox**（AWS EC2）。

**物理设计：**
- "Cattle, not pets"：标准化、可替换，逻辑隔离
- 预热池：10 秒内可启动"热" Devbox，包含完整的代码库镜像、Bazel 缓存、类型检查服务
- 隔离网络：Devbox 运行在 QA 环境，无法访问生产服务和真实用户数据
- 每个任务独占一个 Devbox，多个 Minion 并行运行不互相干扰

**Harness 意义：** 如果 Agent 的行动空间被物理隔离在独立容器中，犯错的影响范围就被封死。这不是靠"告诉 Agent 不要做某事"，而是靠"让 Agent 根本接触不到"。

#### 5.1.2 Blueprint 架构 — 编排可控

Minions 的核心编排机制是 **Blueprint**（蓝图），这是 Stripe 自行设计的一种混合编排原语。

**设计思想：**
- **工作流（Workflow）**：通过固定图的步骤执行，每步狭义定义
- **Agent（Loop）**："工具+判断"的循环，LLM 自行决定下一步
- **Blueprint**：两者混合——节点可以是确定性代码，也可以是 Agent Loop

**具体例子（一个 Minion 任务的 Blueprint 大致结构）：**

```
[Start]
  ↓
[确定性节点：环境检查] → 运行诊断脚本，确认代码库状态
  ↓
[Agent 节点：理解任务] → 读取 Slack 上下文，理解需求
  ↓
[确定性节点：规则注入] → 根据当前目录加载 Cursor Rules（条件规则文件）
  ↓
[Agent 节点：实现任务] → 编写代码（在这个节点里 LLM 有完全的创造力）
  ↓
[确定性节点：运行 Linter] → 自动运行所有适用的 linter（确定性代码，不调用 LLM）
  ↓
[Agent 节点：修复 Lint 错误] → 如果 lint 失败，让 Agent 修复
  ↓
[确定性节点：推送分支] → git push（确定性操作）
  ↓
[确定性节点：触发 CI] → 运行选中的测试套件
  ↓
[条件分支]
  ├─ CI 通过 → [Agent 节点：创建 PR]
  └─ CI 失败（有 autofix）→ [应用 autofix] → 重新运行 CI（第二次机会）
  └─ CI 失败（无 autofix）→ [发回 Agent 节点修复] → 最多再运行一次 CI
  ↓
[End: PR 准备好 human review]
```

**关键控制点：**
1. **确定性节点穿插在 Agent 节点之间**：例如"运行 Linter"是一个确定性节点，永远不会跳过或被 LLM "忽略"。这是"让正确的事情不可避免"的工程实践。
2. **限制 CI 循环次数**：最多 2 次 CI 运行。超过后强制进入 human review。这解决了"Agent 可能在无效循环中浪费大量资源"的问题。
3. **条件分支处理失败**：autofix 自动化处理，没有 autofix 才让人工介入。

#### 5.1.3 上下文工程 — 信息可控

**规则文件（Rule Files）：**
- 使用 **Cursor Rules 格式**（经过标准化，支持目录级条件规则）
- 几乎所有规则都是**条件性**的——根据当前子目录自动附加，不做全局规则
- 这样做是因为：Stripe 代码库太大，全局规则会填满整个上下文窗口
- 三种最常用的 coding agent（Minion、Cursor、Claude Code）共用同一套规则

**MCP（Model Context Protocol）动态上下文收集：**
- Minion 在运行前，会**预先抓取**Slack 线程中所有链接的内容
- 通过内部 MCP 服务器 **Toolshed**（~500 个 MCP 工具）获取：内部文档、工单详情、构建状态、代码智能搜索
- Toolshed 是 Stripe 所有 Agent 的共享能力层，添加一个工具立即对所有 Agent 可用

#### 5.1.4 反馈循环设计 — 迭代可控

**"左移反馈"原则（Shift Feedback Left）：**

```
开发时间线 ──────────────────────────────────────────────────→

[Agent 编写代码]
      ↓
[预推送钩子：本地 Linter（<5秒）]  ← 最快反馈
      ↓
[Agent 迭代修复]
      ↓
[CI 测试（选择性运行 300 万测试中的相关子集）]
      ↓
[Autofix 自动应用]
      ↓
[最多再运行一次 CI]
      ↓
[Human Review]
```

**具体做法：**
- 后台守护进程预计算 lint 规则启发式，缓存结果，使预推送钩子通常**小于 1 秒**
- 3 百万测试不能全在本地跑，但可以在 CI 选择性运行相关子集
- 所有失败都有 autofix 机会，Agent 先尝试自动修复，再进入人工干预

**Harness 设计的核心洞察：** 不是"等 Agent 犯错再纠正"，而是"让正确的事情在错误发生之前就被强制执行"。

#### 5.1.5 小结：Stripe Minions 中的四大 Harness 控制维度

| 维度 | 具体实现 |
|------|----------|
| **Reliability（可靠性）** | Blueprint 混合编排、CI 最多 2 次、autofix 自动化 |
| **Performance（性能）** | Devbox 预热 10 秒、并行 Minion 运行、缓存 lint 启发式 |
| **Observability（可观测性）** | Web UI 显示所有决策和动作、Slack 线程作为日志 |
| **Security（安全）** | Devbox 隔离在 QA 环境、MCP 工具安全控制框架、无生产数据访问 |

---

### 5.2 场景二：ByteDance DeerFlow — "执行优先"的 SuperAgent Harness

**背景约束：**
- 开源项目，2026 年 2 月发布，30 天内 **39,400+ GitHub stars**
- 不是实时编码助手，而是**自主执行长时任务**（分钟到小时级）
- 需要协调多个子 Agent、多种工具、多种沙盒执行环境

**核心问题：** 如何让一个复杂的 Multi-Agent 系统在无人监督的情况下可靠完成多步骤任务？

#### 5.2.1 三角色子 Agent 架构 — 职责可控

DeerFlow 将复杂任务分解为三个专业化子 Agent，每个有独立作用域的上下文：

```
[用户请求]
      ↓
┌──────────────────────────────────────┐
│         Task Decomposer             │
│      （任务分解 + 执行规划）              │
└──────────────────────────────────────┘
      ↓              ↓              ↓
┌──────────┐  ┌──────────┐  ┌──────────┐
│Researcher│  │  Coder   │  │ Reporter │
│  Agent   │  │  Agent   │  │  Agent   │
└──────────┘  └──────────┘  └──────────┘
      ↓              ↓              ↓
┌──────────────────────────────────────┐
│         Result Aggregator            │
│      （结果汇总 + 验证）               │
└──────────────────────────────────────┘
      ↓
┌──────────────────────────────────────┐
│         Memory & Output              │
│      （记忆存储 + 结果交付）             │
└──────────────────────────────────────┘
```

**每个 Agent 的上下文设计：**
- **Researcher**：获得用户查询、搜索结果、相关记忆、技能指令。不写代码，不生成最终输出。
- **Coder**：获得分解器输出的结构化需求、相关代码片段、沙盒配置、执行反馈。专注实现，不关心报告格式。
- **Reporter**：获得 Researcher 的研究发现和 Coder 的输出成果，加上格式偏好和模板指令。专注最终交付。

**Harness 意义：** 这种"职责隔离"本身就是一种控制——不让 Researcher 去操心代码实现，不让 Coder 去操心如何汇报。上下文干净，Agent 就不会因为"想太多"而出错。

#### 5.2.2 三层沙盒执行 — 行动可控

DeerFlow 提供三层沙盒，越外层越安全：

| 沙盒模式 | 适用场景 | 隔离级别 |
|----------|----------|----------|
| **Local Process** | 信任的低风险操作 | 最低（直接宿主机执行）|
| **Docker Container** | 推荐的标准模式 | OS 级隔离（seccomp + cgroups）|
| **Kubernetes Pod** | 团队/生产部署 | 资源配额 + 网络策略 |

**Docker 配置示例：**

```yaml
sandbox:
  mode: docker
  image: python:3.12-slim
  timeout: 300          # 5 分钟超时
  memory_limit: 2g       # 内存上限
  network: restricted   # 禁止公网访问
```

**安全模型的核心逻辑：**
- 即使 AI 生成的脚本"失控"，也被封禁在容器内
- seccomp 配置文件限制系统调用
- cgroups 限制资源使用
- network: restricted 禁止挖矿、外发垃圾邮件等恶意行为

#### 5.2.3 Skills 系统 — 能力可控

DeerFlow 的 Skills 不是代码，而是 **Markdown 格式的能力模块**，定义工作流、最佳实践和支持资源：

```markdown
# Skill: WebApp Development

## Purpose
构建并执行完整的 Web 应用程序

## Required Inputs
- 前端框架选择
- 后端技术栈
- 功能需求描述

## Execution Steps
1. Researcher 检查当前最佳实践
2. Coder 脚手架全栈应用
3. 在 Docker 中执行并测试
4. Reporter 生成 API 文档和部署说明

## Expected Outputs
- 可运行的 Web 应用
- 部署文档
```

**这种设计的控制意义：**
- Skills 定义了"Agent 能做什么"的范围
- 用户可以选择性地启用/禁用某些 Skill
- 内置 Skills：research、report、slides、webapp、datapipeline、image、video
- 自定义 Skills 可以组合成复合工作流

#### 5.2.4 上下文压缩 — 窗口可控

对于长时任务，DeerFlow 会在子任务完成后**主动压缩上下文**：

- 已完成的子任务结果被总结并存入文件系统
- 不再立即相关的信息被存入 Memory
- 当前子 Agent 只看到自己的作用域上下文
- 主协调器维护高层状态，不需要文件级细节

#### 5.2.5 小结：DeerFlow 中的四大 Harness 控制维度

| 维度 | 具体实现 |
|------|----------|
| **Reliability（可靠性）** | 子 Agent 职责隔离、结果验证、容错恢复 |
| **Performance（性能）** | 多模型混合（DeepSeek 研究、Doubao 编码）、并行子 Agent |
| **Observability（可观测性）** | 每个子 Agent 独立日志、任务分解可视化 |
| **Security（安全）** | Docker seccomp/cgroups、network: restricted、超时和内存限制 |

---

### 5.3 场景三：OpenAI Codex CLI — "无状态"的生产级 Agent Loop

**背景约束：**
- 开源自 2025 年 4 月（github.com/openai/codex）
- 需要处理可能涉及**数百次模型-工具交互**的多轮对话
- 企业客户要求 **Zero Data Retention（ZDR）**——模型推理后不能存储任何数据
- 必须跨云（ChatGPT.com backend API、api.openai.com、Azure）和本地（ollama）运行

**核心问题：** 如何在保持"无状态"合规的同时，还能高效处理超长对话？

#### 5.3.1 Agent Loop 架构 — 迭代可控

Codex CLI 的核心是一个精心设计的 Agent Loop：

```
[用户输入]
      ↓
┌─────────────────────────────────────┐
│         构建 Prompt                   │
│  system + developer + tools + input  │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│      HTTP POST → 模型推理             │
│  （Responses API，stateless）        │
└─────────────────────────────────────┘
      ↓
[模型响应]
      ↓
┌─────────────────┐   ┌─────────────────┐
│ 返回文本响应      │   │ 请求工具调用     │
│ （结束此 turn）   │   │ （继续循环）      │
└─────────────────┘   └─────────────────┘
                          ↓
                   [执行工具]
                          ↓
                   [追加到 prompt]
                          ↓
                   [重新推理] ← 循环
```

**关键设计决策——无状态请求：**
- **不用** `previous_response_id`：这需要服务器存储对话状态，与 ZDR 冲突
- **每次请求都发送完整对话历史**：看似低效，但保证了每个请求都是自包含的
- 代价是请求体积二次增长——但通过 **Prompt Caching** 解决这个问题（见下）

#### 5.3.2 Prompt Caching — 性能可控

**为什么能线性而非二次增长？**

OpenAI Responses API 支持 **Prompt Caching**——只要请求的前缀完全相同，就可以复用之前推理的计算结果：

```
请求 1: [前缀: 系统指令 + 工具定义 + 示例] + [本次: 新用户消息]
         ↓ 缓存命中（前缀重用）

请求 2: [前缀: 系统指令 + 工具定义 + 示例] + [本次: 新用户消息]
         ↓ 缓存命中（前缀重用）

请求 N: [前缀: 系统指令 + 工具定义 + 示例] + [本次: 新用户消息]
         ↓ 缓存命中（前缀重用）

（如果中间某次前缀变了，比如工具列表顺序改变 → 缓存未命中 → 重新计算）
```

**Codex 的具体优化：**
- 静态内容（instructions、examples）放在前缀开头
- 动态内容（用户消息、工具结果）放在末尾
- 图像和工具定义在请求间**保持完全一致**以确保缓存命中
- MCP 工具尤其容易出问题——如果 MCP 服务器中途改变工具列表，会导致**昂贵的缓存未命中**

#### 5.3.3 上下文窗口自动压缩

当对话长度超过阈值（`auto_compact_limit`），Codex 自动调用 `/responses/compact` 端点：

```
原始输入（很长）
      ↓
/responses/compact 端点
      ↓
压缩为：
  - 一个 summary（摘要）
  - 一个 type=compaction 的加密 blob（保存模型对对话的"潜在理解"）
      ↓
替换原始输入，窗口空间被释放
```

**这解决了什么问题：**
- 对话进行到后期时，prompt 可能包含数百次工具调用结果
- 全部发送给模型会很快撑爆上下文窗口
- 但不能简单截断（会丢失关键状态）
- compaction blob 加密保存了模型继续工作所需的潜在状态

#### 5.3.4 分层配置管理 — 策略可控

Codex 使用**级联式配置**，允许越来越具体的指令覆盖通用指令：

```
优先级低 ← [~/.codex/config.toml 全局配置]
         [项目根目录 AGENTS.md]
         [子目录 AGENTS.md]
         [当前目录 AGENTS.override.md] → 优先级高
```

这意味着：
- 团队可以有全局代码规范（全局 AGENTS.md）
- 特定模块可以有额外的特殊规则（子目录 AGENTS.md）
- 个人可以有本地覆盖（AGENTS.override.md）

#### 5.3.5 小结：Codex CLI 中的四大 Harness 控制维度

| 维度 | 具体实现 |
|------|----------|
| **Reliability（可靠性）** | 无状态请求（每次都完整历史）、自动 compaction |
| **Performance（性能）** | Prompt Caching（线性而非二次）、精确前缀管理 |
| **Observability（可观测性）** | SSE 流式事件、UI 显示推理过程 |
| **Security（安全）** | ZDR 合规（无状态）、只 Codex 提供的工具有沙盒、MCP 工具自行负责 guardrails |

---

### 5.4 横向对比：三个案例的 Harness 设计选择

| 维度 | Stripe Minions | ByteDance DeerFlow | OpenAI Codex |
|------|----------------|---------------------|--------------|
| **场景定位** | 企业内部编码 Agent | 开源超级 Agent | CLI 编程工具 |
| **Agent 架构** | 单一大 Agent + Blueprint | 多角色子 Agent（Researcher/Coder/Reporter）| 单循环 Agent |
| **隔离方式** | 云端 Devbox（EC2）| Docker 容器 / K8s Pod | 本地进程（用户机器）|
| **上下文策略** | 条件性 Rule Files + MCP 预抓取 | 子 Agent 作用域隔离 + 记忆系统 | Prompt Caching + 自动 compaction |
| **反馈机制** | 预推送 Linter → CI → Autofix | 沙盒执行反馈 + 结果验证 | 工具执行结果 → 循环 |
| **失败处理** | 最多 2 次 CI，自动转 human review | 超时 + 内存限制，任务级重试 | 无状态重试，compaction 处理溢出 |
| **并行能力** | 多 Minion 并行，每个独占 Devbox | 多子 Agent 并行（可能）| 多工作目录并行（worktrees）|
| **安全模型** | Devbox 隔离 + MCP 工具权限 | seccomp + cgroups + 网络限制 | 沙盒工具（Codex 提供）+ MCP 自管 |

---

## 六、关键工程问题

> 以下内容在具象化案例中的体现，参见第五章。

## 六、关键工程问题

> 以下内容在三个真实案例（Stripe Minions、ByteDance DeerFlow、OpenAI Codex）中的体现，参见第五章。

### 6.1 权限模型（Permission Model）

Agent 的权限控制是 IMPACT 中 Authority 的具体体现：

| 工具 | 权限模式 |
|------|----------|
| **Claude Code** | 可配置级别，从完全批准到"危险跳过权限"，hooks 启用自定义自动化 |
| **Cursor** | 沙盒 aware harness，显式文件系统路径、网络访问控制 |
| **Cline** | 每个文件变更需批准，安全但较慢 |
| **Devin** | 完全沙盒云环境，Agent 可在沙盒内做任何事，没有明确 PR 不会泄露 |

**关键权衡：** "stutter-step agents get old fast"——每个操作都要求批准会让开发者沮丧并扼杀 flow state，但跳过所有批准可能造成非预期修改。最好的 Harness 提供细粒度控制。

### 5.2 Apply 层（The Apply Layer）

每个 Harness 都面临同一个最终瓶颈：将编辑应用到文件。LLM 生成编辑意图，但将其合并到现有代码时会出问题：

- Diff 在上下文移位时失败
- 搜索替换在代码移动时错过
- 全文件重写浪费 token 并有覆盖并发变更的风险

**解决方案：**
- Cursor 训练了专门用于此步骤的定制模型
- Manus 优化 KV-cache 效率以使重复应用快速
- **Morph Fast Apply**：专门为此构建，每秒 10,500+ tokens，OpenAI 兼容 API

### 5.3 Keep Quality Left

CI/CD 的成熟经验：测试、检和人工审查的分布应基于成本、速度和关键性。当发现 Bug 越早，修复成本越低。

**反馈传感器的生命周期分布：**
- 提交前（甚至集成前）：Linter、快速测试套件、基本代码审查 Agent
- 集成后流水线：突变测试、更全面的代码审查

### 5.4 Harness 可构建性（Harnessability）

不是所有代码库都同样适合构建 Harness：

- 强类型语言天然有类型检查作为传感器
- 清晰的模块边界有利于架构约束规则
- Spring 等框架抽象了 Agent 甚至不需要担心的细节

**Greenfield vs Legacy：**
- Greenfield 团队可以从第一天就 Bake in 可构建性
- Legacy 团队（尤其有大量技术债务）面临更难的问题：Harness 最需要的地方，也是最难构建的地方

---

## 六、学习资源汇总

### 6.1 核心必读文章

| 文章 | 作者 | 链接 |
|------|------|------|
| Harness engineering for coding agent users | Martin Fowler & Birgitta Böckeler | https://martinfowler.com/articles/harness-engineering.html |
| Agent Engineering: How the Harness Became the Product | MorphLLM | https://www.morphllm.com/agent-engineering |
| Harness Engineering: Leveraging Codex in an Agent-First World | OpenAI | https://openai.com/index/harness-engineering/ |
| Agent Harness Engineering | Addy Osmani | https://addyosmani.com/blog/agent-harness-engineering/ |
| Effective Harnesses for Long-Running Agents | Anthropic | https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents |
| Agentic Engineering Patterns | Simon Willison | https://simonwillison.net/2026/Feb/23/agentic-engineering-patterns/ |
| AI Harness Engineering: A Runtime Substrate for Foundation-Model Software Agents | arXiv | https://arxiv.org/html/2605.13357 |
| Agent Engineering (IMPACT Framework) | swyx | https://www.latent.space/p/agent |

### 6.2 框架文档

| 框架 | 文档 |
|------|------|
| LangChain / LangGraph | https://www.langchain.com |
| CrewAI | https://www.crewai.com |
| Mastra | https://mastra.ai |
| LlamaIndex | https://www.llamaindex.ai |
| AutoGen (Microsoft) | https://microsoft.github.io/autogen/ |
| Semantic Kernel | https://learn.microsoft.com/semantic-kernel/ |
| Google ADK | https://google.github.io/adk-docs/ |
| OpenAI Agents SDK | https://developers.openai.com/docs/guides/agents |

### 6.3 工具深度对比

| 资源 | 说明 |
|------|------|
| All Agent Harnesses: The Live Comparison | https://htek.dev/articles/all-agent-harnesses-live-comparison — 全面横比，覆盖所有主要 Agent 平台（持续更新） |
| Agent Frameworks Compared | https://rywalker.com/research/agent-frameworks — 6 大框架深度对比（LangChain, CrewAI, AutoGen, LlamaIndex, Mastra, Vercel AI SDK） |
| How to think about agent frameworks | https://www.langchain.com/blog/how-to-think-about-agent-frameworks — LangChain CEO Harrison Chase 的深度分析 |
| Agent Harness 知识图谱 | https://agent-harness.ai — 883 个实体和 1590 个关系的可交互地图 |

### 6.4 厂商案例

| 公司 | 案例 |
|------|------|
| OpenAI | 层级架构强制执行自定义 Linter 和结构测试，循环"垃圾回收"扫描漂移并让 Agent 建议修复 |
| Stripe | Pre-push hooks 基于启发式运行相关 Linter，"blueprints"展示如何集成反馈传感器到 Agent 工作流 |
| Manus | 六个月内重建了五个 Harness，每次都提高可靠性 |
| Cursor | 训练 Composer 模型处理工具使用轨迹，dropping reasoning traces 导致 30% 性能下降 |
