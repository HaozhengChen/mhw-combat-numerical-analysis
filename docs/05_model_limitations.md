# Model Limitations

This document explains the current limitations of the Great Sword Skill Priority Recommender.

The model is designed as an Excel / Google Sheets MVP for numerical design portfolio purposes. Its goal is to evaluate skill marginal value and recommendation logic, not to fully replicate every in-game damage detail.

---

## MVP Scope

The current MVP focuses on:

- Great Sword scenarios
- combo-level expected damage
- skill marginal DPS gain
- conditional skill coverage approximation
- dashboard presentation
- explainable recommendation logic

The current MVP does not attempt to be a full combat simulator or armor optimizer.

---

## Continuous Expected Damage

The model uses continuous expected damage.

For example:

```
D_phys = MV_total / 100 × ATK_eff × S_phy × H_phys / 100 × ECM × M_sweet
```

This means the model does not apply per-hit integer flooring at each damage step.

### Limitation

In-game displayed damage is rounded or floored at specific steps.

The MVP may therefore differ slightly from exact in-game numbers.

### Why acceptable for MVP

The goal is to compare relative skill value.

Continuous expected damage is usually sufficient for directional marginal gain comparison.

---

## Combo-Level Aggregation

The model uses combo-level aggregated values:

```
MV_total
MV_ele_total
frames_base
```

### Limitation

It does not calculate each hit separately.

This means it cannot model:

- per-hit flooring
- per-hit critical randomness
- different hitzones per hit
- partial combo whiffs
- hit-by-hit sharpness loss
- multi-hit distribution by body part

### Why acceptable for MVP

The selected Great Sword combos are simple enough for a first-pass expected DPS model.

Combo-level aggregation keeps the Excel model understandable.

---

## No Per-Hit Flooring

The model does not apply per-hit damage flooring.

### Limitation

Exact in-game damage may differ from the workbook.

For exact parity, the model would need to calculate each hit separately:

```
physical_hit_damage = FLOOR(...)
elemental_hit_damage = FLOOR(...)
displayed_hit_damage = physical_hit_damage + elemental_hit_damage
```

### Future improvement

A future version can add a hit-level table and apply floor operations per hit.

---

## Coverage-Rate Approximation

Conditional skills are modeled using coverage rates.

Examples:

```
Agitator_ATK_expected = Agitator_ATK × agitator_coverage
Peak_ATK_expected = Peak_ATK × peak_coverage
Resentment_ATK_expected = Resentment_ATK × resentment_coverage
```

### Limitation

This assumes the skill effect can be averaged linearly.

It does not model a full timeline of combat states.

### Why acceptable for MVP

Coverage-rate approximation is easy to understand and easy to adjust.

It is useful for a portfolio MVP because it shows how conditional effects enter expected value calculations.

---

## No Full State Enumeration

The model does not enumerate all possible combat states.

For example, it does not separately calculate:

```
rage on / off
tenderized / not tenderized
full health / not full health
red health / no red health
latent power active / inactive
maximum might active / inactive
```

### Limitation

Interactions between multiple conditional states are simplified.

### Future improvement

A future model can enumerate states and calculate weighted expected DPS:

```
Expected_DPS = SUM(DPS_state × state_probability)
```

---

## Limited Weapon Scope

The current MVP focuses primarily on Great Sword.

### Limitation

Other weapons may require different modeling for:

- combo structure
- hit count
- animation timing
- Focus effect
- sharpness consumption
- elemental scaling
- phial, shelling, ammo, or gauge systems

### Future improvement

Add weapon-specific model modules.

---

## Focus Model Simplification

Focus is modeled as:

```
focus_reduce_per_charge = ROUNDDOWN(charge_frame_per_charge × focus_reduce_pct, 0)
```

For current Great Sword tests:

```
charge_frame_per_charge = 71
```

### Limitation

This assumes charge segments can be represented by a shared charge frame value.

More precise modeling would calculate Focus reduction per move.

### Future improvement

Move Focus calculation from combo-level to move-level.

---

## Quick Sheath Model Simplification

Quick Sheath is modeled as direct frame reduction:

```
sheathe_frame_reduction = sheathe_count × sheathe_reduce_frames
```

### Limitation

This assumes each sheathing action has the same reduction value.

Actual animation timing may vary by context.

### Future improvement

Use move-level or animation-level sheathing frame data.

---

## Elemental Damage Simplification

Elemental damage is calculated at combo level:

```
D_ele = MV_ele_total × ELE_eff / 10 × S_ele × H_dragon / 100
```

### Limitation

The model does not apply per-hit elemental flooring.

It also currently focuses on dragon element for Great Sword examples.

### Future improvement

Add per-hit elemental calculation and support all element types.

---

## Sharpness Consumption Not Modeled

The model uses the current sharpness multiplier:

```
S_phy
S_ele
```

### Limitation

It does not model sharpness loss over time.

It does not evaluate how long a weapon can maintain purple or white sharpness.

### Future improvement

Add sharpness durability and expected uptime.

---

## No Armor or Decoration Constraints

The model recommends skill priority, not actual equipment sets.

### Limitation

It does not consider:

- armor pieces
- decoration slots
- charm levels
- set bonuses
- skill availability
- opportunity cost by slot size

### Why acceptable for MVP

The project is a Skill Priority Recommender, not an Armor Optimizer.

It answers:

```
Which skill point is most valuable under the current state?
```

It does not answer:

```
Which exact armor set should I wear?
```

---

## No Build Cost Model

The current model treats every skill level as one comparable investment step.

### Limitation

In actual builds, skill levels have different opportunity costs.

Examples:

- one level of Attack Boost may cost a level 1 slot
- Critical Boost may cost a level 2 slot
- some skills are tied to armor pieces
- set bonuses may constrain choices

### Future improvement

Add slot-weighted gain:

```
slot_efficiency = gain / slot_cost
```

---

## No Player Execution Model

The model assumes the selected combo is executed successfully.

### Limitation

It does not model:

- missed attacks
- monster movement
- player positioning
- openings
- hitzone switching
- risk of using longer combos

### Future improvement

Add combo uptime or execution probability.

---

## No Monster Timeline

The model uses static coverage inputs.

### Limitation

It does not simulate a full hunt timeline.

For example, it does not model:

- rage duration
- tenderize uptime
- knockdown windows
- enrage cycles
- area transitions

### Future improvement

Add timeline-based expected DPS.

---

## No Status or Ailment Modeling

The Skills table includes status-related columns, but the MVP does not fully model status buildup.

### Limitation

It does not evaluate:

- poison
- paralysis
- sleep
- blast
- status thresholds
- monster resistance scaling

### Future improvement

Add status accumulation and proc value modeling.

---

## Dashboard Is a Presentation Layer

The Dashboard is designed for presentation, not full analysis.

### Limitation

It only shows selected results such as:

- current DPS
- Top recommendations
- charts
- checks

Full calculations remain in underlying sheets.

### Why acceptable

This keeps the Dashboard readable for portfolio review.

---

## Known Export Limitations

When exporting Excel or Google Sheets outputs to GitHub:

- formulas may not be visible in PDF
- Excel files may not preview well online
- CSV exports lose formulas
- screenshots may become outdated after workbook changes

Recommended practice:

- keep `.xlsx` for interactive use
- keep `.pdf` for quick viewing
- export `.csv` for data table visibility
- export `.png` for README preview
- use `.md` files for explanation

---

## Why These Limitations Are Acceptable

The current MVP is designed to demonstrate:

- formula decomposition
- data modeling
- conditional effect handling
- marginal gain calculation
- recommendation ranking
- dashboard communication

It is not intended to be a final production combat simulator.

The limitations are documented to show model awareness and future planning.

---

## Summary

The current model is best understood as:

```
A transparent, explainable, Excel-based skill marginal value recommender.
```

It is not:

```
A full in-game damage replica.
```

It is not:

```
An armor optimizer.
```

It is not:

```
A complete hunt simulator.
```

This scope is intentional and appropriate for a game numerical design portfolio.