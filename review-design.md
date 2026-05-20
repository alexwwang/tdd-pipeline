# Design Review (Phase 1–3)

This file is loaded ONLY during design reviews (Phases 1–3). For code reviews, load `review-code.md` instead.

## Design Review Checklist

The reviewer must verify each item below. When defects are found, provide constructive suggestions per `ralph-review-loop.md` §Reviewer Output Requirements. When substantive strategic concerns exist, provide critical opinions.

- [ ] **Completeness**: Does the deliverable cover all requirements from prior phases?
- [ ] **Consistency**: No contradictions with prior phase outputs
- [ ] **Clarity**: Unambiguous, explicit, no hand-waving
- [ ] **Edge cases**: Error paths, boundaries, and failure modes considered
- [ ] **Feasibility**: Can this actually be built and tested?
- [ ] **Traceability**: Every acceptance criterion maps to a test or design element
- [ ] **Backward compatibility**: API changes won't break callers
- [ ] **Security**: Data exposure, injection risks, missing validation

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
1. 完整性：是否覆盖了前序阶段的所有需求？
2. 一致性：是否与前序阶段输出矛盾？
3. 清晰性：是否无歧义、明确、无 hand-waving？
4. 边界情况：错误路径、边界条件、失败模式是否考虑？
5. 可行性：这能实际构建和测试吗？
6. 可追溯性：每个验收标准是否映射到测试或设计元素？
7. 向后兼容：API 变更是否会导致调用方崩溃？
8. 安全性：数据暴露、注入风险、缺少验证
9. 安全深度审查：跟踪所有外部输入的数据流 — 哪些入口接受外部输入？数据在哪里跨越信任域？敏感数据是否可能暴露给不该看到的人？认证/授权是否在每个入口点执行？是否存在绕过验证的路径？
10. 正确性深度审查：每个验收标准是否有对应的可测试验证条件？是否有 AC 被遗漏或模糊到无法测试？边界条件和错误情况是否被 AC 覆盖？是否存在非确定性行为（如无 tiebreaker 的排序、无唯一排序键的 LIMIT 查询）？
11. 资源/性能审查：数据量增长时行为如何？打开的连接/句柄是否在所有路径（含错误路径）上正确关闭？是否有潜在的内存增长或资源耗尽路径？检查每个 WHERE/JOIN 列是否有索引覆盖（注意 composite PK prefix scan 限制：PK(a,b) 对 WHERE b=? 无效）

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
  "location": "{文件:行号或文档章节}",
  "description": "{问题描述}",
  "evidence": "{具体证据引用}",
  "suggestion": "{修复建议}",
  "confidence": 0.0-1.0
}
```

## Fact-Gathering for Precision Filter (Design Review)

Between Recall and Precision passes, the main agent gathers these facts from the project:

```
facts_to_gather = [
  read the relevant prior-phase outputs (to verify cross-phase consistency claims)
  check the requirements document (to verify completeness claims)
  read the project's coding conventions / RULES.md (to verify best-practice claims)
]
```

Then inject gathered facts into the Precision Filter prompt from `review-precision-filter.md`.
