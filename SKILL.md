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
version: 1.0.2
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

## 发布前强化检查（高发返工点兜底）

在七项通用检查之外，对高发返工点做强制检查（对应 claim / cross-material 治理线发现的 P0/P1）。这些也是 `content-writer` 表达红线的发布前兜底：

- **P0 命名一致性**：技术博客产品对外名必须与同 campaign 已发布物料（新闻稿/官网/公众号）逐字一致；不得擅自加版本号/后缀（如对外统一叫 X，博客不得写"X 2.0"）。不一致即 FAIL。
- **P0 元语言/自我指涉**：出现"本文…""新闻稿把…讲清楚了""值得单独展开""回到…整体叙事"等跳出框架句式即 FAIL，改为内容直接过渡。
- **P0 商务腔**：出现"多、快、好、省"等四字口号即 FAIL，改工程维度（负载覆盖/执行效率/运维体验/资源效率）。
- **P1 绝对化表述**：扫描"天然打通/无缝/必然/一定/零"等绝对化词，要求改为带边界的定性表述。
- **P1 超范围场景**：落地行业/场景若无知识库或已发文章出处，标 `[需确认]` 或删除，不得凭印象列举（如某行业/场景需有内部来源背书）。
- **P1 忠实转录 vs 量化断言**：基础设施能力有架构图背书可写；量化加速倍数无官方口径标 FAIL。

> 涉及对外发布且同主题已有多份物料时，预审 PASS 后建议追加 `claim-to-source-auditor` + `cross-material-consistency-auditor` + `tech-content-review-panel` 三件套治理线，再定稿。

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
