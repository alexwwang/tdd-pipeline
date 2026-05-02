# Release Notes — v0.4.0

## Phase 6: Pre-Release Testing as Pipeline Validation Closure

将预发布测试整合为 TDD Pipeline 的第 6 阶段——管线验证闭环。Phase 6 不是一个"创作阶段"，而是对 Phase 1–5 全部产出的系统性验证。

### 新增文件

| 文件 | 用途 |
|------|------|
| `skill/phase-6-pre-release-testing.md` | Phase 6 主文件（始终加载）：子阶段测试流程、Integration Gap Checklist、Pipeline Integration |
| `skill/phase-6-root-cause-investigation.md` | 按需加载（子阶段失败时）：Layer Isolation、5-Why Drill、T1-T3 终止条件、HC1-HC4 硬约束、独立确认、V1-V5 修复验证、6 种常见 bug 模式 |
| `phase-6-integration-plan.md` | 设计文档：三轮讨论的完整方案，含定位、流程、追问协议、回滚路径、决策记录 |
| `pre-release-testing.md` | 原始技能文件（设计参考） |

### 变更文件

| 文件 | 变更 |
|------|------|
| `skill/SKILL.md` | 5→6 阶段管线；Phase 6 定义为验证闭环（非 Ralph loop）；新增 Rollback from Phase 6 章节；更新 Gate Rules 区分创作/验证阶段；Anti-Patterns 新增 Phase 6 条目 |
| `skill/ralph-review-loop.md` | 明确 scope 为 Phase 1–5 only；Phase 6 使用独立的验证流程 |
| `skill/README.md` | EN+ZH 同步更新：概述、阶段表、Ralph 章节、Key Rules、文件结构、渐进式披露表 |
| `.gitignore` | 新增 `skill-backup/`、`.omc/` |

### 核心设计决策

1. **Phase 6 不使用 Ralph loop** — 子阶段 gate 是客观 pass/fail，"能不能发"是用户业务决策
2. **追问方法** 替代 Ralph loop — 发散式诊断 + T1-T3 终止条件 + HC1-HC4 硬约束 + 一次独立确认
3. **回滚路径由追问结论驱动** — 根因层面自动映射到回滚目标（代码 bug→Phase 5，设计缺陷→Phase 2）
4. **渐进式披露拆分** — 主文件 165 行（始终加载），追问文件 224 行（仅失败时加载）

### 文件结构

```
skill/
├── SKILL.md
├── ralph-review-loop.md
├── task-tree.md
├── phase-1-product-design.md
├── phase-2-technical-solution.md
├── phase-3-test-plan.md
├── phase-4-test-code.md
├── phase-5-business-code.md
├── phase-6-pre-release-testing.md      ← NEW
├── phase-6-root-cause-investigation.md ← NEW (on-demand)
└── README.md
```

### 版本号理由

`v0.3.1` → `v0.4.0` (minor)：新增完整的 Phase 6 功能模块，改变管线从 5 阶段到 6 阶段，属于功能级增强。

---

**Full Changelog**: v0.3.1...v0.4.0
