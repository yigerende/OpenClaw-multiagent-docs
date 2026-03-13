[README.md](https://github.com/user-attachments/files/25967012/README.md)
# OpenClaw Multi-Agent 协作指南

> 📚 深入解析 OpenClaw 多 Agent 架构，从理论到实践，构建专业化 AI 团队

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-1.0-green.svg)](https://github.com/openclaw/openclaw)
[![Last Updated](https://img.shields.io/badge/last%20updated-2026--03--13-yellow.svg)]()

---

## 📌 目录

- [为什么单个 Agent 不够用](#-为什么单个-agent-不够用)
- [多 Agent 的核心理念](#-多-agent-的核心理念)
- [架构核心：三层隔离](#-架构核心三层隔离)
- [路由绑定：Bindings 机制](#-路由绑定-bindings-机制)
- [实战配置](#-实战配置从零构建多-agent-团队)
- [Agent 间通信](#-agent-间通信从单干到协作)
- [四种协作模式](#-四种协作模式详解)
- [最佳实践](#-生产环境最佳实践)
- [常见问题](#-常见问题与避坑指南)

---

## 📌 为什么单个 Agent 不够用？

### 单个 Agent 的三大瓶颈

| 问题 | 症状 | 根本原因 |
|------|------|---------|
| **内存膨胀** | Agent 响应越来越慢 | 记忆文件不断累积，每次对话加载大量历史 |
| **上下文污染** | 答案跨领域"串味" | 不同领域知识互相干扰 |
| **成本螺旋** | Token 消耗远超预期 | 每次对话携带所有无关背景材料 |

> 💡 **生动比喻**：单个 Agent 就像一个同时打开 50 个浏览器标签页的工人——看起来什么都做，其实什么都做不好。

---

## 🎯 多 Agent 的核心理念

**让专人做专事**

- ✍️ **写作 Agent** 只关心文字——记忆里是写作技巧和风格模板
- 💻 **编码 Agent** 只关心代码逻辑——工作区里是项目代码和技术文档
- 🔍 **研究 Agent** 只关心信息收集——配备搜索工具和数据整理能力

每个 Agent 都有独立的"大脑"（记忆）、"办公室"（工作区）和"工作日志"（会话记录），互不干扰。

---

## 🏗️ 架构核心：三层隔离

```
~/.openclaw/agents/<agentId>/
├── agent/              # 身份凭证层
│   ├── auth-profiles.json
│   └── models.json
└── sessions/           # 状态层
    └── <session-id>.jsonl

~/.openclaw/workspace-<agentId>/  # 工作区层
├── SOUL.md
├── AGENTS.md
├── USER.md
└── memory/
```

| 层级 | 目录 | 用途 | 类比 |
|------|------|------|------|
| 身份层 | `agents/<id>/agent/` | 决定使用哪个模型和凭证 | 员工工牌 |
| 状态层 | `agents/<id>/sessions/` | 独立聊天历史和路由状态 | 工作日志 |
| 工作区层 | `workspace-<id>/` | 独立文件、提示词和记忆 | 个人办公室 |

**核心优势**：物理级上下文隔离。写作 Agent 永远不会看到编码 Agent 的代码文件，编码 Agent 也不会被写作 Agent 的风格指南干扰。

---

## 🔀 路由绑定：Bindings 机制

**匹配优先级**（从高到低）：

1. 精确对等匹配（DM/群 ID）
2. 父对等匹配（线程继承）
3. guildId + role（Discord 专用）
4. guildId 单独匹配
5. teamId（Slack 专用）
6. accountId 匹配
7. 频道级匹配
8. 默认 Agent 回退

> **简单说**：规则越具体，优先级越高。

---

## 🚀 两种部署方式

| 维度 | 克隆模式（单 Bot 路由） | 独立舰队（多 Bot） |
|------|----------------------|------------------|
| 实现 | 一个飞书 Bot，通过 Bindings 路由 | 每个 Agent 独立 Bot |
| 用户体验 | 同一个 Bot，但"切换大脑" | 每个 Bot 有自己的头像和名称 |
| 管理成本 | 低 | 高 |
| 适用场景 | 个人使用 | 团队协作 |

**推荐**：个人用户使用**克隆模式**，配置最简单。

---

## 📝 实战配置：从零构建多 Agent 团队

### 1. 创建新 Agent

```bash
# 创建写作 Agent
openclaw agents add writer \
 --model deepseek/deepseek-chat \
 --workspace ~/.openclaw/workspace-writer

# 创建头脑风暴 Agent
openclaw agents add brainstorm \
 --model zai/glm-4.7 \
 --workspace ~/.openclaw/workspace-brainstorm

# 创建编码 Agent
openclaw agents add coder \
 --model anthropic/claude-sonnet-4-6 \
 --workspace ~/.openclaw/workspace-coder
```

**模型推荐**：

| Agent | 推荐模型 | 原因 |
|-------|---------|------|
| Brainstorm | glm-4.7 | 中文创意能力强 |
| Writer | deepseek | 性价比高，输出稳定 |
| Coder | claude-sonnet | 编码能力强 |
| Researcher | gpt-4o | 多模态理解好 |

### 2. 赋予 Agent 灵魂：编写"入职材料"

#### SOUL.md - 人格定义

```markdown
# Writer Agent

## Role
你是经验丰富的内容写作专家，擅长用犀利真实的视角剖析科技话题。

## Style
- 开头必须用引人入胜的场景或反直觉的洞察
- 段落短小精悍，适合手机阅读
- 善用类比，让外行也能理解复杂概念
- 结尾必须包含行动号召（CTA）

## Prohibitions
- 禁止使用"众所周知""不言而喻"等空洞表述
- 不要堆砌术语，每个技术术语首次出现都要解释
```

#### AGENTS.md - 行为准则

```markdown
# 工作准则

## 输出格式
- 所有文章使用 Markdown 格式
- 标题层级不超过三级（H2 > H3 > H4）
- 代码块必须指定语言类型

## 工作流程
1. 先写大纲，确认后再展开
2. 每段控制在 150 字以内
3. 完成后自检 SEO 要素
```

#### USER.md - 用户信息

```markdown
# 用户信息

## 身份
博主、科技内容创作者，专注 AI 工具与生产力。

## 偏好
- 语言风格：专业但不学术，有温度
- 目标读者：科技爱好者和对 AI 感兴趣的产品经理
- 发布平台：博客 + 社交媒体
```

### 3. 设置身份标识

```bash
openclaw agents set-identity --agent writer --name "Writer" --emoji "✍️"
openclaw agents set-identity --agent brainstorm --name "Brainstormer" --emoji "💡"
openclaw agents set-identity --agent coder --name "Code Smith" --emoji "⚡"
```

### 4. 绑定消息渠道

#### 飞书绑定配置（openclaw.json）

```json
{
  "bindings": [
    {
      "agentId": "writer",
      "match": {
        "channel": "feishu",
        "peer": {
          "kind": "group",
          "id": "oc_abc123..."
        }
      }
    },
    {
      "agentId": "brainstorm",
      "match": {
        "channel": "feishu",
        "peer": {
          "kind": "group",
          "id": "oc_def456..."
        }
      }
    }
  ]
}
```

#### 免@模式配置

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_xxx",
      "appSecret": "xxx",
      "groups": {
        "oc_abc123...": { "requireMention": false }
      }
    }
  }
}
```

> ⚠️ 需要在飞书管理后台开启 `im:message.group_msg` 权限。

---

## 🤝 Agent 间通信：从单干到协作

### 核心机制：sessions_send

```
用户发送指令 → 主管 Agent（main）接收
           ↓
      判断任务类型
      ↙    ↓    ↘
  brainstorm  writer  coder
      ↓
   收集结果
      ↓
  主管 Agent 返回用户
```

### 配置 Agent 间通信

```json
{
  "tools": {
    "agentToAgent": {
      "enabled": true,
      "allow": ["main", "brainstorm", "writer", "coder"]
    }
  }
}
```

**关键点**：
- ✅ 默认关闭：必须显式启用
- ✅ 白名单机制：明确定义哪些 Agent 可以通信
- ✅ 最小权限原则：只给需要的 Agent 开启

### 设计主管 Agent 角色

```markdown
# Main Agent - 首席协调官

## 核心职责
1. 接收请求：理解用户原始指令
2. 精准 dispatch：判断任务类型，分配给合适的专家 Agent
3. 质量控制：审查专家 Agent 输出，必要时要求修改
4. 端到端编排：确保多步骤任务不掉链子

## Dispatch 规则
- 头脑风暴、创意发散 → @brainstorm
- 文章写作、文案优化 → @writer
- 代码编写、技术实现 → @coder
- 简单问答、闲聊 → 自己处理
```

---

## 📊 四种协作模式详解

### 1. 主管模式（Supervisor）

```
     用户请求
        ↓
   ┌─────────┐
   │Supervisor│
   │(Main Agent)│
   └─┬───┬───┬─┘
     ↓   ↓   ↓
   ┌─┐ ┌─┐ ┌─┐
   │A│ │B│ │C│
   └─┘ └─┘ └─┘
     ↓   ↓   ↓
   ┌─────────┐
   │汇总输出  │
   └─────────┘
```

**适用场景**：
- 需要统一入口的个人助理
- 需要跨领域协作的任务
- 需要质量把关和结果审查

### 2. 路由模式（Router）

```
     用户请求
        ↓
   ┌─────────┐
   │ Router  │
   │(Dispatcher)│
   └─┬───┬───┬─┘
     ↓   ↓   ↓ ← 并行执行
   ┌─┐ ┌─┐ ┌─┐
   │A│ │B│ │C│
   └─┘ └─┘ └─┘
     ↓   ↓   ↓
   ┌─────────┐
   │合成输出  │
   └─────────┘
```

**与主管模式区别**：主管是顺序协调（先 A 后 B），路由是并行分发（A 和 B 同时）。

**适用场景**：
- 不同渠道需要不同风格
- 不同用户群体需要不同专业度
- 需要快速响应，无需跨 Agent 协作

### 3. 流水线模式（Pipeline）

```
   用户请求
      ↓
┌─────────┐ ┌─────────┐ ┌─────────┐
│Researcher│ → │ Writer  │ → │Reviewer │
└─────────┘ └─────────┘ └─────────┘
      ↓
   ┌─────────┐
   │最终输出  │
   └─────────┘
```

**适用场景**：
- 内容创作流水线：研究→草稿→审查→格式化
- 代码开发流水线：设计→编码→测试→部署
- 数据处理流水线：收集→清洗→分析→可视化

**博客生产流水线示例**：
1. `@brainstorm` "围绕 AI 编程生产力头脑风暴 5 个选题"
2. 用户选择方向后 → `@writer` "基于{头脑风暴输出}写文章"
3. 写作完成后 → `@coder` "检查文章中的代码示例是否正确可运行"
4. Main Agent 汇总所有反馈，输出最终版本

### 4. 并行模式（Parallel）

```
     用户请求
        ↓
   ┌──────────┐
   │Task Splitter│
   └─┬───┬───┬─┘
     ↓   ↓   ↓ ← 同时执行
   ┌─┐ ┌─┐ ┌─┐
   │A│ │B│ │C│
   └┬┘ └┬┘ └┬┘
     ↓   ↓   ↓
   ┌──────────┐
   │Result Aggregator│
   └──────────┘
```

**适用场景**：
- 竞品分析：多个 Agent 同时研究不同竞品
- 多角度审查：安全、性能、可维护性 Agent 各审查同一代码
- 多语言翻译：同一内容同时翻译成多种语言

---

## 🎯 如何选择合适的模式？

**渐进式升级路径**：

```
单个 Agent + 工具
    ↓（开始出现上下文污染）
单个 Agent + 技能
    ↓（内存膨胀、成本螺旋）
主管 + 2-3 个专家 Agent
    ↓（需要更复杂协作）
流水线/并行混合模式
```

> **核心原则**：加工具优于加 Agent。只有遇到明显瓶颈时才升级到多 Agent 模式。

---

## 🔒 生产环境最佳实践

### 1. 安全沙箱配置

```json
{
  "agents": {
    "list": [
      {
        "id": "writer",
        "tools": {
          "allow": ["read", "browser"],
          "deny": ["exec", "write"]
        }
      },
      {
        "id": "coder",
        "tools": {
          "allow": ["exec", "read", "write"],
          "deny": ["browser"]
        }
      }
    ]
  }
}
```

### 2. 灵活的模型 - 渠道配对

| 渠道 | 推荐模型 | 原因 |
|------|---------|------|
| 飞书（日常） | deepseek | 性价比高，响应快 |
| WhatsApp | claude-sonnet | 快速准确，适合移动端 |
| Telegram（深度任务） | claude-opus | 深度思考，适合复杂问题 |

### 3. 技能共享机制

```
~/.openclaw/skills/              # 全局共享技能
├── web-search/                  # 搜索能力（所有 Agent 共享）
├── file-tools/                  # 文件操作（所有 Agent 共享）
└── image-gen/                   # 图像生成（所有 Agent 共享）

~/.openclaw/workspace-writer/skills/  # Writer 专属技能
├── seo-checker/                 # SEO 检查
└── article-template/            # 文章模板

~/.openclaw/workspace-coder/skills/   # Coder 专属技能
├── code-review/                 # 代码审查
└── test-runner/                 # 测试运行
```

### 4. 监控与调试

- **配置 HEARTBEAT.md**：定期检查 Agent 是否在线和正常响应
- **设置日志级别**：开发时启用详细日志，观察 Agent 间通信
- **建立回退机制**：专家 Agent 无响应时，主管 Agent 应自行处理或通知用户

---

## ❓ 常见问题与避坑指南

### Q1: Agent 越多越好吗？

**不是**。每个额外 Agent 都会增加通信开销和管理成本。

**建议**：
- 个人使用：3-5 个 Agent 足够（主管 + 2-4 个专家）
- 团队使用：按业务线划分，每条线 2-3 个 Agent
- 关键指标：如果两个 Agent 的对话 80% 以上是相同任务类型，考虑合并

### Q2: Agent 间通信会增加成本吗？

**会**。每次 `sessions_send` 都是一次 API 调用。

**降低不必要通信的方法**：
- 主管 Agent 先评估任务复杂度，简单任务直接处理
- 使用 Token 优化策略减少每次通信的上下文长度
- 给子 Agent 分配更轻量级的模型

### Q3: 如何处理 Agent 间的"理解偏差"？

当 Agent A 的输出交给 Agent B 时，可能出现误解。

**解决方案**：
- ✅ 结构化通信：Agent 间传递结构化 JSON 数据而非自由文本
- ✅ 模板化指令：主管 Agent 使用标准化指令模板分发任务
- ✅ 验证检查点：在关键节点增加审查步骤

### Q4: Agent 可以跨平台协作吗？

**可以**。OpenClaw 不仅支持飞书，还支持 Telegram、Discord、WhatsApp 等。甚至可以让一个飞书群 Agent 和一个 Telegram Agent 通过 `sessions_send` 协作。

---

## 📚 总结

多 Agent 架构不是什么高深技术，其核心理念与管理公司完全相同：

1. **选对人**（创建合适的 Agent，选择合适的模型）
2. **分好工**（编写灵魂文件，定义角色边界）
3. **建通道**（配置路由绑定和 Agent 间通信）
4. **立监督**（设置主管 Agent，建立质量控制）

从今天开始，你不再需要一个"无所知但常犯错"的 AI 助理。你需要的是一个**专业化、高效协作的 AI 团队**。

> 💡 **最后一条经验法则**：如果你发现自己经常对 AI 说"忽略那个，专注这个"，那就是时候拆分成多个 Agent 了。

---

## 📖 延伸阅读

- [OpenClaw 详细配置教程](https://github.com/openclaw/openclaw)
- [OpenClaw 自动化架构深度解析](https://github.com/openclaw/openclaw)
- [Agent Skills：自然语言编程时代](https://github.com/openclaw/openclaw)

---

<div align="center">

**文档整理**: OpenClaw AI 助理 | **更新时间**: 2026-03-13

[返回顶部](#openclaw-multi-agent-协作指南)

</div>
