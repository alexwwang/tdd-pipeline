# TDD Pipeline Skill

[English](#english) · [中文](#中文)

---

<a id="english"></a>

## Overview

A rigorous 6-phase TDD development workflow for AI-assisted coding. Phases 1–5 enforce **Red-Green-Refactor** at the pipeline level with mandatory Ralph-loop review. Phase 6 is the pipeline's **validation closure** — systematic pre-release testing with bug root cause analysis (追问), rollback paths, and user go/no-go decision.

```
Product Design → Technical Solution → Test Plan → Test Code → Business Code → Pre-Release Testing
    产品设计    →     技术方案      →   测试方案  →  测试代码  →  业务代码   →    预发布测试
```

> **Core Principle**: If you cannot write a failing test for it, you do not understand it well enough to build it.

## The 6 Phases

Phases 1–5 are **creation phases** with Ralph-loop review. Phase 6 is the **validation closure** with its own quality mechanisms.

| Phase | Deliverable | Quality Mechanism |
|-------|-------------|-------------------|
| 1. Product Design | Requirements Document (with core/secondary priorities) | Ralph design review |
| 2. Technical Solution | Technical Design Document (with key/peripheral priorities) | Ralph design review |
| 3. Test Plan | Test Plan Document (with coverage matrices + consistency checks) | Ralph design review |
| 4. Test Code | Test Files (all failing) | Ralph code review |
| 5. Business Code | Working Business Code | Ralph code review |
| 6. Pre-Release Testing | Release Gate Checklist (with evidence) | Testing flow + 追问 + user go/no-go |

## Ralph Loop Review (Phases 1–5)

Phases 1–5 each end with a mandatory review loop:

- **Up to 10 rounds** total
- **Early stop**: allowed at any round ≥ 2 when the previous 2 rounds were both zero-issue (zero C/H/M/L — stricter than gate pass, which tolerates L issues)
- **Floor**: if early stop never triggers, run at least 5 rounds before evaluating gate pass
- **Gate**: zero C/H/M issues to proceed (L and I are acceptable)
- **Escalation**: if C/H/M persist after 10 rounds → halt and escalate to user (NOT a pass path)
- **Independent reviewer** subagent for each round

Phase 6 does NOT use Ralph loop. See Phase 6 row above and `phase-6-pre-release-testing.md` Part 5 for its quality mechanisms: sub-phase gates, 追问 protocol, and user go/no-go.

## Priority Classification System

A three-tier priority chain ensures test depth is proportional to importance:

```
Phase 1: User Stories & ACs classified as core / secondary
    ↓ consistency validation
Phase 2: Components & failure modes classified as key / peripheral
    ↓ consistency validation
Phase 3: Test depth driven by upstream classifications
    core/key → comprehensive (happy path + edge cases + error scenarios)
    secondary/peripheral → basic coverage (happy path + primary error path)
```

**Cross-phase consistency rules**:
- Every core requirement must map to at least one key component (Phase 1→2)
- Every core item must appear in Core Scenarios, every key item in Key Functional Points (Phase 1→3, Phase 2→3)
- Priority downgrades require explicit justification; upgrades are flagged as potential scope creep
- No core scenario may map entirely to peripheral functional points

## Task Tree & Context Management

For complex tasks that exceed structural complexity thresholds, the pipeline supports modular decomposition:

- **Split trigger**: ≥ 3 modules OR ≥ 5 stories (phase-adaptive: stories = user stories / components / test groups / test modules / implementation files)
- **Execution**: index.md first (shared contracts) → modules in parallel → merged Ralph loop
- **Context reading**: incremental 5-level protocol with practical termination
- **Document versioning**: version headers with change tracking
- **Status tracking**: `tree.md` with rollback handling

See `task-tree.md` (loaded only when split is triggered).

## Progressive Disclosure

At each phase, only load the corresponding phase file. Do not load all files at once.

| File | Content |
|------|---------|
| `SKILL.md` | Overview, triggers, gate rules, split decision |
| `ralph-review-loop.md` | Shared review protocol with decision flowchart |
| `task-tree.md` | Task decomposition, context management, versioning (loaded only when SPLIT=true) |
| `phase-1-product-design.md` | Requirements gathering via deep-interview, core/secondary classification |
| `phase-2-technical-solution.md` | Architecture via Planner→Architect→Critic consensus, key/peripheral classification |
| `phase-3-test-plan.md` | Test strategy with coverage matrices and priority consistency validation |
| `phase-4-test-code.md` | Write all tests first (must compile, must fail at runtime) |
| `phase-5-business-code.md` | Implement minimum code to pass tests, then refactor |
| `phase-6-pre-release-testing.md` | Pre-release testing, 追问 protocol, rollback paths, release gate verification (pipeline validation closure) |
| `phase-6-root-cause-investigation.md` | Bug root cause investigation (Layer Isolation, 5-Why, Fix Verification) + common bug patterns. Loaded only when a Phase 6 sub-phase fails. |

## Usage

Use natural language triggers in your AI coding tool:

- `tdd pipeline <feature description>`
- `tdd`
- `write tests first`
- `test-driven`
- `red green refactor`
- `pre-release`
- `release test`
- `ship it`
- `go/no-go`
- `ready to deploy`
- `产品设计` / `技术方案` / `测试方案` / `测试驱动` / `先写测试` / `上线前测试` / `发版前检查` / `回归测试`

## Key Rules

- **TDD is non-negotiable**: no business code until tests exist and fail
- **Phases 1–5: Ralph loop gate** — zero M+ (C/H/M) issues to pass
- **Phase 6: validation closure** — sub-phase gates + 追问 protocol + user go/no-go (not Ralph loop)
- **Tests define the contract** — business code is just the implementation
- **Priority drives depth**: core/key items get comprehensive testing; secondary/peripheral get basic coverage
- **Cross-phase consistency**: priority classifications must be traceable and justified across all phases
- **Phase 6 failure → rollback**: root cause determines rollback target; full Phase 6 rerun after any fix

## Installation

Copy all files to your Claude Code skills directory:

```bash
# macOS/Linux
cp -r tdd-pipeline ~/.claude/skills/

# Verify
ls ~/.claude/skills/tdd-pipeline/
```

Or clone directly:

```bash
git clone <repo-url> ~/.claude/skills/tdd-pipeline
```

No build step or dependency installation required — skills are loaded on demand by the Claude Code skill system.

## File Structure

```
tdd-pipeline/
├── SKILL.md                        ← entry point (progressive disclosure hub)
├── ralph-review-loop.md            ← shared review protocol (Phases 1–5 only)
├── task-tree.md                    ← loaded only for complex tasks
├── phase-1-product-design.md
├── phase-2-technical-solution.md
├── phase-3-test-plan.md
├── phase-4-test-code.md
├── phase-5-business-code.md
├── phase-6-pre-release-testing.md  ← always loaded for Phase 6
├── phase-6-root-cause-investigation.md  ← loaded only on sub-phase failure
└── README.md
```

## License

MIT

---

<a id="中文"></a>

## 概述

一个严格的 6 阶段 TDD 开发工作流，专为 AI 辅助编程设计。阶段 1–5 在管线级别强制执行 **Red-Green-Refactor**，每个阶段结束后有 Ralph 循环审核。阶段 6 是管线的**验证闭环** —— 系统化预发布测试、缺陷根因追问（追问方法）、回滚路径和用户 go/no-go 决策。

> **核心原则**：如果你无法为某个功能写出一个失败的测试，说明你对它的理解还不够深入。

## 6 个阶段

阶段 1–5 是**创作阶段**，使用 Ralph 循环审核。阶段 6 是**验证闭环**，使用独立的质量机制。

| 阶段 | 交付物 | 质量机制 |
|------|--------|----------|
| 1. 产品设计 | 需求文档（含 core/secondary 优先级分类） | Ralph 设计审核 |
| 2. 技术方案 | 技术设计文档（含 key/peripheral 优先级分类） | Ralph 设计审核 |
| 3. 测试方案 | 测试计划文档（含覆盖矩阵 + 一致性校验） | Ralph 设计审核 |
| 4. 测试代码 | 测试文件（全部失败） | Ralph 代码审核 |
| 5. 业务代码 | 可工作的实现代码 | Ralph 代码审核 |
| 6. 预发布测试 | 发布关卡检查清单（含证据） | 测试流程 + 追问 + 用户 go/no-go |

## Ralph 循环审核（阶段 1–5）

阶段 1–5 每个结束后启动强制审核循环：

- **最多 10 轮**
- **早停**：任意 ≥ 2 轮处，前 2 轮均为零问题时可提前结束（零 C/H/M/L — 比关卡通过更严格，关卡允许 L 级问题）
- **最低轮次**：未触发早停时，至少跑 5 轮才能评估关卡通过
- **关卡条件**：零 C/H/M 问题方可进入下一阶段（L 和 I 可接受）
- **升级**：10 轮后 C/H/M 仍未解决则中止并升级给用户（不是通过路径）
- 每轮由**独立审核 subagent** 执行

阶段 6 **不使用** Ralph 循环。详见上方阶段 6 行及 `phase-6-pre-release-testing.md` Part 5。

## 优先级分类体系

三级优先级链确保测试深度与重要性成正比：

```
Phase 1: 用户故事与验收标准分类为 core / secondary
    ↓ 一致性校验
Phase 2: 组件与故障模式分类为 key / peripheral
    ↓ 一致性校验
Phase 3: 测试深度由上游分类驱动
    core/key → 全面覆盖（正常路径 + 边界情况 + 错误场景）
    secondary/peripheral → 基本覆盖（正常路径 + 主要错误路径）
```

**跨阶段一致性规则**：
- 每个核心需求必须映射到至少一个关键组件（Phase 1→2）
- 每个核心条目必须出现在核心场景中，每个关键条目必须出现在关键功能点中（Phase 1→3, Phase 2→3）
- 优先级降级需要显式理由；升级标记为潜在范围蔓延
- 核心场景不允许全部映射到周边功能点

## 任务树与上下文管理

对于超出结构复杂度阈值的复杂任务，管线支持模块化拆分：

- **拆分触发**：大纲中模块数 ≥ 3 或故事/组件数 ≥ 5
- **执行顺序**：先写 index.md（共享契约）→ 模块并行 → 合并 Ralph 循环
- **上下文读取**：5 级递增式协议，带实用终止条件
- **文档版本管理**：版本头 + 变更追踪
- **状态追踪**：`tree.md` + 回滚处理

详见 `task-tree.md`（仅在触发拆分时加载）。

## 渐进式披露

每个阶段只加载对应的文件，不要一次性加载所有阶段文件。

| 文件 | 内容 |
|------|------|
| `SKILL.md` | 概览、触发词、关卡规则、拆分判定 |
| `ralph-review-loop.md` | 共享审核协议（含决策流程图） |
| `task-tree.md` | 任务拆解、上下文管理、版本管理（仅在 SPLIT=true 时加载） |
| `phase-1-product-design.md` | 深度访谈收集需求、core/secondary 分类 |
| `phase-2-technical-solution.md` | Planner→Architect→Critic 共识架构、key/peripheral 分类 |
| `phase-3-test-plan.md` | 测试策略含覆盖矩阵和优先级一致性校验 |
| `phase-4-test-code.md` | 先写全部测试（必须可编译，必须运行时失败） |
| `phase-5-business-code.md` | 实现最小代码使测试通过，然后重构 |
| `phase-6-pre-release-testing.md` | 预发布测试、追问协议、回滚路径、发布关卡验证（管线验证闭环） |
| `phase-6-root-cause-investigation.md` | 缺陷根因调查（分层定位、5-Why、修复验证）+ 常见 bug 模式。仅在阶段 6 子阶段失败时加载。 |

## 触发方式

在 AI 编码工具中使用自然语言触发：

- `tdd pipeline <功能描述>`
- `tdd`
- `先写测试`
- `write tests first`
- `test-driven`
- `red green refactor`
- `pre-release`
- `release test`
- `ship it`
- `上线前测试` / `发版前检查` / `回归测试`
- `产品设计` / `技术方案` / `测试方案` / `测试驱动`

## 安装方法

将所有文件复制到 Claude Code 的 skills 目录：

```bash
# macOS/Linux
cp -r tdd-pipeline ~/.claude/skills/

# 验证
ls ~/.claude/skills/tdd-pipeline/
```

或直接克隆：

```bash
git clone <仓库地址> ~/.claude/skills/tdd-pipeline
```

无需构建步骤或依赖安装 —— skills 由 Claude Code skill 系统按需加载。

## 文件结构

```
tdd-pipeline/
├── SKILL.md                        ← 入口文件（渐进式披露中心）
├── ralph-review-loop.md            ← 共享审核协议（仅阶段 1–5）
├── task-tree.md                    ← 仅在复杂任务时加载
├── phase-1-product-design.md
├── phase-2-technical-solution.md
├── phase-3-test-plan.md
├── phase-4-test-code.md
├── phase-5-business-code.md
├── phase-6-pre-release-testing.md  ← 阶段 6 始终加载
├── phase-6-root-cause-investigation.md  ← 仅在子阶段失败时加载
└── README.md
```

## 核心规则

- **TDD 不可妥协**：测试代码不存在且未失败时，禁止编写业务代码
- **阶段 1–5：Ralph 循环关卡** — 零 M 级及以上（C/H/M）问题方可通过
- **阶段 6：验证闭环** — 子阶段 gate + 追问协议 + 用户 go/no-go（非 Ralph 循环）
- **测试即契约** — 业务代码只是让测试通过的实现细节
- **优先级驱动深度**：core/key 全面测试；secondary/peripheral 基本覆盖
- **跨阶段一致性**：优先级分类必须可追溯、有理由地贯穿所有阶段
- **阶段 6 失败 → 回滚**：根因决定回滚目标；修复后全量重跑阶段 6

## 许可证

MIT
