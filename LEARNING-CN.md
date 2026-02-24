# Dr. Ralph 插件学习文档

本文档提供 Dr. Ralph 项目的完整中文学习指南，帮助你理解 Claude Code 插件的开发原理和架构设计。

---

## 目录

1. [项目概述](#1-项目概述)
2. [核心概念：Ralph 技术](#2-核心概念ralph-技术)
3. [项目结构](#3-项目结构)
4. [核心组件详解](#4-核心组件详解)
5. [Stop Hook 循环系统](#5-stop-hook-循环系统)
6. [诊断工作流](#6-诊断工作流)
7. [命令开发](#7-命令开发)
8. [本地开发与测试](#8-本地开发与测试)
9. [扩展开发指南](#9-扩展开发指南)

---

## 1. 项目概述

**Dr. Ralph** 是一个 Claude Code 插件，用于 AI 辅助医疗诊断。它实现了 "Ralph 技术" —— 一种使用 Claude Code 的 Stop Hook 系统创建的**自引用 AI 循环模式**，通过迭代方式完成结构化的 5 阶段诊断工作流。

### 核心特性

- **自引用循环**：通过 Stop Hook 实现 AI 自我迭代
- **阶段化工作流**：问诊 → 研究 → 鉴别诊断 → 治疗方案 → 报告生成
- **持久化存储**：患者病历跨会话保存
- **交互式问诊**：使用 `AskUserQuestion` 工具进行结构化访谈

---

## 2. 核心概念：Ralph 技术

### 什么是 Ralph 技术？

Ralph 技术是一种让 Claude 在 `while-true` 循环中持续运行的模式，直到任务完成。其核心原理是：

```
用户请求 → Claude 处理 → 尝试停止 → Stop Hook 拦截 → 检查完成条件 → 未完成则继续循环
```

### 三大核心要素

| 要素 | 说明 |
|------|------|
| **状态文件** | 存储循环状态和提示词 |
| **Stop Hook** | 拦截 Claude 的停止请求 |
| **完成条件** | `<promise>TEXT</promise>` 标签或最大迭代次数 |

---

## 3. 项目结构

```
dr-ralph/
├── .claude-plugin/
│   ├── plugin.json        # 插件元数据（名称、描述、作者）
│   └── marketplace.json   # 市场注册信息（用于分发）
├── commands/              # 斜杠命令定义
│   ├── diagnose.md        # 主诊断工作流命令
│   ├── cancel.md          # 取消活动会话
│   └── help.md            # 帮助文档
├── hooks/
│   ├── hooks.json         # Stop Hook 注册配置
│   └── stop-hook.sh       # 循环控制逻辑（Bash 脚本）
├── scripts/
│   └── setup-dr-ralph-diagnose.sh  # 会话初始化脚本
└── docs/
    └── diagnose-spec.md   # 完整诊断规范
```

---

## 4. 核心组件详解

### 4.1 插件元数据 (.claude-plugin/)

**plugin.json** - 定义插件基本信息：

```json
{
  "name": "dr-ralph",
  "description": "Implementation of the Dr. Ralph technique...",
  "author": {
    "name": "Mike Endale",
    "email": "mike@blencorp.com"
  }
}
```

**marketplace.json** - 用于市场分发：

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "dr-ralph-marketplace",
  "plugins": [{
    "name": "dr-ralph",
    "version": "1.0.0",
    "category": "healthcare",
    "tags": ["medical", "diagnostics", "SOAP"]
  }]
}
```

### 4.2 命令定义 (commands/)

命令使用 **Markdown + YAML 前置元数据** 格式定义：

```yaml
---
description: "命令描述"
argument-hint: "用法提示 [选项]"
allowed-tools: ["Bash(script.sh)", "Read", "Write", "AskUserQuestion"]
hide-from-slash-command-tool: "true"  # 防止递归调用
---

# 命令详细说明（Markdown 格式）
```

### 4.3 Hook 注册 (hooks/hooks.json)

```json
{
  "description": "Dr. Ralph plugin stop hook",
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/hooks/stop-hook.sh"
      }]
    }]
  }
}
```

**关键点**：
- `Stop` 事件触发 hook
- `${CLAUDE_PLUGIN_ROOT}` 是插件根目录的环境变量
- 执行 `stop-hook.sh` 脚本

---

## 5. Stop Hook 循环系统

### 5.1 stop-hook.sh 的运行机制详解

#### 触发时机

`stop-hook.sh` 是在 **Claude 准备停止/退出时** 被自动触发的。具体来说：

```
Claude 完成响应 → 准备停止会话 → 触发 Stop Hook → stop-hook.sh 执行
```

#### 注册机制

在 `hooks/hooks.json` 中注册：

```json
{
  "hooks": {
    "Stop": [{                    // ← 监听 "Stop" 事件
      "hooks": [{
        "type": "command",        // ← 类型是命令
        "command": "${CLAUDE_PLUGIN_ROOT}/hooks/stop-hook.sh"
      }]
    }]
  }
}
```

这是 Claude Code 插件系统的 **Hook API**，不是普通的命令调用。

#### 执行流程图

```
┌─────────────────────────────────────────────────────────────┐
│  Claude Code 会话生命周期                                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 用户输入 → Claude 处理 → 输出响应                          │
│                                                             │
│  2. Claude 认为任务完成，准备停止                              │
│            ↓                                                │
│  3. Claude Code 检查是否有注册的 Stop Hook                     │
│            ↓                                                │
│  4. 如果有 → 执行 stop-hook.sh                               │
│            │                                                │
│            ├─ stdin 输入: JSON 格式的会话信息                  │
│            │  {                                             │
│            │    "transcript_path": "/path/to/transcript",    │
│            │    ...                                         │
│            │  }                                             │
│            ↓                                                │
│  5. stop-hook.sh 决策:                                       │
│            │                                                │
│            ├─ exit 0 (无输出) → 允许 Claude 停止              │
│            │                                                │
│            └─ 输出 JSON {"decision": "block", ...}           │
│               → 阻止停止，将 "reason" 作为新输入发给 Claude     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 两种退出方式

```bash
# 方式 1：允许退出（exit 0，无 JSON 输出）
if [[ ! -f "$RALPH_STATE_FILE" ]]; then
  exit 0  # Claude 正常停止
fi

# 方式 2：阻止退出（输出 JSON）
jq -n \
  --arg prompt "$PROMPT_TEXT" \
  '{
    "decision": "block",      # ← 关键：block 表示阻止停止
    "reason": $prompt         # ← 这个会作为新输入发给 Claude
  }'
```

#### 本质是什么？

`stop-hook.sh` **既是一个 command，也是一个事件处理器**：

| 角度 | 说明 |
|------|------|
| **形式上** | 是一个 Bash 脚本/命令 |
| **功能上** | 是 Stop 事件的拦截器/处理器 |
| **机制上** | Claude Code 在特定时机自动调用它 |

#### 类比理解

可以把它类比为 Web 开发中的事件拦截器：

```javascript
// 类似 Node.js 的中间件
app.on('stop', async (event) => {
  const decision = await stopHook(event)

  if (decision.decision === 'block') {
    event.preventDefault()        // 阻止停止
    event.resumeWith(decision.reason)  // 用新输入继续
  }
  // 否则允许停止
})
```

#### 输入输出总结

```
输入 (stdin):
┌─────────────────────────────────┐
│ {                               │
│   "transcript_path": "...",     │  ← 会话记录文件路径
│   ...                           │
│ }                               │
└─────────────────────────────────┘

输出:
┌─────────────────────────────────┐
│ 情况1: exit 0 (无输出)           │  → 允许停止
│                                 │
│ 情况2: JSON                     │  → 阻止停止，继续循环
│ {                               │
│   "decision": "block",          │
│   "reason": "提示词...",         │
│   "systemMessage": "状态消息"    │
│ }                               │
└─────────────────────────────────┘
```

**这就是 Ralph 技术的核心** —— 通过 **拦截停止事件** 并 **注入新输入** 来实现循环迭代。

---

### 5.2 完整工作原理图

```
┌─────────────────────────────────────────────────────────┐
│                    Claude Code 会话                      │
├─────────────────────────────────────────────────────────┤
│  1. 用户执行 /dr-ralph:diagnose                          │
│  2. setup-dr-ralph-diagnose.sh 创建状态文件               │
│  3. Claude 开始执行诊断流程                               │
│  4. Claude 尝试停止                                       │
│  5. Stop Hook (stop-hook.sh) 被触发                      │
│     ├─ 检查状态文件是否存在                               │
│     ├─ 解析迭代次数和完成条件                             │
│     ├─ 检查输出中是否有 <promise>TEXT</promise>           │
│     └─ 决定：继续循环 或 允许退出                          │
│  6. 如果继续：返回 JSON 阻止退出，重新输入提示词            │
└─────────────────────────────────────────────────────────┘
```

### 5.3 状态文件格式

位置：`.claude/dr-ralph-loop.local.md`

```markdown
---
active: true
iteration: 1
max_iterations: 0
completion_promise: "DONE"
started_at: "2024-01-05T10:00:00Z"
diagnose_mode: true
patient_name: "John Doe"
patient_file: "@notes/john-doe.md"
report_file: "@notes/john-doe-report-20240105-100000.md"
question_count: 15
output_dir: "@notes"
---

[提示词内容]
```

### 5.4 Stop Hook 核心逻辑 (stop-hook.sh)

```bash
#!/bin/bash
set -euo pipefail

# 1. 读取 hook 输入（来自 stdin）
HOOK_INPUT=$(cat)

# 2. 检查状态文件是否存在
RALPH_STATE_FILE=".claude/dr-ralph-loop.local.md"
if [[ ! -f "$RALPH_STATE_FILE" ]]; then
  exit 0  # 无活动循环，允许退出
fi

# 3. 解析前置元数据
FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$RALPH_STATE_FILE")
ITERATION=$(echo "$FRONTMATTER" | grep '^iteration:' | sed 's/iteration: *//')
MAX_ITERATIONS=$(echo "$FRONTMATTER" | grep '^max_iterations:' | sed 's/max_iterations: *//')
COMPLETION_PROMISE=$(echo "$FRONTMATTER" | grep '^completion_promise:' | ...)

# 4. 检查是否达到最大迭代次数
if [[ $MAX_ITERATIONS -gt 0 ]] && [[ $ITERATION -ge $MAX_ITERATIONS ]]; then
  rm "$RALPH_STATE_FILE"
  exit 0
fi

# 5. 读取 Claude 最后的输出
TRANSCRIPT_PATH=$(echo "$HOOK_INPUT" | jq -r '.transcript_path')
LAST_OUTPUT=$(...从 transcript 提取最后一条 assistant 消息...)

# 6. 检查完成条件 <promise>TEXT</promise>
PROMISE_TEXT=$(echo "$LAST_OUTPUT" | perl -0777 -pe 's/.*?<promise>(.*?)<\/promise>.*/$1/s')
if [[ "$PROMISE_TEXT" = "$COMPLETION_PROMISE" ]]; then
  rm "$RALPH_STATE_FILE"
  exit 0
fi

# 7. 未完成 - 继续循环
NEXT_ITERATION=$((ITERATION + 1))
PROMPT_TEXT=$(awk '/^---$/{i++; next} i>=2' "$RALPH_STATE_FILE")

# 8. 更新迭代次数
sed "s/^iteration: .*/iteration: $NEXT_ITERATION/" "$RALPH_STATE_FILE" > ...

# 9. 输出 JSON 阻止退出
jq -n \
  --arg prompt "$PROMPT_TEXT" \
  --arg msg "🔄 Dr. Ralph iteration $NEXT_ITERATION" \
  '{
    "decision": "block",
    "reason": $prompt,
    "systemMessage": $msg
  }'
```

### 5.5 Stop Hook JSON 响应格式

```json
{
  "decision": "block",        // "block" 阻止退出，"allow" 允许退出
  "reason": "提示词内容",       // 将作为新输入发送给 Claude
  "systemMessage": "状态消息"  // 显示给用户的系统消息
}
```

---

## 6. 诊断工作流

### 6.1 五阶段流程

| 阶段 | 描述 | 主要工具 |
|------|------|----------|
| 1. 问诊 (Interview) | 医疗记录接收 + 症状访谈 | `AskUserQuestion`, `Read` |
| 2. 研究 (Research) | 搜索文献和治疗指南 | `WebSearch`, `WebFetch` |
| 3. 鉴别诊断 (Differential) | 分析症状确定诊断 | 内部分析 |
| 4. 治疗方案 (Treatment) | 制定研究支持的行动计划 | 内部分析 |
| 5. 报告 (Report) | 生成 SOAP 格式文档 | `Write` |

### 6.2 医疗记录处理规则

```
1. 首先询问是否有医疗记录
2. 使用 ls -la 或 stat 检查文件大小
3. 逐个处理文件（不能同时处理多个）
4. 大文件处理（>3MB）：
   - 警告用户
   - 提供跳过或提供小文件的选项
   - 建议使用 Adobe Acrobat 拆分 PDF
```

### 6.3 诊断输出规则

- **置信度 >80%**：输出单一最可能诊断
- **置信度不足**：输出 3-5 个鉴别诊断，按可能性排序

### 6.4 SOAP 报告格式

```markdown
# 患者报告：[患者姓名]
## 日期：[时间戳]

## 执行摘要
[2-3 段概述发现和建议]

---

## 主诉 (Subjective)
[患者报告的症状和病史]

## 客观发现 (Objective)
[文件分析和研究发现]

## 评估 (Assessment)
[诊断或鉴别诊断及置信度]

## 计划 (Plan)
[结构化治疗方案]
[随访计划]

---

## 详细发现
[完整访谈记录]
[研究分析]
[鉴别推理]

## 参考文献
[引用来源]
```

---

## 7. 命令开发

### 7.1 命令文件结构

```markdown
---
description: "命令描述"
argument-hint: "参数提示"
allowed-tools: ["工具列表"]
hide-from-slash-command-tool: "true"
---

# 命令标题

命令详细说明...

## 子章节

更多说明...
```

### 7.2 allowed-tools 配置

```yaml
# 允许特定脚本
allowed-tools: ["Bash(${CLAUDE_PLUGIN_ROOT}/scripts/setup.sh)"]

# 允许特定文件操作
allowed-tools: ["Bash(ls *)", "Bash(stat *)", "Read", "Write"]

# 允许交互
allowed-tools: ["AskUserQuestion"]

# 允许网络搜索
allowed-tools: ["WebSearch", "WebFetch"]
```

### 7.3 调用设置脚本

在命令中调用脚本：

```markdown
执行设置脚本初始化工作流：

`"${CLAUDE_PLUGIN_ROOT}/scripts/setup-dr-ralph-diagnose.sh" $ARGUMENTS`
```

---

## 8. 本地开发与测试

### 8.1 本地测试

```bash
# 方法 1：指定插件目录运行
claude --plugin-dir /path/to/dr-ralph

# 方法 2：在项目目录内运行
cd /path/to/dr-ralph
claude --plugin-dir .
```

### 8.2 生产安装

```bash
# 从市场安装
claude plugin marketplace add blencorp/dr-ralph
claude plugin install dr-ralph
```

### 8.3 配置文件安装

添加到 `.claude/settings.json`：

```json
{
  "marketplaces": ["blencorp/dr-ralph"],
  "plugins": {
    "dr-ralph@blencorp": "enabled"
  }
}
```

### 8.4 验证安装

```bash
/dr-ralph:help
```

---

## 9. 扩展开发指南

### 9.1 添加新命令

1. 在 `commands/` 目录创建 `.md` 文件
2. 添加 YAML 前置元数据
3. 编写命令说明和逻辑

### 9.2 修改循环逻辑

编辑 `hooks/stop-hook.sh`：
- 修改完成条件检测
- 调整迭代控制
- 添加新的状态字段

### 9.3 修改诊断流程

编辑 `scripts/setup-dr-ralph-diagnose.sh`：
- 修改提示词模板
- 调整阶段流程
- 添加新的配置选项

### 9.4 修改诊断规范

编辑 `docs/diagnose-spec.md`：
- 更新工作流定义
- 修改输出格式
- 调整安全协议

---

## 关键文件修改对照表

| 修改目标 | 文件 |
|----------|------|
| 插件元数据 | `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json` |
| 命令行为 | `commands/*.md` |
| 循环逻辑 | `hooks/stop-hook.sh` |
| 会话设置 | `scripts/setup-dr-ralph-diagnose.sh` |
| 诊断规范 | `docs/diagnose-spec.md` |

---

## 医疗免责声明

本插件仅供信息参考和教育目的。**不能替代专业医疗建议**。所有输出必须包含适当的免责声明。

---

## 参考资源

- [原始 Ralph 技术](https://ghuntley.com/ralph/)
- [Ralph Orchestrator](https://github.com/mikeyobrien/ralph-orchestrator)
- [Ralph Wiggum Plugin](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/ralph-wiggum)
- [Claude Code 插件文档](https://code.claude.com/docs/en/plugins)
