# TDD Pipeline Skill

[English](#english) · [中文](#中文)

---

<a id="english"></a>

## Overview

A rigorous 5-phase TDD development workflow for AI-assisted coding. Enforces **Red-Green-Refactor** at the pipeline level with mandatory Ralph-loop review after every phase.

```
Product Design → Technical Solution → Test Plan → Test Code → Business Code
    产品设计    →     技术方案      →   测试方案  →  测试代码  →  业务代码
```

> **Core Principle**: If you cannot write a failing test for it, you do not understand it well enough to build it.

## The 5 Phases

| Phase | Deliverable | Review |
|-------|-------------|--------|
| 1. Product Design | Requirements Document | Ralph design review |
| 2. Technical Solution | Technical Design Document | Ralph design review |
| 3. Test Plan | Test Plan Document | Ralph design review |
| 4. Test Code | Test Files (all failing) | Ralph code review |
| 5. Business Code | Working Implementation | Ralph code review |

## Ralph Loop Review

Every phase ends with a mandatory review loop:

- **Minimum 5 rounds** (no exceptions)
- **Early stop**: only after **2 consecutive zero-issue rounds** (intermittent zeros do not count)
- **Gate**: zero Critical/High/Major issues to proceed
- **Independent reviewer** subagent for each round

## Progressive Disclosure

At each phase, only load the corresponding phase file. Do not load all files at once.

| File | Content |
|------|---------|
| `SKILL.md` | Overview, triggers, gate rules |
| `ralph-review-loop.md` | Shared review protocol |
| `phase-1-product-design.md` | Requirements gathering via deep-interview |
| `phase-2-technical-solution.md` | Architecture via Planner→Architect→Critic consensus |
| `phase-3-test-plan.md` | Test strategy derived from acceptance criteria |
| `phase-4-test-code.md` | Write all tests first (must all fail) |
| `phase-5-business-code.md` | Implement minimum code to pass tests, then refactor |

## Usage

Use natural language triggers in your AI coding tool:

- `tdd pipeline <feature description>`
- `write tests first`
- `test-driven development`
- `产品设计` / `技术方案` / `测试驱动` / `先写测试`

## Key Rules

- **TDD is non-negotiable**: no business code until tests exist and fail
- **Each phase requires user approval** before proceeding
- **Ralph loop gate**: zero M+ (C/H/M) issues to pass
- **Tests define the contract** — business code is just the implementation

## License

MIT

---

<a id="中文"></a>

## 概述

一个严格的 5 阶段 TDD 开发工作流，专为 AI 辅助编程设计。在管线级别强制执行 **Red-Green-Refactor**，每个阶段结束后必须有 Ralph 循环审核。

> **核心原则**：如果你无法为某个功能写出一个失败的测试，说明你对它的理解还不够深入。

## Ralph 循环审核

每个阶段结束后启动强制审核循环：

- **最少 5 轮**（不可例外）
- **早停条件**：仅当**连续 2 轮零问题**时方可提前结束（间隔的零问题轮次不计）
- **关卡条件**：零 Critical/High/Major 问题方可进入下一阶段
- 每轮由**独立审核 subagent** 执行

## 渐进式披露

每个阶段只加载对应的文件，不要一次性加载所有阶段文件。详见上方文件列表。

## 触发方式

在 AI 编码工具中使用自然语言触发：

- `tdd pipeline <功能描述>`
- `先写测试`
- `测试驱动开发`
- `产品设计` / `技术方案` / `测试方案`

## 核心规则

- **TDD 不可妥协**：测试代码不存在且未失败时，禁止编写业务代码
- **每阶段需用户审批**后方可推进
- **Ralph 循环关卡**：零 M 级及以上（C/H/M）问题方可通过
- **测试即契约** — 业务代码只是让测试通过的实现细节

## 许可证

MIT
