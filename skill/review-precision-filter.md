# Precision Filter Prompt (Shared)

This file is loaded when dual-pass review mode is active. Used by both design review (Phase 1–3) and code review (Phase 4–5).

## Precision Filter Prompt Template

Inject this prompt into the Precision subagent. The `{VERIFIED_FACTS}` and `{RAW_FINDINGS}` placeholders are filled by the main agent after the Recall pass and fact-gathering step.

```
你是一位资深审查质量审核员（Precision Filter Pass）。
另一位审查员已完成初步扫描，发现了 {N} 个潜在问题。
你的职责是：逐一验证每个发现的真实性和严重性，过滤掉误报。

## 验证所需的事实信息

以下是经过代码验证的客观事实，用于判断 findings 的真伪：

{VERIFIED_FACTS}

## 待验证的发现列表

{RAW_FINDINGS}

## 输出格式
对每个发现：
[F-XX] VERDICT: CONFIRM | DOWNGRADE | REJECT
Adjusted Severity: (仅 CONFIRM/DOWNGRADE 时)
Reason: {一句话判定依据}

## 验证标准
1. 事实准确性：问题是否真实存在？（用事实信息验证，非猜测）
2. 严重性恰当：severity 分级是否准确？（不夸大不缩小）
3. 证据充分：evidence 是否能支撑 description？
4. 可操作性：suggestion 是否可直接执行？
5. 根因去重：同一根因的多个表面症状应合并为一个 finding

## 已知的误报模式（需要警惕）
- 审查者误读了代码意图（如将有意的架构决策判定为缺陷）
- 审查者不了解项目上下文（如将框架惯例判定为反模式）
- 审查者过度泛化（如将局部合理做法判定为全局问题）
- 审查者重复报告同一根因的多个表面症状
- 审查者在没有证据的情况下猜测（如声称某属性不存在但未验证）
```

## Aggregation Logic

After Precision Pass, map confirmed findings into the Ralph tally:

```
for finding in precision_results:
    if finding.verdict == "CONFIRM":
        tally[finding.adjusted_severity] += 1
    elif finding.verdict == "DOWNGRADE":
        tally[finding.adjusted_severity] += 1  # 降级但保留
    elif finding.verdict == "REJECT":
        pass  # 过滤掉

# Confidence-based auto-downgrade for confirmed findings:
# confidence < 0.5 → auto-downgrade one severity level
# (C→H, H→M, M→L, L→I)
```

## Cost-Benefit Reference

Based on empirical testing:

| Metric | Value |
|--------|-------|
| Raw findings per round | 25 → 12 (after Precision filter) |
| True H-level findings | 2 (found with higher confidence) |
| H-level false positives | 0 |
| Effective finding rate | ~92% |
| Expected Ralph loop rounds | 3–5 (vs 5–7 without Precision filter) |

⛔ **Single-pass is forbidden.** The value is not cost reduction but quality improvement — preventing H-level false positives from triggering wasted fix rounds. One wasted round costs the same as an entire dual-pass round.
