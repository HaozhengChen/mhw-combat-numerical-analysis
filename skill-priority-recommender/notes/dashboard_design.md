# Dashboard Design Notes

This document explains the design of the Dashboard sheet in the Great Sword Skill Priority Recommender workbook.

The Dashboard is the presentation layer of the project. It is designed for portfolio review, screenshots, and quick interpretation. It should not contain heavy calculation logic. Most calculations should remain in `Calc_Current`, `Recommend`, and `Curve_Future`.

---

## 1. Dashboard Purpose

The Dashboard answers four questions:

1. What is the current combat state?
2. What is the current DPS?
3. Which skill should be upgraded next?
4. Why does the recommendation make sense?

The Dashboard is not the main calculation engine.

Its role is to summarize and communicate results.

---

## 2. Dashboard Inputs and Dependencies

The Dashboard mainly references the following sheets:

| Sheet | Role |
| --- | --- |
| Input | Current user-selected combat state |
| Calc_Current | Current DPS and intermediate metrics |
| Recommend | Next-point skill recommendation ranking |
| Curve_Plot_Future | Future skill investment curve data |

The Dashboard should not duplicate complex DPS formulas.

Instead, it should reference calculated outputs from other sheets.

---

## 3. Recommended Layout

The Dashboard can be organized into the following areas:

```
Title Area
Current Scenario Summary
KPI Cards
Current Calculation Details
Top Skill Recommendations
Marginal Gain Bar Chart
Future Skill Investment Curve
Model Checks
Model Notes
```

Suggested visual layout:

```
┌──────────────────────────────────────────────┐
│ Title                                        │
├──────────────────────────────────────────────┤
│ Scenario Summary       KPI Cards             │
├───────────────────┬──────────────────────────┤
│ Current Details   │ Top Recommendations      │
├───────────────────┴──────────────────────────┤
│ Bar Chart             Investment Curve       │
├──────────────────────────────────────────────┤
│ Model Checks / Notes                         │
└──────────────────────────────────────────────┘
```

---

## 4. Title Area

The title should clearly communicate the project.

Recommended title:

```
《怪物猎人：世界 Iceborne》大剑技能边际收益推荐器
```

Optional English title:

```
Great Sword Skill Priority Recommender
```

Recommended subtitle:

```
基于当前武器、连段、肉质、覆盖率与技能等级，计算下一点技能的 DPS 边际收益
```

Visual style:

| Element | Recommendation |
| --- | --- |
| Title background | Deep orange |
| Title text | White |
| Title font size | 18–22 |
| Title alignment | Center |
| Subtitle background | Light orange or white |
| Subtitle font size | 10–11 |

---

## 5. Current Scenario Summary

This area summarizes the current input state.

Recommended fields:

| Field | Source |
| --- | --- |
| Weapon | Input weapon_id matched to Weapons weapon_name |
| Combo | Input combo_id matched to Combo_Calc combo_name |
| Target Part | Input hitzone_id matched to Hitzone target / state / part |
| Tenderized State | Input tenderized boolean |
| Sweet Spot Modifier | Input sweet spot modifier |

Example:

| Field | Value |
| --- | --- |
| 武器 | 黑龙玄刃 |
| 连段 | 直斩投射器真蓄 |
| 目标部位 | 训练场木桩 / 固定 / 木桩 |
| 软化状态 | 未软化 |
| 刃中补正 | 1.03 |

Design recommendation:

- left-align field names
- left-align text values
- use light background
- keep this area compact

---

## 6. KPI Cards

KPI cards show the most important outputs.

Recommended cards:

| KPI | Source |
| --- | --- |
| 当前 DPS | Calc_Current DPS |
| 第一推荐 | Recommend rank 1 skill upgrade |
| 第一收益 | Recommend rank 1 gain |
| 当前会心 | Calc_Current CR_eff |
| 当前有效帧 | Calc_Current frames_eff |
| 当前 ATK | Calc_Current ATK_eff |
| 当前 ECM | Calc_Current ECM |
| 弱特状态 | Calc_Current weak_ok |

Example:

```
当前 DPS
154.58
```

```
第一推荐
攻击 3→4
```

```
第一收益
1.96%
```

Design recommendation:

| Element | Recommendation |
| --- | --- |
| KPI title | 10–11 pt, bold |
| KPI value | 16–20 pt, bold |
| Alignment | Center |
| Background | Light orange / light gray / light blue |
| Border | Thin light gray |

---

## 7. Current Calculation Details

This table shows important intermediate values.

Recommended fields:

| Metric | Meaning |
| --- | --- |
| W_base | Weapon true raw |
| weapon_crt | Weapon base affinity |
| H_phys | Current physical hitzone |
| buff_atk_plus | Additive buff attack |
| skill_atk_plus | Expected additive skill attack |
| ATK_eff | Effective attack |
| CR_eff | Effective affinity |
| ECM | Expected Critical Multiplier |
| D_total | Total expected combo damage |
| DPS | Current DPS |

Formatting recommendation:

| Metric Type | Format |
| --- | --- |
| Damage / DPS | 2 decimals |
| ECM | 4 decimals |
| Affinity | Percentage, 2 decimals |
| Frames | Integer |
| Hitzone | Integer |
| Attack | 2 decimals |

This table helps reviewers understand how the final DPS was produced.

---

## 8. Top Skill Recommendations

This table summarizes the next-point recommendation ranking.

Recommended columns:

| Column | Meaning |
| --- | --- |
| Rank | Recommendation rank |
| Skill Upgrade | Candidate skill level change |
| New DPS | DPS after the upgrade |
| Gain | Marginal DPS gain |
| Reason | Short explanation |

Example:

| Rank | Skill Upgrade | New DPS | Gain | Reason |
| --- | --- | --- | --- | --- |
| ---: | --- | ---: | ---: | --- |
| 1 | 攻击 3→4 | 157.61 | 1.96% | Adds raw and affinity |
| 2 | 看破 2→3 | 156.53 | 1.26% | Offsets negative affinity |
| 3 | 集中 0→1 | 156.16 | 1.02% | Reduces charge frames |

Recommended display count:

```
Top 5
```

Full Top 10 can remain in the `Recommend` sheet.

Dashboard should prioritize clarity.

---

## 9. Recommendation Reason Text

Reason text should be short.

Suggested reason text:

| Skill | Reason |
| --- | --- |
| 攻击 | Adds raw; level 4 also adds affinity |
| 看破 | Increases affinity and offsets negative affinity |
| 弱点特效 | Provides affinity when hitzone condition is met |
| 超会心 | Improves critical damage; depends on affinity |
| 挑战者 | Adds raw and affinity based on rage coverage |
| 无伤 | Adds raw based on full-health coverage |
| 怨恨 | Adds raw based on red-health coverage |
| 集中 | Reduces charge frames and improves cycle DPS |
| 纳刀术 | Reduces sheathing frames |
| 属性强化 | Improves elemental damage if the weapon has element |

Reason text should not be too long.

The Dashboard should explain, not overwhelm.

---

## 10. Marginal Gain Bar Chart

This chart answers:

```
Which skill should be upgraded next?
```

Recommended chart type:

```
Horizontal bar chart
```

Recommended data source:

| Axis | Data |
| --- | --- |
| Category | Skill Upgrade |
| Value | Gain |

Design recommendation:

- show Top 5 or Top 8 only
- use orange bars
- add data labels
- format labels as percentages
- remove unnecessary legend
- keep gridlines light or remove them
- use short skill labels if possible

Example label format:

```
攻击 3→4
看破 2→3
集中 0→1
```

---

## 11. Future Skill Investment Curve

This chart answers:

```
If I continue investing in a skill, how does the return change?
```

Recommended chart type:

```
Line chart with markers
```

Recommended x-axis:

```
Additional skill points invested
```

Recommended y-axis:

```
Cumulative gain compared with current DPS
```

Recommended data source:

```
Curve_Plot_Future
```

Recommended series:

- 攻击
- 看破
- 弱点特效
- 超会心
- 挑战者
- 集中

The Dashboard does not need to show every possible skill line.

Too many lines can reduce readability.

Full curve data can remain in the calculation sheets.

---

## 12. Model Checks

The model check area helps reviewers understand whether the current state is valid.

Recommended checks:

| Check | Purpose |
| --- | --- |
| Affinity cap check | Detects if CR_eff exceeds cap range |
| Effective frame check | Ensures frames_eff > 0 |
| Weakness Exploit check | Shows whether WEX is active |
| Focus check | Shows whether the combo has charge segments |
| Quick Sheath check | Shows whether the combo has sheathing actions |
| Element check | Shows whether Element Attack should have value |

Example:

| Check | Status | Explanation |
| --- | --- | --- |
| 会心封顶检查 | OK | 当前有效会心在正常范围内 |
| 有效帧数检查 | OK | frames_eff 正常 |
| 弱特触发检查 | 触发 | 当前 H_phys ≥ 45，弱点特效可按覆盖率生效 |
| 集中收益检查 | 有蓄力段 | 当前连段包含蓄力段，集中可提升 DPS |
| 纳刀术收益检查 | 有纳刀 | 当前连段包含纳刀动作，纳刀术可提升 DPS |

Status formatting:

| Status | Suggested Style |
| --- | --- |
| OK | Green |
| Warning | Orange |
| Error | Red |
| Triggered | Light green |
| Not triggered | Gray |

---

## 13. Model Notes

The Dashboard should include a short model note section.

Suggested text:

```
1. This tool calculates the marginal DPS gain of each next skill level.
2. Conditional skills use coverage-rate approximation.
3. Focus and Quick Sheath do not change single-hit damage; they improve DPS by reducing frames.
4. The current MVP uses continuous expected damage and does not apply per-hit flooring.
5. This tool is a Skill Priority Recommender, not an armor or decoration optimizer.
```

The notes should be short enough to fit on the Dashboard.

More detailed explanation should be placed in documentation files.

---

## 14. Color Palette

Recommended visual direction:

```
Professional numerical tool with a light Monster Hunter-inspired orange theme
```

Suggested colors:

| Purpose | Color |
| --- | --- |
| Title background | Deep orange |
| Accent | Orange |
| Light background | Light warm orange |
| Main text | Dark gray |
| Secondary text | Medium gray |
| Border | Light gray |
| OK status | Green |
| Warning status | Orange |
| Error status | Red |
| Chart secondary color | Blue gray |

The Dashboard should not overuse orange.

Orange should highlight structure and key metrics.

---

## 15. Typography

Recommended fonts:

| Environment | Font |
| --- | --- |
| Chinese Excel | Microsoft YaHei |
| English labels | Calibri / Segoe UI / Arial |
| Numbers | Calibri / Segoe UI / Arial |

Recommended font sizes:

| Area | Size |
| --- | --- |
| --- | ---: |
| Main title | 18–22 |
| Subtitle | 10–11 |
| KPI title | 10–11 |
| KPI value | 16–20 |
| Table header | 10–11 |
| Table body | 9–10 |
| Chart title | 12–14 |
| Notes | 9 |

---

## 16. Alignment Rules

Recommended alignment:

| Content Type | Alignment |
| --- | --- |
| Main title | Center |
| Subtitle | Center |
| KPI cards | Center |
| Table headers | Center |
| Metric names | Left |
| Skill names | Left |
| Reason text | Left |
| Numerical values | Right |
| Percentages | Right |
| Rank | Center |
| Status labels | Center |

This makes the dashboard easier to scan.

---

## 17. Number Formatting

Recommended number formats:

| Field | Format |
| --- | --- |
| DPS | 2 decimals |
| Damage | 2 decimals |
| ATK_eff | 2 decimals |
| ECM | 4 decimals |
| Affinity | Percentage, 2 decimals |
| Gain | Percentage, 2 decimals |
| Frames | Integer |
| Hitzone | Integer |

Avoid showing unnecessary long decimals on the Dashboard.

Long decimals can remain in calculation sheets if needed.

---

## 18. Recommended Dashboard Scope

The Dashboard should show:

- current scenario
- current DPS
- key metrics
- Top 5 recommendations
- one bar chart
- one investment curve
- model checks
- short model notes

The Dashboard should not show:

- every candidate calculation
- every formula
- every raw data table
- full exported data
- all validation cases

Those details should remain in separate sheets or documentation files.

---

## 19. Portfolio Use

The Dashboard is intended for:

- GitHub README screenshot
- portfolio PDF
- interview discussion
- quick project overview

A reviewer should be able to understand the project within 10 seconds:

```
Current state → current DPS → next skill recommendation → why it makes sense
```

---

## 20. Design Summary

The Dashboard should look like a small internal design tool.

It should feel:

- readable
- structured
- data-driven
- easy to explain
- not over-decorated

The main design goal is clarity.

Visual polish should support the numerical model, not distract from it.