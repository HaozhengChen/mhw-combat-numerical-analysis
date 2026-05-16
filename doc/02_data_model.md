# Data Model

This document explains the data model used by the Great Sword Skill Priority Recommender.

The workbook separates combat information into multiple structured sheets. This makes the model easier to inspect, update, and extend.

---

## Data Model Overview

The model uses the following data tables:

| Sheet | Purpose |
| --- | --- |
| MoveData | Individual move data |
| Combo | Combo composition |
| Combo_Calc | Aggregated combo values |
| Skills | Skill parameters by level |
| Weapons | Weapon stats |
| Hitzone | Monster hitzone values |
| Input | User-selected current state |
| Calc_Current | Current DPS calculation |
| Recommend | Skill +1 simulation |
| Curve_Future | Future skill investment calculation |
| Dashboard | Presentation layer |

Key principle:

```
Raw data, calculation, recommendation, and presentation should be separated.
```

---

## MoveData

`MoveData` stores individual move parameters.

Suggested columns:

| Column | Meaning |
| --- | --- |
| move_id | Unique move identifier |
| 招式 | Move name |
| 起手/蓄气 | Startup or charge frames |
| 维持 | Hold frames |
| 击中 | Hit frames |
| 卡肉 | Hitstop frames |
| MV | Physical motion value |
| MV_ele | Elemental motion value |
| 总帧数 | Total frames |
| 蓄力(Y/N) | Whether the move is a charge move |

Important notes:

- Hitstop is included in total frames.
- Slinger Burst has 0 MV and 0 elemental MV.
- True Charged Slash charge frames and hit frames should not be confused.
- Charge-related moves are used by Focus frame reduction logic.

---

## Combo

`Combo` stores combo composition by move IDs.

Suggested columns:

| Column | Meaning |
| --- | --- |
| combo_id | Unique combo identifier |
| combo_name | Combo display name |
| move_1 | First move |
| move_2 | Second move |
| move_3 | Third move |
| move_4 | Fourth move |
| move_5 | Fifth move |

The Combo sheet defines sequences.

The actual DPS formula uses aggregated values from `Combo_Calc`.

---

## Combo_Calc

`Combo_Calc` stores combo-level aggregated values.

Suggested columns:

| Column | Meaning |
| --- | --- |
| combo_id | Unique combo identifier |
| combo_name | Combo display name |
| MV_total | Total physical motion value |
| MV_ele_total | Total elemental motion value |
| frames_base | Base combo frames |
| Mpf | Motion value per frame |
| Epf | Elemental MV per frame |
| charge_count | Number of charge segments |
| charge_frame_per_charge | Base charge frames per charge segment |
| sheathe_count | Number of sheathing actions |

Modeling role:

```
frames_eff =
frames_base
- charge_count × focus_reduce_per_charge
- sheathe_count × sheathe_reduce_frames
```

This allows Focus and Quick Sheath to affect DPS through frames.

---

## Skills

`Skills` stores skill parameters by level.

Suggested columns:

| Column | Meaning |
| --- | --- |
| key | skill + level key |
| skill | Skill name |
| level | Skill level |
| level_max | Maximum level |
| coverage_key | Which coverage input controls this skill |
| include_in_recommend | Whether the skill appears in default recommendation |
| ATK_plus | Additive attack |
| ATK_multi | Attack multiplier |
| CRT_plus | Affinity bonus |
| CD | Critical damage multiplier |
| WEX_raw_CRT | Weakness Exploit affinity before tenderizing |
| WEX_soft_CRT | Weakness Exploit affinity after tenderizing |
| focus_reduce_pct | Focus charge reduction percentage |
| sheathe_reduce_frames | Quick Sheath frame reduction |
| ELE_plus | Additive element |
| ELE_multi | Element multiplier |
| status_plus | Additive status |
| status_multi | Status multiplier |
| action_multi | Action / shot / shelling multiplier |
| sharpness_effect | Sharpness-related note or effect |
| condition_note | Condition or note |

Recommended key format:

```
skill|level
```

Important rules:

- Every skill should include a level 0 row.
- Additive columns should default to 0.
- Multiplier columns should default to 1.
- Critical Boost level 0 should use `CD = 1.25`.
- High-condition skills can remain in the table but have `include_in_recommend = FALSE`.

---

## Weapons

`Weapons` stores weapon stats.

Suggested columns:

| Column | Meaning |
| --- | --- |
| weapon_id | Unique weapon ID |
| weapon_name | Weapon name |
| weapon_type | Weapon type |
| weapon_coef | Display attack coefficient |
| W_base | True raw |
| Display_ATK | Displayed attack |
| weapon_crt | Weapon affinity |
| ele_type | Element type |
| weapon_ele | Element value |
| sharpness | Sharpness color |
| S_phy | Physical sharpness multiplier |
| S_ele | Elemental sharpness multiplier |
| 备注 | Notes |

Important rules:

- Damage uses `W_base`, not `Display_ATK`.
- Negative affinity must remain negative.
- Non-elemental weapons should use `weapon_ele = 0`.

---

## Hitzone

`Hitzone` stores monster part data.

Suggested columns:

| Column | Meaning |
| --- | --- |
| hitzone_id | Unique hitzone ID |
| target | Monster or target name |
| state | Monster state |
| part | Body part |
| H_cut | Raw sever hitzone |
| H_dragon | Dragon elemental hitzone |
| H_soft_cut | Tenderized raw hitzone |
| weak_raw | Whether raw hitzone triggers WEX |
| weak_soft | Whether tenderized hitzone triggers WEX |
| Δ_soft% | Tenderizing improvement |
| 备注 | Notes |

Tenderized hitzone formula:

```
H_soft = ROUNDDOWN(0.75 × H_raw + 25, 0)
```

Weakness Exploit trigger:

```
weak_raw = H_cut >= 45
weak_soft = H_soft_cut >= 45
```

Important notes:

- Elemental hitzone does not use tenderizing.
- Tenderizing can change Weakness Exploit from inactive to active.
- Hitzone values should be numeric, not text.

---

## Input

`Input` stores current combat state.

Suggested sections:

### Basic Inputs

| Field | Example |
| --- | --- |
| weapon_id | GS_P2 |
| combo_id | COMBO_SLINGER_TCS |
| hitzone_id | TRAINING_POLE |
| tenderized | FALSE |
| sweet_spot_modifier | 1.03 |

### Coverage Inputs

| Field | Example |
| --- | --- |
| --- | ---: |
| WEX coverage | 75% |
| Agitator coverage | 75% |
| Peak Performance coverage | 50% |
| Resentment coverage | 0% |
| Latent Power coverage | 0% |
| Maximum Might coverage | 0% |

### Skill Level Inputs

| Skill | Example |
| --- | --- |
| --- | ---: |
| 攻击 | 3 |
| 看破 | 2 |
| 弱点特效 | 1 |
| 超会心 | 1 |
| 挑战者 | 3 |
| 无伤 | 1 |
| 怨恨 | 0 |
| 集中 | 0 |
| 纳刀术 | 0 |
| 属性强化 | 0 |

---

## Calc_Current

`Calc_Current` calculates the current DPS.

It reads from:

- Input
- Weapons
- Combo_Calc
- Hitzone
- Skills

Main outputs:

| Output | Meaning |
| --- | --- |
| W_base | Weapon true raw |
| weapon_crt | Weapon affinity |
| MV_total | Combo physical MV |
| MV_ele_total | Combo elemental MV |
| frames_base | Combo base frames |
| H_phys | Current physical hitzone |
| buff_atk_plus | Additive buff attack |
| skill_atk_plus | Additive skill attack |
| ATK_eff | Effective attack |
| CR_eff | Effective affinity |
| ECM | Expected Critical Multiplier |
| frames_eff | Effective frames |
| D_phys | Physical damage |
| D_ele | Elemental damage |
| D_total | Total damage |
| DPS | Current DPS |

---

## Recommend

`Recommend` simulates each candidate skill gaining one level.

Main columns:

| Column | Meaning |
| --- | --- |
| skill | Candidate skill |
| current_level | Current level |
| next_level | Simulated next level |
| max_level | Maximum level |
| valid | Whether this candidate is valid |
| simulated skill levels | Skill levels used in simulation |
| DPS_new | DPS after the simulated upgrade |
| gain | Marginal DPS gain |
| label | Display label |
| reason | Short explanation |

Core formula:

```
gain = DPS_new / DPS_current - 1
```

---

## Curve_Future

`Curve_Future` calculates future skill investment curves.

Instead of only simulating +1 level, it simulates additional investment levels.

The curve usually uses cumulative gain:

```
cumulative_gain = DPS_target_level / DPS_current - 1
```

---

## Dashboard

`Dashboard` is the presentation layer.

It summarizes:

- current scenario
- current DPS
- first recommendation
- first gain
- effective affinity
- effective frames
- ATK_eff
- ECM
- Weakness Exploit status
- Top skill recommendations
- marginal gain bar chart
- future skill investment curve
- model checks
- model notes

The Dashboard should not contain heavy formulas.

---

## Exported CSV Files

Recommended CSV exports:

```
exports/
├── MoveData.csv
├── Combo.csv
├── Combo_Calc.csv
├── Skills.csv
├── Weapons.csv
├── Hitzone.csv
├── Recommend.csv
└── Curve_Plot_Future.csv
```

CSV exports make the data model visible on GitHub without requiring Excel.

---

## Data Model Principles

### Separation of concerns

Raw data, formulas, recommendation logic, and presentation are separated.

### Stable identifiers

Use IDs such as:

```
weapon_id
combo_id
hitzone_id
move_id
```

### Level 0 skill rows

Every skill should have level 0 data.

### Numeric defaults

Use:

```
0 for additive values
1 for multiplier values
```

Avoid using text symbols such as `—` in numeric columns.

### Coverage keys

Conditional skills should use `coverage_key`.

This makes future expansion easier.

---

## Extension Potential

The current data model can be extended to support:

- more Great Sword combos
- more monsters and hitzones
- more weapons
- other weapon types
- per-hit damage calculation
- state enumeration
- armor and decoration constraints
- Python or Streamlit implementation