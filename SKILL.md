---
name: tcms-compliance-reviewer
description: |
  For tech-product marketing teams — pre-publish compliance and quality review of brand-side drafts (blogs, case studies, product write-ups, press releases), checking facts, customer redaction, product naming, competitor rules, and internal-info leaks.
  Reports issues and fixes only; never auto-edits the original draft.
  Not for monthly performance analysis — use tcms-performance-analyst for that.
read_when:
  - 预审
  - 审核
  - 审稿
  - 终审
  - review
  - 检查初稿
  - fact check
  - 合规检查
  - 发布前审核
  - 发稿前检查
  - pre-publication review
version: 1.0.1
disable: false
---

# TCMS Compliance Reviewer

对完稿后的技术博客、客户案例、产品解读或新闻稿做发布前合规与质量预审。只检查和报告，不自动修改原文。

## When to use

- 需要检查文章中的数据事实、客户脱敏、产品正式名称、竞品规则和内部信息。
- 需要在发布前确认引用追溯和格式。
- 需要生成带审批建议的预审报告。

## Do not use

- 月度效果分析和内容复盘，由 `content-performance-analyst` 处理。
- 自动修改或重写原文；仅提供位置和修改建议。
- 初次写作或渠道适配。

## Input

```yaml
draft_path:
article_type: tech_blog | case_study | product_update | other
compliance_profile: path or object
  - brand_rules:
  - sensitive_terms:
  - product_public_status:
  - customer_redaction_policy:
  - competitor_policy:
review_requested_by:
```

合规 profile 由项目私有配置提供。通用引擎使用逻辑字段；真实路径和规则表留私有层。

## Workflow

### Step 1: [Deterministic] Load the draft and reference

1. 读取待审文章。
2. 读取合规 profile 中的品牌规则、敏感词表、产品公开状态和客户脱敏规则。
3. 按需查找知识库中对应产品章节，用于验证数据出处。

### Step 2: [LLM] Item-by-item inspection

| # | Check | PASS description |
|---|---|---|
| 1 | Customer redaction | No real internal customer name; description cannot identify the customer |
| 2 | Product formal name | First mention uses the official full name |
| 3 | Competitor rules | No competitor company or product name |
| 4 | Data traceability | Each figure can be traced to an approved source |
| 5 | Internal information | No internal code names, project names or unconfirmed capabilities |
| 6 | Format and structure | Word count, brand title, closing and citation table comply |
| 7 | Content quality | Core message is clear, logic is consistent, no obvious AI template language |

Every item gets: `PASS` / `NEEDS_FIX` / `FAIL`.

### Step 3: [LLM] Generate pre-review report

Output:

```markdown
# Pre-Review Report

## Summary

| Item | Result | Note |
|---|---|---|
| Customer redaction | PASS/NEEDS_FIX/FAIL | |
| Product name | PASS/NEEDS_FIX/FAIL | |
| Competitor | PASS/NEEDS_FIX/FAIL | |
| Data traceability | PASS/NEEDS_FIX/FAIL | |
| Internal info | PASS/NEEDS_FIX/FAIL | |
| Format | PASS/NEEDS_FIX/FAIL | |
| Content quality | PASS/NEEDS_FIX/FAIL | |

## Specific issues

- Location, problem, suggested fix for each NEEDS_FIX or FAIL.

## Citation verification

| Claim | Source | Status |
|---|---|---|

## Overall assessment

- Publishable / Minor fixes / Major revision / Rewrite recommended

## Recommended approval level

- Level and rationale.
```

### Step 4: [Deterministic] Save

保存到：`content/drafts/{original-name}-compliance-review.md`

执行摘要：

```markdown
## Execution Summary
- Article reviewed:
- Results: N pass / M needs-fix / K fail
- Overall: publishable | minor-fix | major-fix | rewrite
- Critical issues:
```

## Hard Rules

1. Never modify the original draft. Only report issues and suggestions.
2. Data checks must reference an approved source. Missing sources receive a fail.
3. Any real customer name hit is an automatic fail.
4. Unconfirmed product capabilities receive a needs-fix.
5. Competitor named references must be flagged.
6. Internal code names or project names are an automatic fail.
7. Human confirmation required before marking the review complete and handing off to adaptation.

## Failure Handling

| Scenario | Action |
|---|---|
| Draft not found | List available drafts for the user |
| Compliance profile missing | Warn and skip brand/sensitive-term checks; flag as incomplete |
| Knowledge-base section unavailable | Mark all data items as unverifiable |
| Draft has no citation table | Mark as recommended |
| Competitor, customer or product policy changed | Request updated profile before proceeding |

## Output Format

```text
content/drafts/{original-name}-compliance-review.md
```

## Verification

- [ ] All seven inspection items evaluated.
- [ ] Every data claim checked against approved source.
- [ ] Every real customer name flagged.
- [ ] Report saved without modifying the original draft.
- [ ] Execution summary includes next action.
