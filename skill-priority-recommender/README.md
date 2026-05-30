# Great Sword Skill Priority Recommender

An Excel / Google Sheets based MVP for evaluating skill priority in Monster Hunter: World Iceborne Great Sword builds.

This project calculates current DPS from weapon, combo, hitzone, buff, coverage, and skill inputs. It then simulates each candidate skill gaining one additional level and ranks all candidates by marginal DPS gain.

The goal is not to build a full armor optimizer. Instead, this project focuses on combat numerical modeling, skill marginal value calculation, and dashboard-based tool presentation for a game systems / numerical design portfolio.

---

## Overview

In Monster Hunter: World Iceborne, the value of a skill is highly context-dependent.

A skill's value depends on:

- weapon true raw
- weapon affinity
- elemental value
- sharpness multiplier
- combo motion value
- combo frame length
- hitzone value
- tenderized state
- current skill levels
- buff state
- conditional skill coverage

This project answers one core question:

> Given the current combat state, which skill level provides the highest next-point DPS gain?
> 

---

## Dashboard Preview

The workbook includes a dashboard that summarizes current combat state, current DPS, next-skill recommendations, marginal gain ranking, investment curves, and model checks.

Recommended screenshot path:

screenshots/dashboard.png

Add this image after exporting the dashboard:

![Dashboard](./screenshots/dashboard.png)

Dashboard

---

## Key Features

- Current DPS calculation
- Physical and elemental damage split
- True raw based attack calculation
- Positive and negative affinity support
- Expected Critical Multiplier, or ECM
- Weakness Exploit trigger logic
- Tenderized and non-tenderized hitzone handling
- Conditional skill coverage approximation
- Focus frame reduction
- Quick Sheath frame reduction
- Candidate skill +1 simulation
- Marginal DPS gain ranking
- Future skill investment curve
- Portfolio-ready dashboard

---

## Workbook Structure

| Sheet | Purpose |
| --- | --- |
| MoveData | Stores move values, elemental MV, frames, and charge flag |
| Combo | Defines combo composition by move IDs |
| Combo_Calc | Aggregates combo MV, elemental MV, base frames, charge count, and sheathe count |
| Skills | Stores skill parameters by level |
| Weapons | Stores weapon true raw, affinity, element, and sharpness multipliers |
| Hitzone | Stores raw hitzone, elemental hitzone, tenderized hitzone, and Weakness Exploit flags |
| Input | User-defined current combat state |
| Calc_Current | Calculates current DPS |
| Recommend | Simulates each candidate skill gaining one level |
| Curve_Future | Calculates future skill investment curves |
| Curve_Plot_Future | Prepares curve data for charting |
| Dashboard | Final presentation layer |

---

## Core Model

The MVP uses a continuous expected damage model.

Effective attack:
```
ATK_eff = (W_base + buff_atk_plus + skill_atk_plus) × ATK_multi_eff
```
Effective affinity:
```
CR_eff = weapon_crt + CRT_plus
```
Expected Critical Multiplier:
```
If CR_eff >= 0:

ECM = 1 + CR_eff × (CD - 1)

If CR_eff < 0:

ECM = 1 + CR_eff × 0.25
```
Physical damage:
```
D_phys = MV_total / 100 × ATK_eff × S_phy × H_phys / 100 × ECM × M_sweet
```
Elemental damage:
```
D_ele = MV_ele_total × ELE_eff / 10 × S_ele × H_dragon / 100
```
Effective frames:
```
frames_eff = frames_base - charge_count × ROUNDDOWN(charge_frame_per_charge × focus_reduce_pct, 0) - sheathe_count × sheathe_reduce_frames
```
DPS:
```
DPS = D_total × 30 / frames_eff
```
Marginal gain:
```
gain = DPS_new / DPS_current - 1
```

More detailed formula notes are available in:

notes/[formulas.md](./notes/formulas.md)

---

## Recommendation Logic

For each candidate skill, the model performs the following steps:

1. Read the current skill level.
2. Increase only that skill by one level.
3. Keep all other skills and inputs unchanged.
4. Recalculate DPS.
5. Compare the new DPS with current DPS.
6. Sort all candidate skills by marginal gain.

This allows the tool to recommend the best next-point skill under the current combat state.

---

## Candidate Skills

The current MVP includes the following candidate skills:

| Skill | Role |
| --- | --- |
| 攻击 | Adds attack and grants affinity from level 4 onward |
| 看破 | Adds affinity |
| 弱点特效 | Adds affinity if hitzone conditions are met |
| 超会心 | Increases critical damage multiplier |
| 挑战者 | Adds attack and affinity based on rage coverage |
| 无伤 | Adds attack based on full-health coverage |
| 怨恨 | Adds attack based on red-health coverage |
| 集中 | Reduces charge frames |
| 纳刀术 | Reduces sheathing frames |
| 属性强化 | Increases elemental value |

Conditional or high-risk skills such as Heroics, Fortify, Offensive Guard, and Coalescence can remain in the Skills table but are not included in the default recommendation set.

---

## Example Output

Example recommendation result from one Fatalis Blade test scenario:

| Rank | Skill Upgrade | Gain |
| --- | --- | --- |
| 1 | 攻击 3→4 | +1.96% |
| 2 | 看破 2→3 | +1.26% |
| 3 | 集中 0→1 | +1.02% |
| 4 | 弱点特效 1→2 | +0.75% |
| 5 | 挑战者 3→4 | +0.69% |

In this example, Attack Boost 3→4 ranks highly because it provides both raw attack and affinity. Critical Boost has low value because effective affinity is still negative.

---

## Conditional Skill Modeling

The MVP uses coverage-rate approximation for conditional skills.

Examples:

Agitator_ATK_expected = Agitator_ATK × agitator_coverage

Agitator_CRT_expected = Agitator_CRT × agitator_coverage

Peak_ATK_expected = Peak_ATK × peak_coverage

Resentment_ATK_expected = Resentment_ATK × resentment_coverage

Weakness Exploit also depends on hitzone state:

If H_phys >= 45, Weakness Exploit can trigger.

If H_phys < 45, Weakness Exploit gain should be 0.

Tenderizing can change H_phys and therefore change whether Weakness Exploit can trigger.

---

## Dashboard Outputs

The dashboard displays:

- current weapon
- current combo
- current target hitzone
- tenderized state
- sweet spot modifier
- current DPS
- first recommended skill
- first marginal gain
- effective affinity
- effective frames
- effective attack
- ECM
- Weakness Exploit trigger state
- recommendation ranking
- marginal gain bar chart
- future skill investment curve
- model checks
- model assumptions

---

## Validation

The workbook includes several checks:

| Check | Purpose |
| --- | --- |
| Affinity cap check | Ensures final affinity is capped between -100% and 100% |
| Effective frame check | Ensures frames_eff is greater than 0 |
| Weakness Exploit check | Shows whether current hitzone can trigger WEX |
| Focus check | Shows whether the current combo contains charge segments |
| Quick Sheath check | Shows whether the current combo contains sheathing actions |

Suggested validation cases are documented in:

notes/[validation_notes.md](./notes/validation_notes.md)

---

## Model Assumptions

The current MVP makes the following assumptions:

- Uses continuous expected damage.
- Does not apply per-hit flooring.
- Uses coverage-rate approximation for conditional skills.
- Does not enumerate all possible combat states.
- Does not search armor, decorations, or slot constraints.
- Focused primarily on Great Sword scenarios.
- Uses combo-level aggregated MV and frames rather than per-hit simulation.

These limitations are intentional for the MVP stage.

The goal is to validate skill marginal value modeling, not to build a complete armor optimizer.

---

## Files

Recommended folder structure:

```
skill-priority-recommender/

├── [README.md](./README.md)
├── MHW_Iceborne_GS_Skill_Recommender.xlsx
├── MHW_Iceborne_GS_Skill_Recommender.pdf
├── exports/
│   ├── MoveData.csv
│   ├── Combo.csv
│   ├── Combo_Calc.csv
│   ├── Skills.csv
│   ├── Weapons.csv
│   ├── Hitzone.csv
│   ├── Recommend.csv
│   └── Curve_Plot_Future.csv
├── screenshots/
│   ├── dashboard.png
│   ├── recommendation_bar_chart.png
│   └── skill_investment_curve.png
└── notes/
    ├── [formulas.md](./notes/formulas.md)
    ├── [validation_notes.md](./notes/validation_notes.md)
    └── [dashboard_design.md](./notes/dashboard_design.md)
```
---

## Future Work

Possible next steps:

- Add per-hit damage floor.
- Replace coverage-rate approximation with state enumeration.
- Add armor and decoration constraints.
- Add slot efficiency evaluation.
- Add more weapons.
- Add more monsters and hitzones.
- Build a Python or Streamlit interactive version.
- Compare model output with in-game training area tests.

---

## Disclaimer

This is a fan-made analytical portfolio project for educational and non-commercial purposes.

Monster Hunter: World Iceborne and all related intellectual property belong to Capcom. This repository does not claim ownership of any game data, names, or assets.