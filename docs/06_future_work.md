# Future Work

This document outlines possible future improvements for the Great Sword Skill Priority Recommender and the broader Monster Hunter: World Iceborne combat numerical analysis project.

The current MVP focuses on Excel-based skill marginal value recommendation. Future work can improve precision, scope, automation, and usability.

---

## Version Roadmap

Suggested roadmap:

| Version | Focus |
| --- | --- |
| V1 | Excel MVP skill priority recommender |
| V2 | Per-hit damage calculation |
| V3 | State enumeration expected value model |
| V4 | Armor and decoration constraint solver |
| V5 | Python / Streamlit interactive app |
| V6 | Multi-weapon expansion |
| V7 | Hunt timeline and uptime model |

---

## V1: Current Excel MVP

Current completed scope:

- Great Sword focused model
- combo-level expected damage
- physical and elemental damage split
- positive and negative affinity
- ECM calculation
- Weakness Exploit hitzone logic
- tenderized hitzone logic
- Focus frame reduction
- Quick Sheath frame reduction
- conditional skill coverage
- next-point recommendation ranking
- future investment curve
- Dashboard presentation

The current MVP is suitable for portfolio presentation.

---

## V2: Per-Hit Damage Calculation

The next precision upgrade is per-hit damage calculation.

Instead of using:

```
MV_total
MV_ele_total
```

The model can calculate each hit separately.

Per-hit model:

```
D_phys_hit = FLOOR(MV_hit / 100 × ATK_eff × S_phy × H_phys / 100 × ECM × M_sweet)
D_ele_hit = FLOOR(MV_ele_hit × ELE_eff / 10 × S_ele × H_ele / 100)
D_hit = D_phys_hit + D_ele_hit
D_total = SUM(D_hit)
```

Benefits:

- closer in-game damage parity
- supports multi-hit attacks
- supports different hitzones per hit
- supports per-hit rounding behavior
- improves validation against training area tests

Required data changes:

- add hit-level move table
- store per-hit MV
- store per-hit elemental MV
- store per-hit hitzone if needed

---

## V3: State Enumeration Model

The current MVP uses coverage-rate approximation.

Future model can enumerate combat states.

Example states:

| State | Variables |
| --- | --- |
| State 1 | rage on, tenderized, WEX active |
| State 2 | rage on, not tenderized, WEX inactive |
| State 3 | rage off, tenderized, WEX active |
| State 4 | rage off, not tenderized, WEX inactive |

Expected DPS:

```
Expected_DPS = SUM(DPS_state × probability_state)
```

Benefits:

- handles conditional skill interactions more accurately
- separates tenderize uptime from rage uptime
- supports more realistic combat assumptions
- improves explanation of conditional skills

---

## V4: Armor and Decoration Constraint Solver

The current recommender does not search equipment.

Future version can add armor and decoration constraints.

Potential inputs:

- armor pieces
- decoration slots
- charms
- set bonuses
- skill caps
- owned decorations
- required skills
- banned skills

Potential outputs:

- best build under constraints
- skill priority with slot cost
- marginal gain per slot
- build comparison table

Possible metric:

```
slot_efficiency = DPS_gain / slot_cost
```

This would turn the tool from a Skill Priority Recommender into an Armor Optimizer.

---

## V5: Python / Streamlit App

The Excel MVP can be converted into an interactive Python app.

Possible stack:

- Python
- pandas
- Streamlit
- Plotly
- CSV data tables

Potential app features:

- dropdowns for weapon, combo, and hitzone
- sliders for coverage rates
- skill level selectors
- live recommendation ranking
- interactive charts
- scenario comparison
- exportable results

Benefits:

- easier sharing
- cleaner UI
- better chart interaction
- easier version control
- easier formula testing

---

## V6: Multi-Weapon Expansion

The current model focuses on Great Sword.

Future versions can support additional weapons.

Each weapon type may need unique modeling.

Examples:

| Weapon | Additional Modeling Needs |
| --- | --- |
| Dual Blades | high hit count, elemental focus, stamina management |
| Long Sword | gauge levels, Spirit Gauge uptime |
| Charge Blade | phials, AED / SAED, guard points |
| Gunlance | shelling, fixed damage, artillery |
| Bow | charge levels, coatings, stamina |
| Heavy Bowgun | ammo type, reload, recoil, clip size |

This requires weapon-specific data modules.

---

## V7: Hunt Timeline Model

A future advanced version can model a full hunt timeline.

Potential timeline states:

- monster rage
- tenderized uptime
- downed windows
- mount / topple windows
- area transitions
- sharpness maintenance
- buff duration
- seed and powder uptime

Expected damage can be calculated over time.

Example:

```
Total_Damage = SUM(DPS_window × duration_window)
```

This would move the project closer to a full combat simulator.

---

## Sharpness Uptime

The current MVP uses fixed sharpness multipliers.

Future model can add sharpness durability.

Potential variables:

- sharpness color
- sharpness hits available
- Handicraft effect
- Master's Touch
- Razor Sharp
- Protective Polish
- whetstone time loss

Possible output:

```
expected_sharpness_multiplier_over_time
```

This would improve long-fight DPS modeling.

---

## Skill Cost and Opportunity Cost

The current model treats each skill level as a comparable next point.

Future model can include cost.

Possible cost models:

- decoration slot level
- armor piece opportunity cost
- charm opportunity cost
- set bonus requirement
- skill tax
- build availability

Possible metric:

```
weighted_gain = DPS_gain / cost
```

This would help distinguish high-value but expensive skills from efficient low-cost skills.

---

## Better Visualization

Future dashboard improvements:

- scenario comparison table
- before / after radar chart
- skill gain waterfall chart
- cumulative investment curve
- marginal investment curve
- affinity saturation visualization
- hitzone sensitivity chart
- coverage sensitivity chart

Example chart ideas:

```
Weakness Exploit gain vs H_phys
Critical Boost gain vs CR_eff
Agitator gain vs rage coverage
Focus gain vs charge_count
```

These charts can improve portfolio storytelling.

---

## Automated Validation

Future validation can be automated.

Possible validation tests:

- Focus gain is 0 when charge_count = 0
- Quick Sheath gain is 0 when sheathe_count = 0
- Weakness Exploit gain is 0 when H_phys < 45
- Element Attack gain is 0 when weapon_ele = 0
- ECM < 1 when CR_eff < 0
- affinity is capped at 100%

In Python, these could become unit tests.

---

## In-Game Training Area Validation

The current model can be compared with training area damage tests.

Validation targets:

- displayed damage
- non-critical hit
- critical hit
- negative critical hit
- buffed attack
- elemental contribution
- Focus timing
- Quick Sheath timing

This would improve credibility.

---

## Data Pipeline Improvements

Future data pipeline improvements:

- standardized CSV schema
- automated export from workbook
- script-based data validation
- versioned data tables
- data dictionary
- formula regression tests

This would make the repository more maintainable.

---

## Portfolio Expansion

The current project can be part of a larger portfolio.

Possible related modules:

- Great Sword stat budget analysis
- weapon outlier detection
- EFR visualization
- rarity-based weapon value regression
- build efficiency comparison
- monster hitzone sensitivity study

This would show both tool-building and analytical ability.

---

## Priority Recommendation

Suggested next steps after the current MVP:

1. Finish GitHub documentation.
2. Export workbook, PDF, CSV, and screenshots.
3. Add validation results for 3–5 test cases.
4. Add root README.
5. Record a short project walkthrough.
6. Only then consider Python or Streamlit.

The immediate next goal should be portfolio presentation, not feature expansion.

---

## Summary

The current Excel MVP is already enough to demonstrate the core design idea:

```
Skill value is state-dependent and can be evaluated through marginal DPS gain.
```

Future work should improve:

- precision
- scope
- automation
- usability
- validation

But the main portfolio value is already present in the current system:

```
data model → formula model → recommendation logic → dashboard explanation
```