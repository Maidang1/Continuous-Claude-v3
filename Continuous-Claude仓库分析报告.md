# Continuous Claude 仓库分析报告

> 一个基于 Claude Code 构建的持久化、学习型、多智能体开发环境

---

## 目录

- [一、系统概述](#一系统概述)
- [二、核心设计理念](#二核心设计理念)
- [三、核心架构](#三核心架构)
- [四、主要组件详解](#四主要组件详解)
- [五、工作流程](#五工作流程)
- [六、快速上手](#六快速上手)
- [七、使用方法](#七使用方法)
- [八、技术栈](#八技术栈)

---

## 一、系统概述

### 1.1 什么是 Continuous Claude？

**Continuous Claude** 是一个将 Claude Code 转变为持续学习系统的开发环境。它通过智能代码分析、上下文管理和专业化智能体协调，解决了 AI 辅助编程中的核心问题：**上下文丢失**。

### 1.2 核心问题与解决方案

| 问题 | Continuous Claude 的解决方案 |
|------|---------------------------|
| 上下文压缩导致信息丢失 | YAML 格式的交接文档，更节省 token |
| 每次会话重新开始 | 记忆系统 + 自动学习提取守护进程 |
| 读取完整文件浪费 token | 5 层代码分析 + 语义索引（节省 95% token） |
| 复杂任务需要协调 | 元技能编排智能体工作流 |
| 手动重复工作流程 | 109 个技能 + 自然语言触发 |

### 1.3 核心数据

- **技能（Skills）**: 109 个模块化能力
- **智能体（Agents）**: 32 个专业化子助手
- **钩子（Hooks）**: 30 个生命周期拦截器
- **代码节省**: 95% 的 token 节省（TLDR 分析）

---

## 二、核心设计理念

### 2.1 五要素架构

一个智能体由五个要素组成：**提示词 + 工具 + 上下文 + 记忆 + 模型**

| 组件 | Continuous Claude 的优化策略 |
|------|----------------------------|
| **提示词** | 技能注入相关上下文；钩子添加系统提醒 |
| **工具** | TLDR 减少 token；智能体并行工作 |
| **上下文** | 不仅是提供什么，更是如何提供 |
| **记忆** | 守护进程提取学习成果；召回时浮现相关内容 |
| **模型** | 当其他四个稳固时，模型可以随意切换 |

### 2.2 核心原则

#### "复利，而非压缩"（Compound, don't compact）

- 自动提取学习成果
- 在完整上下文中重新开始
- 每次会话都让系统变得更智能

#### 反复杂性（Anti-Complexity）

- **投入时间，而非金钱** — 无需付费服务（Perplexity 和 NIA 是可选的）
- **学习，而非积累** — 学习型系统比插件堆砌更好地处理边缘情况
- **左移验证** — 钩子在编辑后运行 pyright/ruff，在测试前捕获错误

### 2.3 技能激活系统

**无需记忆斜杠命令**，只需自然描述你的需求。

```
用户："修复 auth.py 中的登录错误"

🎯 技能激活检查
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️ 关键技能（必需）：
  → create_handoff

📚 推荐技能：
  → fix
  → debug

🤖 推荐智能体（节省 token）：
  → debug-agent
  → scout

操作：响应前使用技能工具
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 2.4 优先级系统

| 级别 | 含义 |
|------|------|
| ⚠️ **关键** | 必须使用（例如：会话结束前的交接） |
| 📚 **推荐** | 应该使用（例如：工作流技能） |
| 💡 **建议** | 考虑使用（例如：优化工具） |
| 📌 **可选** | 锦上添花（例如：文档助手） |

---

## 三、核心架构

### 3.1 系统架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CONTINUOUS CLAUDE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│  │   技能层    │    │   智能体    │    │    钩子     │             │
│  │   (109)     │───▶│    (32)     │◀───│    (30)     │             │
│  └─────────────┘    └─────────────┘    └─────────────┘             │
│         │                  │                  │                     │
│         ▼                  ▼                  ▼                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   TLDR 代码分析（5层）                       │   │
│  │   L1:AST → L2:调用图 → L3:控制流 → L4:数据流 → L5:切片     │   │
│  │                    (节省 95% token)                          │   │
│  └─────────────────────────────────────────────────────────────┘   │
│         │                  │                  │                     │
│         ▼                  ▼                  ▼                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│  │  记忆系统   │    │  连续性账本 │    │  协调层     │             │
│  │ PostgreSQL  │    │  交接文档   │    │   多智能体  │             │
│  └─────────────┘    └─────────────┘    └─────────────┘             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 数据流：会话生命周期

```
会话开始                      工作中                        会话结束
    │                           │                             │
    ▼                           ▼                             ▼
┌─────────┐                ┌─────────┐                   ┌─────────┐
│  加载   │                │  跟踪   │                   │  保存   │
│  上下文 │───────────────▶│  变更   │──────────────────▶│  状态   │
└─────────┘                └─────────┘                   └─────────┘
    │                           │                             │
    ├── 连续性账本               ├── 文件声明                  ├── 交接
    ├── 记忆召回                 ├── TLDR 索引                 ├── 学习成果
    └── 符号索引                 └── 白板                      └── 结果
                                    │
                                    ▼
                               ┌─────────┐
                               │ /clear  │
                               │ 清空    │
                               │ 上下文  │
                               └─────────┘
```

### 3.3 连续性循环（详细）

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            连续性循环                                        │
└─────────────────────────────────────────────────────────────────────────────┘

  1. 会话开始                        2. 工作中
  ┌────────────────────┐            ┌────────────────────┐
  │                    │            │                    │
  │  加载账本 ─────────┼──▶ 上下文  │  工具使用后 ───────┼──▶ 索引交接
  │  加载交接          │            │  用户提示 ─────────┼──▶ 技能提示
  │  召回记忆          │            │  编辑跟踪 ─────────┼──▶ 脏标志++
  │  预热 TLDR 缓存    │            │  子智能体停止 ─────┼──▶ 智能体报告
  │                    │            │                    │
  └────────────────────┘            └────────────────────┘
           │                                 │
           │                                 ▼
           │                        ┌────────────────────┐
           │                        │ 3. 压缩前          │
           │                        │                    │
           │                        │  自动交接 ─────────┼──▶ thoughts/shared/
           │                        │  (YAML 格式)       │    handoffs/*.yaml
           │                        │  脏 > 20? ─────────┼──▶ TLDR 重新索引
           │                        │                    │
           │                        └────────────────────┘
           │                                 │
           │                                 ▼
           │                        ┌────────────────────┐
           │                        │ 4. 会话结束        │
           │                        │                    │
           │                        │  心跳停止 ─────────┼──▶ 守护进程唤醒
           │                        │  守护进程启动 ─────┼──▶ 无头 Claude
           │                        │  思考块 ───────────┼──▶ archival_memory
           │                        │                    │
           │                        └────────────────────┘
           │                                 │
           │                                 │
           └──────────────◀────── /clear ◀──┘
                          清空上下文 + 保留状态
```

### 3.4 数据层架构

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           数据层架构                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  TLDR 5层代码分析              语义索引                                      │
│  ┌────────────────────────┐   ┌────────────────────────┐                   │
│  │ L1: AST (~500 tok)     │   │ BGE-large-en-v1.5      │                   │
│  │     └── 函数、类       │   │ ├── 所有 5 层          │                   │
│  │         签名           │   │ ├── 10 行上下文        │                   │
│  │                        │   │ └── FAISS 索引         │                   │
│  │ L2: 调用图 (+440)      │   │                        │                   │
│  │     └── 跨文件         │───│ 查询："认证逻辑"       │                   │
│  │         依赖关系       │   │ 返回：排序的函数       │                   │
│  │                        │   └────────────────────────┘                   │
│  │ L3: 控制流图 (+110)    │                                                │
│  │     └── 控制流         │                                                │
│  │                        │   记忆 (PostgreSQL+pgvector)                   │
│  │ L4: 数据流图 (+130)    │   ┌────────────────────────┐                   │
│  │     └── 数据流         │   │ sessions (心跳)        │                   │
│  │                        │   │ file_claims (锁)       │                   │
│  │ L5: 程序依赖图 (+150)  │   │ archival_memory (BGE)  │                   │
│  │     └── 切片           │   │ handoffs (嵌入向量)    │                   │
│  └────────────────────────┘   └────────────────────────┘                   │
│         ~1,200 tokens                                                       │
│         vs 23,000 原始                                                      │
│         = 95% 节省             文件系统                                     │
│                                ┌────────────────────────┐                   │
│                                │ thoughts/              │                   │
│                                │ ├── ledgers/           │                   │
│                                │ │   └── CONTINUITY_*.md│                   │
│                                │ └── shared/            │                   │
│                                │     ├── handoffs/*.yaml│                   │
│                                │     └── plans/*.md     │                   │
│                                │                        │                   │
│                                │ .tldr/                 │                   │
│                                │ └── (守护进程缓存)     │                   │
│                                └────────────────────────┘                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 四、主要组件详解

### 4.1 技能系统（Skills System）

技能是由自然语言触发的模块化能力。位于 `.claude/skills/`。

#### 元技能（工作流编排器）

| 元技能 | 执行链 | 使用场景 |
|--------|--------|----------|
| `/workflow` | 路由器 → 适当的工作流 | 不知道从哪里开始 |
| `/build` | 发现 → 计划 → 验证 → 实现 → 提交 | 构建功能 |
| `/fix` | 侦查 → 事前分析 → 修复 → 测试 → 提交 | 修复 bug |
| `/tdd` | 计划 → 测试 → 实现 → 验证 | 测试驱动开发 |
| `/refactor` | 分析 → 计划 → 实现 → 审查 → 测试 | 安全的代码重构 |
| `/review` | 并行专业审查 → 综合 | 代码审查 |
| `/explore` | 侦察（快速/深入/架构） | 理解代码库 |
| `/security` | 漏洞扫描 → 验证 | 安全审计 |
| `/release` | 审计 → E2E → 审查 → 变更日志 | 发布版本 |

#### 核心技能（高价值工具）

**规划与风险**
- **premortem**: TIGERS & ELEPHANTS 风险分析 - 任何重要实现前使用
- **discovery-interview**: 将模糊想法转化为详细规格

**上下文管理**
- **create_handoff**: 捕获会话状态以便转移
- **resume_handoff**: 从交接中恢复上下文
- **continuity_ledger**: 在会话内跟踪状态

**代码分析（节省 95% Token）**
- **tldr-code**: 调用图、控制流图、数据流图、切片
- **ast-grep-find**: 结构化代码搜索
- **morph-search**: 快速文本搜索（比 grep 快 20 倍）

**研究**
- **perplexity-search**: AI 驱动的网络搜索
- **nia-docs**: 库文档搜索
- **github-search**: 搜索 GitHub 代码/issue/PR

**质量**
- **qlty-check**: 70+ linters，自动修复
- **braintrust-analyze**: 会话分析、回放和调试失败会话

**数学与形式化证明**
- **math**: 统一计算（SymPy、Z3、Pint）— 所有数学的单一入口
- **prove**: Lean4 定理证明，5 阶段工作流（研究 → 设计 → 测试 → 实现 → 验证）
- **pint-compute**: 单位感知算术和转换
- **shapely-compute**: 计算几何

#### 思维过程

```
我想做什么？
├── 不知道 → /workflow (引导路由器)
├── 构建 → /build greenfield 或 brownfield
├── 修复 → /fix bug
├── 理解 → /explore
├── 规划 → premortem 先，然后 plan-agent
├── 研究 → oracle 或 perplexity-search
├── 审查 → /review
├── 证明 → /prove (Lean4 形式化验证)
├── 计算 → /math (SymPy、Z3、Pint)
└── 发布 → /release
```

### 4.2 智能体系统（Agents System）

智能体是通过 Task 工具生成的专业化 AI 工作者。位于 `.claude/agents/`。

#### 智能体分类（32 个活跃）

**编排者（2）**
- **maestro**: 多智能体协调，模式（Pipeline、Swarm、Jury）
- **kraken**: TDD 实现智能体，支持检查点/恢复

**规划者（4）**
- **architect**: 功能规划 + API 集成
- **phoenix**: 重构 + 框架迁移规划
- **plan-agent**: 轻量级规划，带研究/MCP 工具
- **validate-agent**: 根据最佳实践验证计划

**探索者（4）**
- **scout**: 代码库探索（用它代替 Explore）
- **oracle**: 外部研究（网络、文档、API）
- **pathfinder**: 外部仓库分析
- **research-codebase**: 记录代码库现状

**实现者（3）**
- **kraken**: TDD 实现，严格的测试优先工作流
- **spark**: 轻量级修复和快速调整
- **agentica-agent**: 使用 Agentica SDK 构建 Python 智能体

**调试者（3）**
- **sleuth**: 一般 bug 调查和根本原因
- **debug-agent**: 通过日志/代码搜索进行问题调查
- **profiler**: 性能分析和竞态条件

**验证者（2）** - arbiter、atlas

**审查者（6）** - critic、judge、surveyor、liaison、plan-reviewer、review-agent

**专业化（8）** - aegis、herald、scribe、chronicler、session-analyst、braintrust-analyst、memory-extractor、onboard

#### 常见工作流

| 工作流 | 智能体链 |
|--------|----------|
| 功能开发 | architect → plan-reviewer → kraken → review-agent → arbiter |
| 重构 | phoenix → plan-reviewer → kraken → judge → arbiter |
| Bug 修复 | sleuth → spark/kraken → arbiter → scribe |

### 4.3 钩子系统（Hooks System）

钩子在 Claude Code 生命周期点拦截。位于 `.claude/hooks/`。

#### 钩子事件（30 个钩子）

| 事件 | 关键钩子 | 目的 |
|------|----------|------|
| **SessionStart** | session-start-continuity、session-register、braintrust-tracing | 加载上下文，注册会话 |
| **PreToolUse** | tldr-read-enforcer、smart-search-router、tldr-context-inject、file-claims | 节省 token，搜索路由 |
| **PostToolUse** | post-edit-diagnostics、handoff-index、post-edit-notify | 验证，索引 |
| **PreCompact** | pre-compact-continuity | 压缩前自动保存 |
| **UserPromptSubmit** | skill-activation-prompt、memory-awareness | 技能提示，记忆召回 |
| **SubagentStop** | subagent-stop-continuity | 保存智能体状态 |
| **SessionEnd** | session-end-cleanup、session-outcome | 清理，提取学习成果 |

#### 关键钩子

| 钩子 | 目的 |
|------|------|
| **tldr-context-inject** | 向智能体提示添加代码分析 |
| **smart-search-router** | 在适当时将 grep 路由到 AST-grep |
| **post-edit-diagnostics** | 编辑后运行 pyright/ruff |
| **memory-awareness** | 浮现相关学习成果 |

### 4.4 TLDR 代码分析

TLDR 通过 5 层分析提供节省 token 的代码摘要。

#### 5 层堆栈

| 层 | 名称 | 提供内容 | Token |
|----|------|----------|-------|
| **L1** | AST | 函数、类、签名 | ~500 tokens |
| **L2** | 调用图 | 谁调用什么（跨文件） | +440 tokens |
| **L3** | 控制流图 | 控制流、复杂度 | +110 tokens |
| **L4** | 数据流图 | 数据流、变量跟踪 | +130 tokens |
| **L5** | 程序依赖图 | 程序切片、影响分析 | +150 tokens |

**总计：~1,200 tokens vs 23,000 原始 = 95% 节省**

#### CLI 命令

```bash
# 结构分析
tldr tree src/                      # 文件树
tldr structure src/ --lang python   # 代码结构（代码地图）

# 搜索和提取
tldr search "process_data" src/     # 查找代码
tldr context process_data --project src/ --depth 2  # LLM 就绪上下文

# 流分析
tldr cfg src/main.py main           # 控制流图
tldr dfg src/main.py main           # 数据流图
tldr slice src/main.py main 42      # 什么影响第 42 行？

# 代码库分析
tldr impact process_data src/       # 谁调用这个函数？
tldr dead src/                      # 查找不可达代码
tldr arch src/                      # 检测架构层

# 语义搜索（自然语言）
tldr daemon semantic "find authentication logic"
```

#### 语义索引

超越结构分析，TLDR 构建代码库的**语义索引**：

- **自然语言查询** — 询问"错误处理在哪里？"而不是 grep
- **自动重建** — 脏标志钩子跟踪文件变更；索引在 N 次编辑后重建
- **选择性索引** — 使用 `.tldrignore` 控制索引内容

```bash
# .tldrignore 示例
__pycache__/
*.test.py
node_modules/
.venv/
```

语义索引使用所有 5 层加上 10 行周围代码上下文——不仅仅是文档字符串。

### 4.5 记忆系统

由 PostgreSQL + pgvector 驱动的跨会话学习。

#### 工作原理

```
会话结束 → 数据库检测到停止的心跳（>5 分钟）
         → 守护进程生成无头 Claude (Sonnet)
         → 分析会话中的思考块
         → 提取学习成果到 archival_memory
         → 下次会话召回相关学习成果
```

关键见解：**思考块包含真正的推理**——不仅是 Claude 做了什么，还有为什么。守护进程自动提取这些。

#### 对话界面

| 你说什么 | 发生什么 |
|---------|---------|
| "记住认证使用 JWT" | 存储带上下文的学习成果 |
| "召回认证模式" | 搜索记忆，浮现匹配项 |
| "我们对 X 决定了什么？" | 通过 memory-awareness 钩子隐式召回 |

#### 数据库模式（4 个表）

| 表 | 目的 |
|----|------|
| **sessions** | 跨终端感知 |
| **file_claims** | 跨终端文件锁定 |
| **archival_memory** | 带 BGE 嵌入的长期学习成果 |
| **handoffs** | 带嵌入的会话交接 |

### 4.6 连续性系统

跨上下文清除和会话保留状态。

#### 连续性账本

会话内状态跟踪。位置：`thoughts/ledgers/CONTINUITY_<topic>.md`

```markdown
# 会话: feature-x
更新时间: 2026-01-08

## 目标
使用适当的错误处理实现功能 X

## 已完成
- [x] 设计 API 模式
- [x] 实现核心逻辑

## 进行中
- [ ] 添加错误处理

## 阻塞
- 需要澄清重试策略
```

#### 交接

会话间知识转移。位置：`thoughts/shared/handoffs/<session>/`

```yaml
---
date: 2026-01-08T15:26:01+0000
session_name: feature-x
status: complete
---

# 交接: 功能 X 实现

## 任务
| 任务 | 状态 |
|------|------|
| 设计 API | 已完成 |
| 实现核心 | 已完成 |
| 错误处理 | 待定 |

## 下一步
1. 向 API 调用添加重试逻辑
2. 编写集成测试
```

---

## 五、工作流程

### 5.1 工作流链

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           元技能工作流                                       │
└─────────────────────────────────────────────────────────────────────────────┘

  /fix bug                              /build greenfield
  ─────────                             ─────────────────
  ┌──────────┐  ┌──────────┐            ┌──────────┐  ┌──────────┐
  │  sleuth  │─▶│ premortem│            │discovery │─▶│plan-agent│
  │ (诊断)   │  │  (风险)  │            │ (澄清)   │  │ (设计)   │
  └──────────┘  └────┬─────┘            └──────────┘  └────┬─────┘
                     │                                      │
                     ▼                                      ▼
              ┌──────────┐                          ┌──────────┐
              │  kraken  │                          │ validate │
              │  (修复)  │                          │ (检查)   │
              └────┬─────┘                          └────┬─────┘
                   │                                      │
                   ▼                                      ▼
              ┌──────────┐                          ┌──────────┐
              │  arbiter │                          │  kraken  │
              │ (测试)   │                          │  (实现)  │
              └────┬─────┘                          └────┬─────┘
                   │                                      │
                   ▼                                      ▼
              ┌──────────┐                          ┌──────────┐
              │  commit  │                          │  commit  │
              └──────────┘                          └──────────┘
```

### 5.2 自然语言示例

| 你说什么 | 激活什么 |
|---------|---------|
| "修复损坏的登录" | `/fix` 工作流 → debug-agent、scout |
| "构建用户仪表板" | `/build` 工作流 → plan-agent、kraken |
| "我想了解这个代码库" | `/explore` + scout 智能体 |
| "这个计划可能出什么问题？" | `/premortem` |
| "帮我弄清楚我需要什么" | `/discovery-interview` |
| "今天完成了" | `create_handoff`（关键） |
| "从上次继续" | `resume_handoff` |
| "研究认证模式" | oracle 智能体 + perplexity |
| "查找此 API 的所有用法" | scout 智能体 + ast-grep |

### 5.3 技能 vs 工作流 vs 智能体

| 类型 | 目的 | 示例 |
|------|------|------|
| **技能** | 单一目的工具 | `commit`、`tldr-code`、`qlty-check` |
| **工作流** | 多步骤流程 | `/fix`（sleuth → premortem → kraken → commit） |
| **智能体** | 专业化子会话 | scout（探索）、oracle（研究） |

---

## 六、快速上手

### 6.1 前置要求

- Python 3.11+
- [uv](https://github.com/astral-sh/uv) 包管理器
- Docker（用于 PostgreSQL）
- Claude Code CLI

### 6.2 安装

```bash
# 克隆
git clone https://github.com/parcadei/Continuous-Claude-v3.git
cd Continuous-Claude-v3/opc

# 运行设置向导（12 个步骤）
uv run python -m scripts.setup.wizard
```

### 6.3 向导做什么

| 步骤 | 做什么 |
|------|--------|
| 1 | 备份现有 .claude/ 配置（如果存在） |
| 2 | 检查前置要求（Docker、Python、uv） |
| 3-5 | 数据库 + API 密钥配置 |
| 6-7 | 启动 Docker 栈，运行迁移 |
| 8 | 安装 Claude Code 集成（32 智能体、109 技能、30 钩子） |
| 9 | 数学功能（SymPy、Z3、Pint - 可选） |
| 10 | TLDR 代码分析工具 |
| 11-12 | 诊断工具 + Loogle（可选） |

### 6.4 第一次会话

```bash
# 启动 Claude Code
claude

# 尝试一个工作流
> /workflow
```

### 6.5 第一次会话命令

| 命令 | 做什么 |
|------|--------|
| `/workflow` | 基于目标的路由（研究/计划/构建/修复） |
| `/fix bug <描述>` | 调查并修复 bug |
| `/build greenfield <功能>` | 从头构建新功能 |
| `/explore` | 理解代码库 |
| `/premortem` | 实现前的风险分析 |

---

## 七、使用方法

### 7.1 配置

#### .claude/settings.json

钩子、工具和工作流的中央配置。

```json
{
  "hooks": {
    "SessionStart": [...],
    "PreToolUse": [...],
    "PostToolUse": [...],
    "UserPromptSubmit": [...]
  }
}
```

#### .claude/skills/skill-rules.json

技能激活触发器。

```json
{
  "rules": [
    {
      "skill": "fix",
      "keywords": ["fix this", "broken", "not working"],
      "intentPatterns": ["fix.*(bug|issue|error)"]
    }
  ]
}
```

### 7.2 环境变量

| 变量 | 目的 | 必需 |
|------|------|------|
| `DATABASE_URL` | PostgreSQL 连接字符串 | 是 |
| `BRAINTRUST_API_KEY` | 会话跟踪 | 否 |
| `PERPLEXITY_API_KEY` | 网络搜索 | 否 |
| `NIA_API_KEY` | 文档搜索 | 否 |

没有 API 密钥的服务仍然可以工作：
- 连续性系统（账本、交接）
- TLDR 代码分析
- 本地 git 操作
- TDD 工作流

### 7.3 目录结构

```
continuous-claude/
├── .claude/
│   ├── agents/           # 32 个专业化智能体
│   ├── hooks/            # 30 个生命周期钩子
│   │   ├── src/          # TypeScript 源代码
│   │   └── dist/         # 编译的 JavaScript
│   ├── skills/           # 109 个模块化能力
│   ├── rules/            # 系统策略
│   ├── scripts/          # Python 工具
│   └── settings.json     # 钩子配置
├── opc/
│   ├── packages/
│   │   └── tldr-code/    # 5 层代码分析
│   ├── scripts/
│   │   ├── setup/        # 向导、Docker、集成
│   │   └── core/         # recall_learnings、store_learning
│   └── docker/
│       └── init-schema.sql  # 4 表 PostgreSQL 模式
├── thoughts/
│   ├── ledgers/          # 连续性账本（CONTINUITY_*.md）
│   └── shared/
│       ├── handoffs/     # 会话交接（*.yaml）
│       └── plans/        # 实现计划
└── docs/                 # 文档
```

### 7.4 更新

拉取最新变更并同步安装：

```bash
cd continuous-claude/opc
uv run python -m scripts.setup.update
```

这将：
- 从 GitHub 拉取最新
- 更新钩子、技能、规则、智能体
- 如果已安装则升级 TLDR
- 如果变更则重建 TypeScript 钩子

### 7.5 针对现有项目

安装后，启动 Claude 并运行：
```
> /onboard
```

这会分析代码库并创建初始连续性账本。

---

## 八、技术栈

### 8.1 核心技术

| 组件 | 技术 |
|------|------|
| **语言** | Python 3.11+、TypeScript |
| **AI 模型** | Claude 3.5 Sonnet、Claude 3 Opus |
| **数据库** | PostgreSQL + pgvector |
| **向量嵌入** | BGE-large-en-v1.5（1024 维） |
| **代码解析** | tree-sitter |
| **包管理** | uv（Python）、npm（TypeScript） |
| **容器化** | Docker、Docker Compose |

### 8.2 依赖的工具和服务

**核心工具**
- **[uv](https://github.com/astral-sh/uv)** - Python 打包
- **[tree-sitter](https://tree-sitter.github.io/)** - 代码解析
- **[Braintrust](https://braintrust.dev)** - LLM 评估、日志和会话跟踪
- **[qlty](https://github.com/qltysh/qlty)** - 通用代码质量 CLI（70+ linters）
- **[ast-grep](https://github.com/ast-grep/ast-grep)** - 基于 AST 的代码搜索和重构

**可选服务**
- **[Nia](https://trynia.ai)** - 库文档搜索
- **[Morph](https://www.morphllm.com)** - WarpGrep 快速代码搜索
- **[Firecrawl](https://www.firecrawl.dev)** - 网络爬取 API
- **[RepoPrompt](https://repoprompt.com)** - 节省 token 的代码库地图

### 8.3 数学和形式化验证

| 工具 | 目的 | 示例 |
|------|------|------|
| **SymPy** | 符号数学 | 求解方程、积分、矩阵运算 |
| **Z3** | 约束求解 | 证明不等式、SAT 问题 |
| **Pint** | 单位转换 | 英里转公里、量纲分析 |
| **Lean4** | 形式化证明 | 机器验证定理 |
| **Mathlib** | 10 万+ 定理 | 预形式化引理以构建 |
| **Loogle** | 类型感知搜索 | 按签名查找 Mathlib 引理 |

### 8.4 数据库模式

**PostgreSQL 表（4 个）**

```sql
-- 会话跟踪
CREATE TABLE sessions (
    session_id TEXT PRIMARY KEY,
    heartbeat TIMESTAMPTZ,
    terminal_id TEXT,
    status TEXT
);

-- 文件锁定
CREATE TABLE file_claims (
    file_path TEXT PRIMARY KEY,
    session_id TEXT,
    claimed_at TIMESTAMPTZ
);

-- 长期记忆（带向量嵌入）
CREATE TABLE archival_memory (
    id SERIAL PRIMARY KEY,
    session_id TEXT,
    content TEXT,
    embedding vector(1024),  -- BGE-large-en-v1.5
    created_at TIMESTAMPTZ,
    confidence TEXT,
    type TEXT
);

-- 会话交接
CREATE TABLE handoffs (
    id SERIAL PRIMARY KEY,
    session_name TEXT,
    content TEXT,
    embedding vector(1024),
    created_at TIMESTAMPTZ
);
```

### 8.5 文件格式

**连续性账本（Markdown）**
```markdown
# 会话: <名称>
更新时间: <时间戳>

## 目标
...

## 已完成
- [x] 任务 1

## 进行中
- [ ] 任务 2
```

**交接（YAML + Markdown）**
```yaml
---
date: 2026-01-08T15:26:01+0000
session_name: feature-x
status: complete
---

# 交接: <标题>

## 任务
| 任务 | 状态 |
|------|------|
| ... | ... |
```

---

## 九、关键统计数据

| 指标 | 数量 | 备注 |
|------|------|------|
| Python 函数 | 2,328 | 跨所有 `opc/scripts/` |
| TypeScript 钩子 | 34 | 活跃在 `.claude/hooks/src/` |
| 技能 | 109 | 在 `.claude/skills/` 中 |
| 智能体 | 32 | 在系统提示中定义 |
| 测试 | 265+ | 仅 TLDR-code |
| Token 节省 | 95% | TLDR vs 原始文件读取 |
| 循环依赖 | 0 | 通过归档作用域模块解决 |
| 最常调用函数 | get_connection | 38 个调用者 |

---

## 十、成就

### 10.1 Sylvester-Gallai 定理

使用 `/prove` 技能创建了 Sylvester-Gallai 定理的**首个 Lean 形式化**。

### 10.2 Token 效率

- **95% token 节省**：TLDR 5 层分析 vs 读取原始文件
- **1,200 tokens** vs **23,000 tokens**

### 10.3 系统复杂度

- **109 技能**，用自然语言触发
- **32 智能体**，专业化子助手
- **30 钩子**，自动行为
- **5 层代码分析**

---

## 十一、贡献

参见 [CONTRIBUTING.md](CONTRIBUTING.md) 了解以下指南：

- 添加新技能
- 创建智能体
- 开发钩子
- 扩展 TLDR

---

## 十二、许可证

[MIT](LICENSE) - 自由使用，回馈贡献。

---

## 十三、总结

**Continuous Claude** 不仅是一个编码助手——它是一个持久化、学习型、多智能体开发环境，随着每次会话变得更智能。

### 核心优势

1. **持久化记忆**：跨会话学习和上下文保留
2. **Token 效率**：95% 的 token 节省通过智能代码分析
3. **专业化智能体**：32 个专家智能体处理复杂任务
4. **自然交互**：无需记忆命令，只需描述你的目标
5. **自动学习**：守护进程提取洞察并构建知识库

### 适用场景

- ✅ 大型代码库探索和理解
- ✅ 复杂功能开发（需要规划和协调）
- ✅ Bug 调查和修复
- ✅ 代码重构和架构改进
- ✅ 形式化验证和数学证明
- ✅ 跨会话的长期项目

### 开始使用

```bash
git clone https://github.com/parcadei/Continuous-Claude-v3.git
cd Continuous-Claude-v3/opc
uv run python -m scripts.setup.wizard
```

只需 5 分钟，你就能拥有一个智能、持久化、不断学习的 AI 开发伙伴。

---

*本报告基于 Continuous Claude v3 代码库分析生成，涵盖架构、组件、工作流程和使用方法。*

*生成时间：2026-01-12*
