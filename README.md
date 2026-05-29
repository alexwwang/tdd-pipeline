# TDD Pipeline Skill

[English](#english) · [中文](#中文)

---

<a id="english"></a>

## Overview

A rigorous 8-phase TDD development workflow for AI-assisted coding. Phases 1–5 enforce **Red-Green-Refactor** at the pipeline level with mandatory Ralph-loop review. Phase 6 is the pipeline's **validation closure** — systematic pre-release testing with bug root cause investigation (追问) and rollback paths. Phase 7 is an **incremental system audit** — a second-pass quality review with 16-pattern catalog, integration pair discovery, and execution-order analysis. Phase 8 is **functional acceptance** — requirements traceability, AC-level verification, and release decision.

```
Product Design → Technical Solution → Test Plan → Test Code → Business Code → Pre-Release Testing → System Quality Audit → Acceptance Testing
    产品设计    →     技术方案      →   测试方案  →  测试代码  →  业务代码   →    预发布测试      →     系统质量审计     →     功能验收
```

> **Core Principle**: If you cannot write a failing test for it, you do not understand it well enough to build it.
>
> **Pace Principle**: 慢就是快，欲速不达 (Slow is fast; haste makes waste). The Ralph loop's review rounds are not overhead — they are where quality is built. Every shortcut through a gate saves minutes now but costs hours in debugging later.

## The 8 Phases

Phases 1–5 are **creation phases** with Ralph-loop review. Phase 6 is the **validation closure** with its own quality mechanisms. Phase 7 is an **incremental system audit** that runs after Phase 6 completes, finding issues missed by single-pass analysis. Phase 8 is **functional acceptance** that traces requirements to implementation, verifying every AC has evidence before release.

| Phase | Deliverable | Quality Mechanism |
|-------|-------------|-------------------|
| 1. Product Design | Requirements Document (with core/secondary priorities) | Ralph design review |
| 2. Technical Solution | Technical Design Document (with key/peripheral priorities) | Ralph design review |
| 3. Test Plan | Test Plan Document (with coverage matrices + consistency checks) | Ralph design review |
| 4. Test Code | Test Files (all failing) | Ralph code review |
| 5. Business Code | Working Business Code | Ralph code review |
| 6. Pre-Release Testing | Release Gate Checklist (with evidence) | Testing flow + 追问 (root cause investigation) |
| 7. System Quality Audit | Incremental audit report (issues Phase 6 missed) | 16-pattern catalog + pair discovery + execution-order analysis |
| 8. Acceptance Testing | Acceptance report (requirements traceability + coverage) | AC-level verification + independent check + user go/no-go |

## Ralph Loop Review (Phases 1–5)

Phases 1–5 each end with a mandatory review loop:

- **Stop condition**: 2 consecutive rounds with zero **new** C/H/M findings (P and L do not reset the counter; known issues excluded). Issues already listed in the Known Issues document do not count as new. Can trigger at any round ≥ 2. No round cap.
- **Persistent issues escalation**: if the same C/H/M findings recur despite fixes → model evaluates root cause → escalate to user for clarification OR rollback to prior phase
- **Known Issues management**: deferred findings are recorded in a KI document; re-evaluated every 3 rounds and at loop end (valid, fixed, severity change, false positive)
- **Independent reviewer** subagent for each round

Phase 6 does NOT use Ralph loop. See Phase 6 row above and `phase-6-pre-release-testing.md` Part 4 for its quality mechanisms: sub-phase gates and 追问 (root cause investigation) protocol. Phase 7 runs after Phase 6 completes — see `phase-7-system-quality-audit.md` for details. Phase 8 runs after Phase 7 completes — see `phase-8-acceptance-testing.md` for requirements traceability and release decision.

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
| `ralph-gpav.md` | GPAV submission protocol, RPS scanner (loaded when Watchdog active) |
| `ralph-continuation.md` | Rounds 2+ protocol: evaluation, stop conditions, flowchart |
| `ralph-contested.md` | Contested issue protocol (loaded when C/H/M is REJECTed) |
| `ralph-examples.md` | Worked examples: stop conditions, contested issues (loaded on demand) |
| `ralph-log-template.md` | Review log format template (loaded at loop start) |
| `review-design.md` | Design review checklist + Recall prompt (Phase 1–3) |
| `review-code.md` | Code review checklist + Recall prompt (Phase 4–5) |
| `review-precision-filter.md` | Dual-pass Precision Filter prompt + aggregation (shared) |
| `severity-migration.md` | Severity classification migration guide (pre-v0.13 → v0.13). Loaded on demand. |
| `task-tree.md` | Task decomposition, context management, versioning (loaded only when SPLIT=true) |
| `phase-1-product-design.md` | Requirements gathering via deep-interview, core/secondary classification |
| `phase-2-technical-solution.md` | Architecture via Planner→Architect→Critic consensus, key/peripheral classification |
| `phase-3-test-plan.md` | Test strategy with coverage matrices and priority consistency validation |
| `phase-3-edge-reference.md` | Edge case discovery and coverage guidelines for test planning. Loaded with Phase 3 review only (not during deliverable creation). |
| `phase-4-test-code.md` | Write all tests first (must compile, must fail at runtime) |
| `phase-4-test-infrastructure-checklist.md` | Test infrastructure validation checklist. Applied after test code, before Ralph review. |
| `phase-5-business-code.md` | Implement minimum code to pass tests, then refactor |
| `phase-6-pre-release-testing.md` | Pre-release testing, 追问 (root cause investigation) protocol, rollback paths, release gate verification |
| `phase-6-root-cause-investigation.md` | Bug root cause investigation (Layer Isolation, 5-Why, Fix Verification) + common bug patterns. Loaded only when a Phase 6 sub-phase fails. |
| `phase-7-system-quality-audit.md` | Incremental system audit: 16-pattern catalog, integration pair discovery, execution-order analysis. Loaded after Phase 6 completes. |
| `phase-8-acceptance-testing.md` | Functional acceptance: requirements traceability, AC-level verification, independent check, and release decision. Loaded after Phase 7 completes. |

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
- **Phases 1–5: Ralph loop gate** — zero defect-layer (C/H/M) issues to pass
- **Phase 6: validation closure** — sub-phase gates + 追问 (root cause investigation) (not Ralph loop)
- **Phase 7: system audit** — incremental second pass after Phase 6, 16-pattern catalog + pair discovery + execution-order analysis
- **Phase 8: acceptance** — functional verification after Phase 7, requirements traceability + AC coverage + user go/no-go
- **Tests define the contract** — business code is just the implementation
- **Priority drives depth**: core/key items get comprehensive testing; secondary/peripheral get basic coverage
- **Cross-phase consistency**: priority classifications must be traceable and justified across all phases
- **Phase 6 failure → rollback**: root cause determines rollback target; full Phase 6 rerun after any fix, then continue through Phase 7→8
- **Dual severity systems**: Ralph loop (Phases 1–5) uses C/H/M/P/L/I with gate blocking on M+; Pipeline (Phases 6–8) uses C/H/M/P/L with gate blocking on C/H only. See `SKILL.md` for full details.

## Dual-Pass Review Mode

The Ralph loop uses a **two-pass Recall/Precision pipeline** in all rounds. Single-pass mode was removed — `ralph-review-loop.md` now mandates dual-pass for every round ("⛔ Single-pass is forbidden"). This mode is based on the key insight that **splitting recall and precision into separate prompts outperforms a single complex prompt** ([G-Research, "Building a code review tool: The LLM patterns that actually work"](https://www.gresearch.com/news/building-a-code-review-tool-the-llm-patterns-that-actually-work/)).

### How it works

| Pass | Goal | Subagent | Output |
|------|------|----------|--------|
| **1 — Recall** | Maximize coverage — report everything suspicious, even if uncertain | Independent reviewer (宽松 prompt) | `raw_findings` (may include false positives) |
| **2 — Precision** | Verify each finding against codebase facts | Precision filter (严格 prompt + verifiable facts) | `CONFIRM` / `DOWNGRADE` / `REJECT` per finding |

Between passes, the main agent **gathers verifiable facts** from the codebase (grep for residual references, read suspect implementations, check exports) to inject into the Precision prompt. This fact-gathering step is critical — without it, the filter relies solely on LLM judgment.

### When to use

| Mode | When | Cost |
|------|------|------|
| **Dual-pass** (all rounds) | All rounds — single-pass is forbidden per `ralph-review-loop.md` | 2× LLM calls per round, but fewer total rounds |
| ~~Single-pass~~ | ~~Removed — dual-pass is now mandatory~~ | ~~1× LLM call~~ |

### Verified results

The dual-pass mode was validated against a real Phase 7 code review (16-file diff, multi-component refactoring):

| Metric | Single-pass | Dual-pass | Change |
|--------|-------------|-----------|--------|
| Raw findings per round | 16 | 25 → 12 (after filter) | +56% recall, −52% after filter |
| True H-level findings | 2 | 2 | Same, found with higher confidence |
| H-level false positives | 1 | 0 | −100% |
| Effective finding rate | ~75% | ~92% | +17% |
| LLM calls per round | 1 | 2 | +100% |
| Expected Ralph loop rounds | 5–7 | 3–5 | −30–40% |

The dual-pass also discovered an additional real bug that single-pass missed: residual column references from a prior schema migration.

**Key takeaway**: The value is not cost reduction but quality improvement — preventing H-level false positives from triggering wasted fix rounds. One wasted round costs the same as an entire dual-pass round.

### File reference

| Phase | Load | Contains |
|-------|------|----------|
| 1–3 (Design) | `review-design.md` | Checklist + Recall prompt + fact-gather guide |
| 4–5 (Code) | `review-code.md` | Checklist + Recall prompt + fact-gather guide |
| Precision (shared) | `review-precision-filter.md` | Precision Filter prompt + aggregation logic |

## Installation

Copy all skill files to your AI coding tool's skills directory:

```bash
# Claude Code (macOS/Linux)
cp skill/*.md ~/.claude/skills/tdd-pipeline/

# OpenCode (macOS/Linux)
cp skill/*.md ~/.config/opencode/skills/tdd-pipeline/

# Verify
ls ~/.claude/skills/tdd-pipeline/
ls ~/.config/opencode/skills/tdd-pipeline/
```

Or clone and symlink:

```bash
git clone <repo-url> ~/tdd-pipeline
mkdir -p ~/.claude/skills/tdd-pipeline
ln -s ~/tdd-pipeline/skill/*.md ~/.claude/skills/tdd-pipeline/
mkdir -p ~/.config/opencode/skills/tdd-pipeline
ln -s ~/tdd-pipeline/skill/*.md ~/.config/opencode/skills/tdd-pipeline/
```

**Note**: Skill files must be synced to **both** deployment targets (`~/.claude/skills/tdd-pipeline/` and `~/.config/opencode/skills/tdd-pipeline/`). The OpenCode path is required if you use OpenCode as your AI coding tool.

### Double-Blind Experiment Skill

The `experiment/` directory contains a separate skill for rigorous A/B comparison of prompt/skill variants. Install independently:

```bash
cp experiment/*.md ~/.claude/skills/double-blind-experiment/

# Verify
ls ~/.claude/skills/double-blind-experiment/
```

Usage: `double-blind experiment` or `compare variants A/B`. The skill auto-selects a tier based on stakes:
- **Tier 1 (Quick Screen)**: internal sanity check, ≥3 scenarios, 1 scorer
- **Tier 2 (Standard)**: adoption decision, ≥4 scenarios, full checklist
- **Tier 3 (Decision Grade)**: external claim, ≥5 scenarios, ≥2 scorers, pre-registration

No build step or dependency installation required — skills are loaded on demand by the Claude Code skill system.

## File Structure

```
tdd-pipeline/
├── skill/                                ← deploy to ~/.claude/skills/tdd-pipeline/
│   ├── SKILL.md                          ← entry point (progressive disclosure hub)
│   ├── ralph-review-loop.md              ← shared review protocol (Phases 1–5 only)
│   ├── ralph-gpav.md                     ← GPAV submission protocol (Watchdog active)
│   ├── ralph-continuation.md             ← Rounds 2+: evaluation, stop conditions, flowchart
│   ├── ralph-contested.md                ← contested issue protocol (on REJECT C/H/M)
│   ├── ralph-examples.md                 ← worked examples (contested issues, stop scenarios)
│   ├── ralph-log-template.md             ← review log format template
│   ├── review-design.md                  ← design review checklist + Recall prompt (Phase 1–3)
│   ├── review-code.md                    ← code review checklist + Recall prompt (Phase 4–5)
│   ├── review-precision-filter.md        ← dual-pass Precision Filter prompt (shared)
│   ├── task-tree.md                      ← loaded only for complex tasks
│   ├── phase-1-product-design.md
│   ├── phase-2-technical-solution.md
│   ├── phase-3-test-plan.md
│   ├── phase-3-edge-reference.md
│   ├── phase-4-test-code.md
│   ├── phase-4-test-infrastructure-checklist.md
│   ├── phase-5-business-code.md
│   ├── phase-6-pre-release-testing.md    ← always loaded for Phase 6
│   ├── phase-6-root-cause-investigation.md  ← loaded only on sub-phase failure
│   ├── phase-7-system-quality-audit.md   ← loaded after Phase 6 completes
│   └── phase-8-acceptance-testing.md    ← loaded after Phase 7 completes
├── experiment/                           ← separate skill (double-blind experiment)
│   ├── SKILL.md                          ← entry point (progressive disclosure)
│   ├── tier-1.md                         ← Tier 1: quick screen
│   ├── tier-2.md                         ← Tier 2: standard (3-scenario + magnitude filter)
│   └── tier-3.md                         ← Tier 3: full (multi-scenario + regression)
├── README.md
└── LICENSE
```

## License

MIT

---

<a id="中文"></a>

## 概述

一个严格的 8 阶段 TDD 开发工作流，专为 AI 辅助编程设计。阶段 1–5 在管线级别强制执行 **Red-Green-Refactor**，每个阶段结束后有 Ralph 循环审核。阶段 6 是管线的**验证闭环** —— 系统化预发布测试、缺陷根因追问（追问方法）和回滚路径。阶段 7 是**增量系统审计** —— 第二轮质量审核，包含 16 模式目录、集成对发现和执行顺序分析。阶段 8 是**功能验收** —— 需求追溯、AC 级别验证和发布决策。

> **核心原则**：如果你无法为某个功能写出一个失败的测试，说明你对它的理解还不够深入。
>
> **节奏原则**：慢就是快，欲速不达。Ralph 循环的审查轮次不是开销——质量就是在这里构建的。每个跨越关卡的捷径省下了几分钟，但日后调试要花几小时。

## 8 个阶段

阶段 1–5 是**创作阶段**，使用 Ralph 循环审核。阶段 6 是**验证闭环**，使用独立的质量机制。阶段 7 是**增量系统审计**，在阶段 6 完成后运行，发现单次审核遗漏的问题。阶段 8 是**功能验收**，在阶段 7 完成后运行，追溯需求到实现，验证每个 AC 有证据支持。

| 阶段 | 交付物 | 质量机制 |
|------|--------|----------|
| 1. 产品设计 | 需求文档（含 core/secondary 优先级分类） | Ralph 设计审核 |
| 2. 技术方案 | 技术设计文档（含 key/peripheral 优先级分类） | Ralph 设计审核 |
| 3. 测试方案 | 测试计划文档（含覆盖矩阵 + 一致性校验） | Ralph 设计审核 |
| 4. 测试代码 | 测试文件（全部失败） | Ralph 代码审核 |
| 5. 业务代码 | 可工作的实现代码 | Ralph 代码审核 |
| 6. 预发布测试 | 发布关卡检查清单（含证据） | 测试流程 + 追问 |
| 7. 系统质量审计 | 增量审计报告（阶段 6 遗漏的问题） | 16 模式目录 + 对发现 + 执行顺序分析 |
| 8. 功能验收 | 验收报告（需求追溯 + 覆盖度） | AC 级别验证 + 独立检查 + 用户 go/no-go |

## Ralph 循环审核（阶段 1–5）

阶段 1–5 每个结束后启动强制审核循环：

- **唯一停止条件**：连续两轮零缺陷层新发现（C/H/M 均无新增，P/L 和已知问题不重置计数器）。已知问题文档中已列示的问题不算新发现。可在任意 ≥ 2 轮时触发。无轮次上限。
- **持续问题升级**：同类 C/H/M 问题反复出现且修复无效时 → 模型评估根因 → 向用户确认关键信息或回退到上一阶段排查
- **已知问题管理**：未当场修复的问题写入 Known Issues 文档；每 3 轮及循环结束时对所有 KI 做评估（真实性、失效、升降级）
- 每轮由**独立审核 subagent** 执行

阶段 6 **不使用** Ralph 循环。详见上方阶段 6 行及 `phase-6-pre-release-testing.md` Part 4。阶段 7 在阶段 6 完成后运行，详见 `phase-7-system-quality-audit.md`。阶段 8 在阶段 7 完成后运行，详见 `phase-8-acceptance-testing.md`。

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
| `ralph-gpav.md` | GPAV 提交协议、RPS 扫描器（Watchdog 激活时加载） |
| `ralph-continuation.md` | 第 2 轮起协议：评估、停止条件、决策流程图 |
| `ralph-contested.md` | 争议问题协议（REJECT C/H/M 时加载） |
| `ralph-examples.md` | 示例集：停止条件、争议问题（按需加载） |
| `ralph-log-template.md` | 审查日志格式模板（循环启动时加载） |
| `review-design.md` | 方案审查清单 + Recall 提示词（阶段 1–3） |
| `review-code.md` | 代码审查清单 + Recall 提示词（阶段 4–5） |
| `review-precision-filter.md` | 双轮 Precision Filter 提示词 + 聚合逻辑（共用） |
| `severity-migration.md` | 严重级别迁移指南（pre-v0.13 → v0.13）。按需加载。 |
| `task-tree.md` | 任务拆解、上下文管理、版本管理（仅在 SPLIT=true 时加载） |
| `phase-1-product-design.md` | 深度访谈收集需求、core/secondary 分类 |
| `phase-2-technical-solution.md` | Planner→Architect→Critic 共识架构、key/peripheral 分类 |
| `phase-3-test-plan.md` | 测试策略含覆盖矩阵和优先级一致性校验 |
| `phase-3-edge-reference.md` | 边界情况发现与覆盖指南。仅 Phase 3 审查期间加载（创建交付物期间不加载）。 |
| `phase-4-test-code.md` | 先写全部测试（必须可编译，必须运行时失败） |
| `phase-4-test-infrastructure-checklist.md` | 测试基础设施验证清单。测试代码完成后、Ralph 审查前使用。 |
| `phase-5-business-code.md` | 实现最小代码使测试通过，然后重构 |
| `phase-6-pre-release-testing.md` | 预发布测试、追问协议、回滚路径、发布关卡验证（管线验证闭环） |
| `phase-6-root-cause-investigation.md` | 缺陷根因调查（分层定位、5-Why、修复验证）+ 常见 bug 模式。仅在阶段 6 子阶段失败时加载。 |
| `phase-7-system-quality-audit.md` | 增量系统审计：16 模式目录、集成对发现、执行顺序分析。阶段 6 完成后加载。 |
| `phase-8-acceptance-testing.md` | 功能验收：需求追溯、AC 级别验证、独立检查、发布决策。阶段 7 完成后加载。 |

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
- `go/no-go`
- `ready to deploy`
- `上线前测试` / `发版前检查` / `回归测试`
- `产品设计` / `技术方案` / `测试方案` / `测试驱动`

## 双轮审查模式

Ralph 循环在所有轮次中使用**双轮 Recall/Precision 审查**。单轮模式已移除 — `ralph-review-loop.md` 现在强制要求每轮使用双轮审查（"⛔ 禁止单轮审查"）。该方法的核心洞见：**将召回率和精确率拆分到两个独立的 prompt 中，效果优于单个复杂 prompt**（[G-Research, "Building a code review tool: The LLM patterns that actually work"](https://www.gresearch.com/news/building-a-code-review-tool-the-llm-patterns-that-actually-work/)）。

### 工作原理

| 轮次 | 目标 | 子代理 | 输出 |
|------|------|--------|------|
| **1 — Recall** | 最大化覆盖 — 报告所有可疑之处，即使不确定 | 独立审查员（宽松 prompt） | `raw_findings`（可能包含误报） |
| **2 — Precision** | 对照代码库事实验证每个发现 | 精确率过滤器（严格 prompt + 可验证事实） | 每个发现给出 `CONFIRM` / `DOWNGRADE` / `REJECT` |

两轮之间，主代理从代码库中**收集可验证事实**（grep 残留引用、读取可疑实现、检查导出），注入 Precision prompt。这一步至关重要——没有事实支撑，过滤器仅凭 LLM 判断不可靠。

### 使用场景

| 模式 | 适用场景 | 开销 |
|------|----------|------|
| **双轮**（所有轮次） | 所有轮次 — 单轮审查已禁止，见 `ralph-review-loop.md` | 每轮 2 次 LLM 调用，但总轮数更少 |
| ~~单轮~~ | ~~已移除 — 双轮审查现为强制要求~~ | ~~每轮 1 次 LLM 调用~~ |

### 验证结果

双轮模式经真实阶段 7 代码审查验证（16 文件 diff，多组件重构）：

| 指标 | 单轮 | 双轮 | 变化 |
|------|------|------|------|
| 每轮原始发现数 | 16 | 25 → 12（过滤后） | 召回率 +56%，过滤后 −52% |
| 真实 H 级发现 | 2 | 2 | 相同，但置信度更高 |
| H 级误报 | 1 | 0 | −100% |
| 有效发现率 | ~75% | ~92% | +17% |
| 每轮 LLM 调用 | 1 | 2 | +100% |
| 预期 Ralph 循环轮数 | 5–7 | 3–5 | −30–40% |

双轮还发现了单轮遗漏的一个真实 bug：schema 迁移后残留的列引用。

**核心结论**：价值不在降低成本，而在提升质量——防止 H 级误报触发浪费的修复轮次。一轮浪费的成本等于一整个双轮的开销。

### 文件索引

| 阶段 | 加载文件 | 内容 |
|------|----------|------|
| 1–3（方案审查） | `review-design.md` | 审查清单 + Recall 提示词 + 事实收集指南 |
| 4–5（代码审查） | `review-code.md` | 审查清单 + Recall 提示词 + 事实收集指南 |
| Precision（共用） | `review-precision-filter.md` | Precision Filter 提示词 + 聚合逻辑 |

## 安装方法

将 skill 文件复制到 AI 编码工具的 skills 目录：

```bash
# Claude Code（macOS/Linux）
cp skill/*.md ~/.claude/skills/tdd-pipeline/

# OpenCode（macOS/Linux）
cp skill/*.md ~/.config/opencode/skills/tdd-pipeline/

# 验证
ls ~/.claude/skills/tdd-pipeline/
ls ~/.config/opencode/skills/tdd-pipeline/
```

或克隆后符号链接：

```bash
git clone <仓库地址> ~/tdd-pipeline
mkdir -p ~/.claude/skills/tdd-pipeline
ln -s ~/tdd-pipeline/skill/*.md ~/.claude/skills/tdd-pipeline/
mkdir -p ~/.config/opencode/skills/tdd-pipeline
ln -s ~/tdd-pipeline/skill/*.md ~/.config/opencode/skills/tdd-pipeline/
```

**注意**：Skill 文件必须同步到**两个**部署目标（`~/.claude/skills/tdd-pipeline/` 和 `~/.config/opencode/skills/tdd-pipeline/`）。如果你使用 OpenCode 作为 AI 编码工具，必须配置 OpenCode 路径。

### 双盲实验技能

`experiment/` 目录包含独立技能，用于严格对比 prompt/skill 变体。需独立安装：

```bash
cp experiment/*.md ~/.claude/skills/double-blind-experiment/

# 验证
ls ~/.claude/skills/double-blind-experiment/
```

使用：`double-blind experiment` 或 `compare variants A/B`。技能根据利害程度自动选择层级：
- **Tier 1（快速筛选）**：内部验证，≥3 场景，1 评分者
- **Tier 2（标准）**：采纳决策，≥4 场景，完整检查清单
- **Tier 3（决策级）**：外部声明，≥5 场景，≥2 评分者，预注册

无需构建步骤或依赖安装 —— skills 由 Claude Code skill 系统按需加载。

## 文件结构

```
tdd-pipeline/
├── skill/                                ← 部署到 ~/.claude/skills/tdd-pipeline/
│   ├── SKILL.md                          ← 入口文件（渐进式披露中心）
│   ├── ralph-review-loop.md              ← 共享审核协议（仅阶段 1–5）
│   ├── ralph-gpav.md                     ← GPAV 提交协议（Watchdog 激活时）
│   ├── ralph-continuation.md             ← 第 2 轮起：评估、停止条件、流程图
│   ├── ralph-contested.md                ← 争议问题协议（REJECT C/H/M 时）
│   ├── ralph-examples.md                 ← 示例集（争议问题、停止条件）
│   ├── ralph-log-template.md             ← 审查日志格式模板
│   ├── review-design.md                  ← 方案审查清单 + Recall 提示词（阶段 1–3）
│   ├── review-code.md                    ← 代码审查清单 + Recall 提示词（阶段 4–5）
│   ├── review-precision-filter.md        ← 双轮 Precision Filter 提示词（共用）
│   ├── task-tree.md                      ← 仅在复杂任务时加载
│   ├── phase-1-product-design.md
│   ├── phase-2-technical-solution.md
│   ├── phase-3-test-plan.md
│   ├── phase-3-edge-reference.md
│   ├── phase-4-test-code.md
│   ├── phase-4-test-infrastructure-checklist.md
│   ├── phase-5-business-code.md
│   ├── phase-6-pre-release-testing.md    ← 阶段 6 始终加载
│   ├── phase-6-root-cause-investigation.md  ← 仅在子阶段失败时加载
│   ├── phase-7-system-quality-audit.md   ← 阶段 6 完成后加载
│   └── phase-8-acceptance-testing.md    ← 阶段 7 完成后加载
├── experiment/                           ← 独立 skill（双盲实验）
│   ├── SKILL.md                          ← 入口文件（渐进式披露）
│   ├── tier-1.md                         ← Tier 1：快速筛选
│   ├── tier-2.md                         ← Tier 2：标准（3 场景+幅度筛选）
│   └── tier-3.md                         ← Tier 3：完整（多场景+回归分析）
├── README.md
└── LICENSE
```

## 核心规则

- **TDD 不可妥协**：测试代码不存在且未失败时，禁止编写业务代码
- **阶段 1–5：Ralph 循环关卡** — 零缺陷层（C/H/M）问题方可通过
- **阶段 6：验证闭环** — 子阶段 gate + 追问协议（非 Ralph 循环）
- **阶段 7：系统审计** — 阶段 6 完成后的增量第二轮审核，16 模式目录 + 对发现 + 执行顺序分析
- **阶段 8：功能验收** — 阶段 7 完成后的功能验证，需求追溯 + AC 覆盖度 + 用户 go/no-go
- **测试即契约** — 业务代码只是让测试通过的实现细节
- **优先级驱动深度**：core/key 全面测试；secondary/peripheral 基本覆盖
- **跨阶段一致性**：优先级分类必须可追溯、有理由地贯穿所有阶段
- **阶段 6 失败 → 回滚**：根因决定回滚目标；修复后全量重跑阶段 6，然后继续执行阶段 7→8
- **双严重级别系统**：Ralph 循环（阶段 1–5）使用 C/H/M/P/L/I，gate 阻塞于 M+；Pipeline（阶段 6–8）使用 C/H/M/P/L，gate 阻塞于 C/H。详见 `SKILL.md`。

## 许可证

MIT
