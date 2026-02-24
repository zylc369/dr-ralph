# Dr. Ralph OpenCode Migration Plan

## TL;DR

> **Quick Summary**: Migrate the Dr. Ralph Claude Code plugin to OpenCode, using Oh My OpenCode's ralph-loop hook for self-referential loop capability. Place migrated code in `opencode/` directory.
>
> **Deliverables**:
> - OpenCode commands in `opencode/.opencode/commands/`
> - Diagnostic skill/workflow in `opencode/.opencode/skills/`
> - Optional plugin for loop control in `opencode/.opencode/plugins/`
> - Updated documentation
>
> **Estimated Effort**: Medium
> **Parallel Execution**: YES - 3 waves
> **Critical Path**: API Verification → Commands Migration → Loop Integration → E2E Test

---

## Context

### Original Request
将 Dr. Ralph Claude Code 插件改造成 OpenCode 可用的插件，放置在 `opencode/` 目录下。

### Interview Summary
**Key Discussions**:
- **运行环境**: Oh My OpenCode (增强版 OpenCode)
- **循环机制**: 使用 Oh My OpenCode 内置的 ralph-loop hook
- **功能范围**: 完整迁移（保留所有诊断阶段、状态管理、患者记录）
- **验证方式**: Agent 自动验证

**Research Findings**:
- OpenCode 没有 Stop Hook 等价机制
- Oh My OpenCode 提供 ralph-loop hook 用于自引用循环
- OpenCode 命令格式：Markdown + YAML frontmatter 或 JSON
- OpenCode 工具权限格式与 Claude Code 不同（布尔值 vs 数组）

### Metis Review
**Identified Gaps** (to be addressed in plan):
- `/ralph-loop` 命令 API 需要验证
- `askquestion` 工具 API 需要测试
- `websearch`/`webfetch` 工具可用性需要确认
- 状态文件位置策略需要明确

---

## Work Objectives

### Core Objective
将 Dr. Ralph 的完整诊断工作流迁移到 OpenCode，保持功能对等，利用 Oh My OpenCode 的 ralph-loop hook 实现自引用循环。

### Concrete Deliverables
- `opencode/.opencode/commands/diagnose.md` - 主诊断命令
- `opencode/.opencode/commands/cancel.md` - 取消会话命令
- `opencode/.opencode/commands/help.md` - 帮助命令
- `opencode/.opencode/skills/dr-ralph/SKILL.md` - 诊断技能定义
- `opencode/.opencode/plugins/dr-ralph-loop.ts` - 循环控制插件（可选）
- `opencode/README.md` - 使用文档
- `opencode/AGENTS.md` - 项目规则

### Definition of Done
- [ ] 所有命令可通过 `/dr-ralph:diagnose`、`/dr-ralph:cancel`、`/dr-ralph:help` 执行
- [ ] 诊断工作流完整执行 5 个阶段
- [ ] 患者记录在 `@notes/` 目录正确创建和更新
- [ ] 自引用循环正常工作（通过 ralph-loop 或替代方案）
- [ ] Agent 验证场景全部通过

### Must Have
- 完整的 5 阶段诊断工作流（问诊 → 研究 → 鉴别诊断 → 治疗方案 → 报告）
- 患者记录持久化（`@notes/[patient].md`）
- SOAP 格式报告生成
- 与 Oh My OpenCode ralph-loop hook 的集成
- 医疗记录文件大小检查（3MB 限制）
- 紧急症状标记

### Must NOT Have (Guardrails)
- ❌ Marketplace 集成（OpenCode 无等价机制）
- ❌ 重新实现 Stop Hook（使用 ralph-loop 或替代方案）
- ❌ 状态文件迁移工具
- ❌ 独立的 `@hona/ralph-cli` 集成
- ❌ 多患者数据库（保持文件式 `@notes/`）
- ❌ 通用 Ralph Loop 抽象层
- ❌ 配置系统（仅使用命令行参数）
- ❌ 增强功能（仅保持功能对等）
- ❌ 多模型支持（使用默认模型）
- ❌ UI/UX 改进
- ❌ 自动化测试套件（除非明确要求）
- ❌ AI slop 模式：过度注释、过度抽象、通用命名

---

## Verification Strategy

> **ZERO HUMAN INTERVENTION** — ALL verification is agent-executed.

### Test Decision
- **Infrastructure exists**: NO (OpenCode plugin testing is manual)
- **Automated tests**: None
- **Agent-Executed QA**: YES - Playwright for TUI, manual command execution verification

### QA Policy
Every task includes agent-executed QA scenarios:
- **CLI/Command**: Execute command, validate output, check file creation
- **File Operations**: Verify files exist, content format correct
- **Workflow**: Run full diagnostic flow, capture evidence

---

## Execution Strategy

### Parallel Execution Waves

```
Wave 1 (API Verification - Sequential, ~15 min):
├── Task 1: Verify ralph-loop hook API [quick]
├── Task 2: Verify askquestion tool API [quick]
└── Task 3: Verify websearch/webfetch availability [quick]

Wave 2 (Core Migration - Parallel, ~30 min):
├── Task 4: Create diagnose command [quick]
├── Task 5: Create cancel command [quick]
├── Task 6: Create help command [quick]
├── Task 7: Create dr-ralph skill [quick]
└── Task 8: Create AGENTS.md [quick]

Wave 3 (Loop Integration - Sequential, ~20 min):
├── Task 9: Implement loop control logic [unspecified-high]
└── Task 10: Create README documentation [quick]

Wave 4 (Verification - Parallel, ~15 min):
├── Task 11: E2E test - basic diagnostic flow [unspecified-high]
├── Task 12: E2E test - state management [unspecified-high]
└── Task 13: E2E test - cancellation [unspecified-high]

Critical Path: Task 1 → Task 2 → Task 3 → Task 4 → Task 9 → Task 11
Parallel Speedup: ~40% faster than sequential
Max Concurrent: 5 (Wave 2)
```

### Dependency Matrix

- **1-3**: — — 4-8, 9
- **4-8**: 1-3 — 9, 10
- **9**: 1-8 — 11-13
- **10**: 4-8 — —
- **11-13**: 9 — —

### Agent Dispatch Summary

- **Wave 1**: **3** — T1-T3 → `quick`
- **Wave 2**: **5** — T4-T8 → `quick`
- **Wave 3**: **2** — T9 → `unspecified-high`, T10 → `quick`
- **Wave 4**: **3** — T11-T13 → `unspecified-high`

---

## TODOs

- [x] 1. **Verify ralph-loop hook API**

  **What to do**:
  - 在 Oh My OpenCode 环境中验证 `/ralph-loop` 命令的可用性和参数格式
  - 确认如何从命令中程序化调用 ralph-loop
  - 确认状态文件位置（`.sisyphus/` vs `.opencode/`）
  - 确认完成条件检测机制（`<promise>` 标签或 API）

  **Must NOT do**:
  - 不要假设 API 格式，必须实际验证
  - 不要尝试重新实现 ralph-loop 逻辑

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: 简单的 API 验证任务
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 1 (Sequential)
  - **Blocks**: Tasks 4-9
  - **Blocked By**: None

  **References**:
  - `commands/diagnose.md:1-57` - 当前命令格式参考
  - `hooks/stop-hook.sh:1-177` - 当前循环控制逻辑
  - OpenCode 文档: https://opencode.ai/docs/plugins/

  **Acceptance Criteria**:
  - [ ] 确认 `/ralph-loop` 命令是否可用
  - [ ] 文档化命令参数格式
  - [ ] 确认状态文件位置

  **QA Scenarios**:
  ```
  Scenario: Verify ralph-loop availability
    Tool: Bash
    Steps:
      1. Check if ralph-loop hook exists in Oh My OpenCode config
      2. Document command syntax and parameters
    Expected Result: API documentation created
    Evidence: .sisyphus/evidence/task-01-ralph-loop-api.md
  ```

  **Commit**: NO

---

- [x] 2. **Verify askquestion tool API**

  **What to do**:
  - 验证 OpenCode 的 `askquestion` 工具（对应 Claude Code 的 `AskUserQuestion`）
  - 确认支持单选/多选模式
  - 确认选项描述格式
  - 确认超时和取消行为

  **Must NOT do**:
  - 不要假设 API 与 Claude Code 完全相同

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: 简单的工具验证任务
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 1 (Sequential, after Task 1)
  - **Blocks**: Tasks 4-9
  - **Blocked By**: Task 1

  **References**:
  - `commands/diagnose.md:4` - 当前使用 AskUserQuestion 的方式
  - OpenCode 工具文档

  **Acceptance Criteria**:
  - [ ] 确认 askquestion 工具可用性
  - [ ] 文档化 API 格式

  **QA Scenarios**:
  ```
  Scenario: Verify askquestion tool
    Tool: Bash
    Steps:
      1. Test askquestion with single-select options
      2. Test askquestion with multi-select options
    Expected Result: API documentation created
    Evidence: .sisyphus/evidence/task-02-askquestion-api.md
  ```

  **Commit**: NO

---

- [x] 3. **Verify websearch/webfetch availability**

  **What to do**:
  - 确认 OpenCode 中 `websearch` 和 `webfetch` 工具的可用性
  - 确认这些是内置工具还是需要 MCP 配置
  - 文档化 API 格式

  **Must NOT do**:
  - 不要假设工具自动可用

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: 简单的工具验证任务
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 1 (Sequential, after Task 2)
  - **Blocks**: Tasks 4-9
  - **Blocked By**: Task 2

  **References**:
  - `commands/diagnose.md:4` - 当前使用 WebSearch, WebFetch

  **Acceptance Criteria**:
  - [ ] 确认 websearch/webfetch 可用性
  - [ ] 文档化 API 格式

  **QA Scenarios**:
  ```
  Scenario: Verify web tools
    Tool: Bash
    Steps:
      1. Check if websearch tool exists
      2. Check if webfetch tool exists
    Expected Result: API documentation created
    Evidence: .sisyphus/evidence/task-03-web-tools-api.md
  ```

  **Commit**: NO

---
- [ ] 4. **Create diagnose command**

  **What to do**:
  - 创建 `opencode/.opencode/commands/diagnose.md`
  - 将 Claude Code 命令格式转换为 OpenCode 格式
  - 保留完整的 5 阶段诊断工作流
  - 调整工具权限格式（布尔值 vs 数组）

  **Must NOT do**:
  - 不要改变诊断工作流逻辑
  - 不要添加新功能

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: 文件格式转换任务
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 5-8)
  - **Blocks**: Task 9
  - **Blocked By**: Tasks 1-3

  **References**:
  - `commands/diagnose.md:1-57` - 源命令文件
  - `scripts/setup-dr-ralph-diagnose.sh:1-427` - 诊断提示词模板

  **Acceptance Criteria**:
  - [ ] 文件 `opencode/.opencode/commands/diagnose.md` 存在
  - [ ] YAML frontmatter 格式正确
  - [ ] 包含完整 5 阶段工作流说明

  **QA Scenarios**:
  ```
  Scenario: Verify diagnose command file
    Tool: Bash
    Steps:
      1. test -f opencode/.opencode/commands/diagnose.md
      2. grep -q "description:" opencode/.opencode/commands/diagnose.md
      3. grep -q "Phase 1" opencode/.opencode/commands/diagnose.md
    Expected Result: All checks pass
    Evidence: .sisyphus/evidence/task-04-diagnose-cmd.md
  ```

  **Commit**: YES (groups with 5-8)
  - Message: `feat(opencode): add diagnose command`
  - Files: `opencode/.opencode/commands/diagnose.md`

---

- [ ] 5. **Create cancel command**

  **What to do**:
  - 创建 `opencode/.opencode/commands/cancel.md`
  - 实现取消活动诊断会话的功能
  - 清理状态文件

  **Must NOT do**:
  - 不要添加额外的取消逻辑

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: 简单的命令文件创建
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 4, 6-8)
  - **Blocks**: Task 9
  - **Blocked By**: Tasks 1-3

  **References**:
  - `commands/cancel.md` - 源取消命令

  **Acceptance Criteria**:
  - [ ] 文件 `opencode/.opencode/commands/cancel.md` 存在
  - [ ] 包含状态文件清理逻辑

  **QA Scenarios**:
  ```
  Scenario: Verify cancel command file
    Tool: Bash
    Steps:
      1. test -f opencode/.opencode/commands/cancel.md
      2. grep -q "description:" opencode/.opencode/commands/cancel.md
    Expected Result: All checks pass
    Evidence: .sisyphus/evidence/task-05-cancel-cmd.md
  ```

  **Commit**: YES (groups with 4-8)

---

- [ ] 6. **Create help command**

  **What to do**:
  - 创建 `opencode/.opencode/commands/help.md`
  - 提供插件使用说明

  **Must NOT do**:
  - 不要添加新的帮助内容

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: 简单的命令文件创建
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 4-5, 7-8)
  - **Blocks**: None
  - **Blocked By**: Tasks 1-3

  **References**:
  - `commands/help.md` - 源帮助命令

  **Acceptance Criteria**:
  - [ ] 文件 `opencode/.opencode/commands/help.md` 存在

  **QA Scenarios**:
  ```
  Scenario: Verify help command file
    Tool: Bash
    Steps:
      1. test -f opencode/.opencode/commands/help.md
    Expected Result: File exists
    Evidence: .sisyphus/evidence/task-06-help-cmd.md
  ```

  **Commit**: YES (groups with 4-8)

---

- [ ] 7. **Create dr-ralph skill**

  **What to do**:
  - 创建 `opencode/.opencode/skills/dr-ralph/SKILL.md`
  - 定义诊断技能的元数据和使用说明
  - 包含 5 阶段工作流概述

  **Must NOT do**:
  - 不要将完整的诊断逻辑放在 skill 中（保留在 command 中）

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: 技能定义文件创建
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 4-6, 8)
  - **Blocks**: Task 9
  - **Blocked By**: Tasks 1-3

  **References**:
  - `docs/diagnose-spec.md` - 诊断规范
  - `LEARNING-CN.md` - 学习文档

  **Acceptance Criteria**:
  - [ ] 文件 `opencode/.opencode/skills/dr-ralph/SKILL.md` 存在
  - [ ] YAML frontmatter 包含 name 和 description

  **QA Scenarios**:
  ```
  Scenario: Verify skill file
    Tool: Bash
    Steps:
      1. test -f opencode/.opencode/skills/dr-ralph/SKILL.md
      2. grep -q "name:" opencode/.opencode/skills/dr-ralph/SKILL.md
    Expected Result: All checks pass
    Evidence: .sisyphus/evidence/task-07-skill.md
  ```

  **Commit**: YES (groups with 4-8)

---

- [ ] 8. **Create AGENTS.md**

  **What to do**:
  - 创建 `opencode/AGENTS.md`
  - 定义 OpenCode 项目规则
  - 包含目录结构和约定说明

  **Must NOT do**:
  - 不要添加与诊断无关的规则

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: 项目规则文件创建
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 2 (with Tasks 4-7)
  - **Blocks**: Task 10
  - **Blocked By**: Tasks 1-3

  **References**:
  - `LEARNING-CN.md` - 项目结构说明

  **Acceptance Criteria**:
  - [ ] 文件 `opencode/AGENTS.md` 存在

  **QA Scenarios**:
  ```
  Scenario: Verify AGENTS.md file
    Tool: Bash
    Steps:
      1. test -f opencode/AGENTS.md
    Expected Result: File exists
    Evidence: .sisyphus/evidence/task-08-agents.md
  ```

  **Commit**: YES (groups with 4-8)
  - Message: `feat(opencode): add commands, skills, and AGENTS.md`
  - Files: `opencode/.opencode/`, `opencode/AGENTS.md`

---
- [ ] 9. **Implement loop control logic**

  **What to do**:
  - 根据任务 1-3 的验证结果，实现自引用循环控制
  - 优先方案：使用 Oh My OpenCode 的 ralph-loop hook
  - 备选方案：创建 OpenCode 插件使用 session.idle 事件
  - 实现状态文件管理

  **Must NOT do**:
  - 不要重新实现完整的 Stop Hook 逻辑
  - 不要假设 ralph-loop 可用（需要验证后决定）

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: 需要根据 API 验证结果调整实现方案
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: NO
  - **Parallel Group**: Wave 3 (Sequential)
  - **Blocks**: Tasks 11-13
  - **Blocked By**: Tasks 1-8

  **References**:
  - `hooks/stop-hook.sh:1-177` - 当前循环控制逻辑
  - `hooks/hooks.json` - Hook 注册配置
  - 任务 1-3 的 API 验证结果

  **Acceptance Criteria**:
  - [ ] 循环控制逻辑已实现
  - [ ] 状态文件正确创建和更新
  - [ ] 完成条件检测正常工作

  **QA Scenarios**:
  ```
  Scenario: Test loop control
    Tool: Bash
    Steps:
      1. Verify loop control mechanism exists
      2. Test state file creation
      3. Test completion detection
    Expected Result: All tests pass
    Evidence: .sisyphus/evidence/task-09-loop-control.md
  ```

  **Commit**: YES
  - Message: `feat(opencode): implement loop control logic`

---

- [ ] 10. **Create README documentation**

  **What to do**:
  - 创建 `opencode/README.md`
  - 包含安装说明、使用示例、依赖要求
  - 说明与 Claude Code 版本的差异

  **Must NOT do**:
  - 不要复制 Claude Code 版本的 README（需要更新）

  **Recommended Agent Profile**:
  - **Category**: `quick`
    - Reason: 文档创建任务
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 3 (with Task 9)
  - **Blocks**: None
  - **Blocked By**: Tasks 4-8

  **References**:
  - `README.md` - 源 README
  - `LEARNING-CN.md` - 学习文档

  **Acceptance Criteria**:
  - [ ] 文件 `opencode/README.md` 存在
  - [ ] 包含安装和使用说明

  **QA Scenarios**:
  ```
  Scenario: Verify README
    Tool: Bash
    Steps:
      1. test -f opencode/README.md
      2. grep -q "安装" opencode/README.md || grep -q "Install" opencode/README.md
    Expected Result: All checks pass
    Evidence: .sisyphus/evidence/task-10-readme.md
  ```

  **Commit**: YES
  - Message: `docs(opencode): add README`

---

- [ ] 11. **E2E test - basic diagnostic flow**

  **What to do**:
  - 执行完整的诊断工作流测试
  - 验证 5 个阶段正常执行
  - 验证患者记录和报告正确生成

  **Must NOT do**:
  - 不要跳过任何阶段验证

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: 需要 E2E 测试和验证
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 4 (with Tasks 12-13)
  - **Blocks**: None
  - **Blocked By**: Task 9

  **References**:
  - `docs/diagnose-spec.md` - 诊断规范

  **Acceptance Criteria**:
  - [ ] 诊断命令可执行
  - [ ] 5 个阶段都有输出
  - [ ] 患者记录文件创建
  - [ ] SOAP 报告生成

  **QA Scenarios**:
  ```
  Scenario: Full diagnostic flow
    Tool: interactive_bash (tmux)
    Steps:
      1. Start OpenCode with plugin
      2. Execute /dr-ralph:diagnose "test symptoms" --patient "test-user"
      3. Wait for completion
      4. Verify @notes/test-user.md exists
      5. Verify @notes/test-user-report-*.md exists
    Expected Result: All files created, workflow complete
    Evidence: .sisyphus/evidence/task-11-e2e-basic.md
  ```

  **Commit**: NO

---

- [ ] 12. **E2E test - state management**

  **What to do**:
  - 验证状态文件正确管理
  - 测试跨会话的患者记录持久化

  **Must NOT do**:
  - 不要假设状态文件位置

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: 需要 E2E 测试和验证
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 4 (with Tasks 11, 13)
  - **Blocks**: None
  - **Blocked By**: Task 9

  **References**:
  - `hooks/stop-hook.sh` - 状态文件逻辑

  **Acceptance Criteria**:
  - [ ] 状态文件正确创建
  - [ ] 患者记录跨会话持久化

  **QA Scenarios**:
  ```
  Scenario: State management
    Tool: Bash
    Steps:
      1. Create first session with patient "user1"
      2. Verify @notes/user1.md exists
      3. Create second session with same patient
      4. Verify previous notes are read
    Expected Result: Notes persist across sessions
    Evidence: .sisyphus/evidence/task-12-state-mgmt.md
  ```

  **Commit**: NO

---

- [ ] 13. **E2E test - cancellation**

  **What to do**:
  - 验证取消命令正常工作
  - 测试状态文件清理

  **Must NOT do**:
  - 不要跳过清理验证

  **Recommended Agent Profile**:
  - **Category**: `unspecified-high`
    - Reason: 需要 E2E 测试和验证
  - **Skills**: []

  **Parallelization**:
  - **Can Run In Parallel**: YES
  - **Parallel Group**: Wave 4 (with Tasks 11-12)
  - **Blocks**: None
  - **Blocked By**: Task 9

  **References**:
  - `commands/cancel.md` - 取消命令

  **Acceptance Criteria**:
  - [ ] 取消命令可执行
  - [ ] 状态文件被清理

  **QA Scenarios**:
  ```
  Scenario: Cancellation
    Tool: Bash
    Steps:
      1. Start diagnostic session
      2. Execute /dr-ralph:cancel
      3. Verify state file is removed
    Expected Result: State file cleaned up
    Evidence: .sisyphus/evidence/task-13-cancellation.md
  ```

  **Commit**: NO

---
## Final Verification Wave (MANDATORY)

> 4 review agents run in PARALLEL. ALL must APPROVE.

- [ ] F1. **Plan Compliance Audit** — `oracle`
  Read the plan end-to-end. Verify:
  - All "Must Have" items are implemented
  - All "Must NOT Have" items are absent
  - Evidence files exist in `.sisyphus/evidence/`
  - Deliverables match plan
  Output: `Must Have [N/N] | Must NOT Have [N/N] | Tasks [N/N] | VERDICT: APPROVE/REJECT`

- [ ] F2. **Code Quality Review** — `unspecified-high`
  Review all created files for:
  - Valid YAML frontmatter format
  - No `as any`/`@ts-ignore` in TypeScript
  - No excessive comments or over-abstraction
  - Consistent naming conventions
  Output: `Files [N clean/N issues] | VERDICT`

- [ ] F3. **Real Manual QA** — `unspecified-high`
  Execute EVERY QA scenario from EVERY task:
  - Run diagnostic flow
  - Verify file creation
  - Test cancellation
  Save evidence to `.sisyphus/evidence/final-qa/`
  Output: `Scenarios [N/N pass] | VERDICT`

- [ ] F4. **Scope Fidelity Check** — `deep`
  For each task: verify 1:1 — everything in spec was built (no missing), nothing beyond spec was built (no creep).
  Check "Must NOT do" compliance.
  Output: `Tasks [N/N compliant] | VERDICT`

---

## Commit Strategy

| Commit | Message | Files |
|--------|---------|-------|
| 1 | `feat(opencode): add commands, skills, and AGENTS.md` | `opencode/.opencode/`, `opencode/AGENTS.md` |
| 2 | `feat(opencode): implement loop control logic` | `opencode/.opencode/plugins/` |
| 3 | `docs(opencode): add README` | `opencode/README.md` |

**Pre-commit**: None (no automated tests)

---

## Success Criteria

### Verification Commands
```bash
# Verify all command files exist
ls -la opencode/.opencode/commands/

# Verify skill file exists
test -f opencode/.opencode/skills/dr-ralph/SKILL.md && echo "PASS" || echo "FAIL"

# Verify AGENTS.md exists
test -f opencode/AGENTS.md && echo "PASS" || echo "FAIL"

# Verify README exists
test -f opencode/README.md && echo "PASS" || echo "FAIL"
```

### Final Checklist
- [ ] All "Must Have" present
- [ ] All "Must NOT Have" absent
- [ ] All commands discoverable via `/help`
- [ ] Diagnostic workflow executes 5 phases
- [ ] Patient notes persist in `@notes/`
- [ ] Self-referential loop works
- [ ] Evidence files captured








