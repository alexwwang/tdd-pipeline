# Code Review (Phase 4–5)

This file is loaded ONLY during code reviews (Phases 4–5). For design reviews, load `review-design.md` instead.

## Code Review Checklist

The reviewer must verify each item below. When defects are found, provide constructive suggestions per `ralph-review-loop.md` §Reviewer Output Requirements. When substantive strategic concerns exist, provide critical opinions.

- [ ] **Test quality**: Completeness, edge cases, descriptive names, one assertion per test where practical
- [ ] **Code quality**: Clean code, no duplication, proper abstractions, follows language idioms
- [ ] **TDD compliance**: No untested code; no code exists without a failing test justifying it
- [ ] **Correctness**: Logic errors, missing cases, broken invariants
- [ ] **Completeness**: Dead code remnants, missed cleanups, orphaned references (imports/columns/variables)
- [ ] **Consistency**: Naming inconsistencies, mixed conventions, partial refactoring
- [ ] **Security**: Data exposure, injection risks, missing validation
- [ ] **Performance**: Unnecessary computation, resource leaks, N+1 patterns
- [ ] **Maintainability**: Readability, documentation gaps, confusing abstractions
- [ ] **Backward compatibility**: Public API removal/signature changes safe?
- [ ] **Import cleanup**: No residual imports/exports of deleted symbols

### Phase 4 Specifics

- Are ALL tests genuinely failing? No premature implementation?
- Do tests import non-existent modules/functions as intended?
- Are stubs truly minimal and not passing tests?

### Phase 5 Specifics

- Is refactoring clean? Any regressions introduced?
- Is the minimum code principle respected? No gold-plating?
- Does every line of business code have test coverage?

## Single-Pass Review

Use the standard reviewer prompt from `ralph-review-loop.md` §Reviewer Selection, injecting the checklist above as the review scope.

## Dual-Pass Recall Prompt

If using dual-pass mode, inject this as the Recall subagent prompt:

```
你是一位独立审查专家（Recall Pass）。你的职责是尽可能全面地发现所有潜在问题。
后续会有另一位专家对你的发现进行精确性过滤。

**核心原则：宁可多报，不可遗漏。**

- 如果你不确定某个问题是否真实存在，仍然报告它，但标记较低的 confidence
- 如果你认为某处可能有问题但缺乏充分证据，仍然报告它
- 不要因为"可能不是问题"就跳过任何可疑之处

## 审查维度
逐一检查以下维度：
1. 正确性：逻辑错误、遗漏情况、不变量破坏
2. 完整性：死代码残留、遗漏的清理、孤立的引用（import/列名/变量）
3. 一致性：命名不一致、混合约定、部分重构
4. 安全性：数据暴露、注入风险、缺少验证
5. 性能：不必要的计算、资源泄漏、N+1 模式
6. 测试质量：缺失断言、弱化覆盖、不应删除的测试
7. 可维护性：可读性、文档缺口、令人困惑的抽象
8. TDD 合规：{Phase 4: 所有测试是否确实失败？无过早实现？} / {Phase 5: 最小实现原则？无过度工程？}
9. 向后兼容：公开 API 删除/签名变更是否安全？
10. import 清理：是否有残留的 import/导出

## 交付物
{DELIVERABLE}

## 前序阶段输出（用于跨阶段一致性检查）
{PRIOR_PHASE_OUTPUTS}

## 争议事项（来自上一轮）
{CONTESTED_ISSUES}

## 输出格式
对每个发现，严格按以下 JSON 格式输出：
{
  "id": "F-{序号}",
  "severity": "C|H|M|L|I",
  "category": "{分类}",
  "location": "{文件:行号或描述}",
  "description": "{问题描述}",
  "evidence": "{具体证据引用}",
  "suggestion": "{修复建议}",
  "confidence": 0.0-1.0
}
```

## Fact-Gathering for Precision Filter (Code Review)

Between Recall and Precision passes, the main agent gathers these facts from the codebase:

```
facts_to_gather = [
  grep for residual references to deleted symbols (classes, functions, variables, columns)
  cat the actual implementation of suspect functions (not the diff, the full code)
  read config files referenced in findings (to verify staleness)
  check __all__ / exports for orphaned references
]
```

Then inject gathered facts into the Precision Filter prompt from `review-precision-filter.md`.
