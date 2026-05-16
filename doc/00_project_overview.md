# Project Overview

This document introduces the portfolio project for the Monster Hunter: World Iceborne combat numerical analysis repository.

The current main module is the Great Sword Skill Priority Recommender: an Excel / Google Sheets MVP that calculates current DPS, simulates each candidate skill gaining one level, and ranks skills by marginal DPS gain.

---

## Project Goal

The goal is to demonstrate game numerical design and systems design ability through a concrete combat modeling tool.

This project focuses on:

- damage formula decomposition
- weapon, move, combo, skill, hitzone, and buff data modeling
- conditional skill value estimation
- skill marginal DPS gain calculation
- recommendation logic
- dashboard-based tool presentation

This project is not a gameplay guide and not a full armor optimizer.

The core question is:

```
Given the current combat state, which skill level provides the highest next-point DPS gain?
```

---

## Why This Project Matters

In Monster Hunter: World Iceborne, skill value is context-dependent.

A skill can be strong or weak depending on:

- weapon true raw
- weapon affinity
- elemental value
- combo motion value
- combo frame length
- monster hitzone
- tenderized state
- current skill levels
- buff state
- conditional uptime
- player behavior assumptions

Examples:

- Critical Boost is valuable only when effective affinity is high.
- Weakness Exploit is valuable only when the hitzone condition is met.
- Focus does not increase single-hit damage, but improves DPS by reducing charge frames.
- Quick Sheath does not increase single-hit damage, but improves cycle DPS by reducing sheathing frames.
- Element Attack has no value on a non-elemental weapon.

This makes skill recommendation a suitable numerical design problem.

---

## Core Deliverable

The main deliverable is an Excel / Google Sheets workbook with:

- structured data tables
- current DPS calculation
- skill +1 simulation
- recommendation ranking
- future skill investment curve
- dashboard visualization
- model checks

Recommended workbook files:

```
skill-priority-recommender/
├── MHW_Iceborne_GS_Skill_Recommender.xlsx
└── MHW_Iceborne_GS_Skill_Recommender.pdf
```

---

## Repository Structure

Recommended repository structure:

```
mhw-combat-numerical-analysis/
├── README.md
├── docs/
│   ├── 00_project_overview.md
│   ├── 01_damage_formula.md
│   ├── 02_data_model.md
│   ├── 03_skill_priority_recommender.md
│   ├── 04_test_cases.md
│   ├── 05_model_limitations.md
│   └── 06_future_work.md
│
├── skill-priority-recommender/
│   ├── README.md
│   ├── MHW_Iceborne_GS_Skill_Recommender.xlsx
│   ├── MHW_Iceborne_GS_Skill_Recommender.pdf
│   ├── exports/
│   ├── screenshots/
│   └── notes/
│
└── assets/
```

---

## Main Workbook Sheets

| Sheet | Purpose |
| --- | --- |
| MoveData | Move values, elemental MV, frames, and charge flag |
| Combo | Combo composition by move IDs |
| Combo_Calc | Aggregated combo MV, frames, charge count, and sheathe count |
| Skills | Skill parameters by level |
| Weapons | Weapon true raw, affinity, element, and sharpness |
| Hitzone | Raw hitzone, elemental hitzone, tenderized hitzone, and WEX status |
| Input | Current combat state input |
| Calc_Current | Current DPS calculation |
| Recommend | Skill +1 simulation and ranking |
| Curve_Future | Future skill investment calculation |
| Curve_Plot_Future | Chart-ready curve data |
| Dashboard | Presentation layer |

---

## Key Design Idea

The recommender is built around marginal value.

Instead of asking:

```
Which skill is generally best?
```

The model asks:

```
Given the current state, what happens if only one skill increases by one level?
```

The model calculates:

```
gain = DPS_new / DPS_current - 1
```

Then sorts candidate skills by gain.

This makes the recommendation dynamic and state-dependent.

---

## Example Scenario

Example input state:

```
Weapon: 黑龙玄刃
Combo: 直斩投射器真蓄
Target: 训练场木桩
Tenderized: FALSE
Sweet spot modifier: 1.03
```

Example output:

| Rank | Skill Upgrade | Gain |
| --- | --- | --- |
| ---: | --- | ---: |
| 1 | 攻击 3→4 | +1.96% |
| 2 | 看破 2→3 | +1.26% |
| 3 | 集中 0→1 | +1.02% |
| 4 | 弱点特效 1→2 | +0.75% |
| 5 | 挑战者 3→4 | +0.69% |

Interpretation:

- Attack Boost 3→4 is strong because it adds both raw attack and affinity.
- Critical Eye is useful because the weapon starts with negative affinity.
- Focus has value because the selected combo contains a charge segment.
- Critical Boost is weak when effective affinity is still negative or very low.

---

## Portfolio Value

This project demonstrates:

### Numerical modeling

The project decomposes combat damage into true raw, motion value, sharpness, hitzone, affinity, critical damage, elemental damage, frame duration, buffs, and skills.

### Data table design

The model separates data into dedicated tables instead of hardcoding all values.

### Systems thinking

The recommendation is not a static tier list. It responds to current state.

### Tool design

The Dashboard presents model output in a way that can be understood quickly by a reviewer.

---

## Current MVP Scope

The current MVP includes:

- Great Sword focused combo model
- physical and elemental expected damage
- positive and negative affinity
- Weakness Exploit hitzone logic
- tenderized hitzone logic
- Focus frame reduction
- Quick Sheath frame reduction
- conditional skill coverage approximation
- next-point skill ranking
- future skill investment curves

---

## Out of Scope for MVP

The current MVP does not include:

- armor set search
- decoration slot optimization
- full equipment constraints
- per-hit damage flooring
- sharpness consumption
- monster state timeline
- full status enumeration
- all weapons
- all monsters

These are planned as future extensions.

---

## Summary

This project turns a game combat system into a structured numerical model.

The main contribution is not just calculating DPS, but building a tool that explains how skill value changes under different combat states.

```
Combat system → data model → damage formula → skill simulation → recommendation output → dashboard presentation
```