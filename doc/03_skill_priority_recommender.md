# Skill Priority Recommender

This document explains the recommendation logic used by the Great Sword Skill Priority Recommender.

The recommender calculates the marginal DPS gain of upgrading each candidate skill by one level under the current combat state.

---

## Core Question

The recommender answers:

```
Given the current combat state, which skill level provides the highest next-point DPS gain?
```

It does not ask:

```
Which skill is generally best?
```

This distinction is important because skill value is context-dependent.

---

## Why Static Skill Priority Is Not Enough

In Monster Hunter: World Iceborne, skill value changes based on:

- current weapon
- current affinity
- current skill levels
- selected combo
- hitzone
- tenderized state
- buff state
- conditional coverage
- frame efficiency
- elemental contribution

Examples:

- Critical Boost is weak when affinity is low.
- Critical Boost is strong when affinity is high.
- Weakness Exploit is useless if the hitzone does not meet the threshold.
- Focus is useful only when the combo contains charge frames.
- Quick Sheath is useful only when the combo contains sheathing frames.
- Element Attack has no value on a non-elemental weapon.

Therefore, the recommender calculates value dynamically.

---

## Recommendation Algorithm

For each candidate skill:

1. Read the current skill level.
2. Check the maximum skill level.
3. If the skill is already maxed, mark it invalid.
4. Simulate only that skill increasing by one level.
5. Keep all other skills and inputs unchanged.
6. Recalculate effective attack, affinity, ECM, frames, damage, and DPS.
7. Calculate marginal gain.
8. Sort all valid skills by gain.

Core gain formula:

```
gain = DPS_new / DPS_current - 1
```

---

## Candidate Skill Table

The MVP default recommendation set includes:

| Skill | Max Level | Main Effect |
| --- | --- | --- |
| --- | ---: | --- |
| 攻击 | 7 | Raw attack and affinity from level 4 onward |
| 看破 | 7 | Affinity |
| 弱点特效 | 3 | Conditional affinity |
| 超会心 | 3 | Critical damage multiplier |
| 挑战者 | 7 | Raw attack and affinity under rage coverage |
| 无伤 | 3 | Raw attack under full-health coverage |
| 怨恨 | 5 | Raw attack under red-health coverage |
| 集中 | 3 | Charge frame reduction |
| 纳刀术 | 3 | Sheathing frame reduction |
| 属性强化 | 6 | Element value increase |

High-condition skills such as Heroics, Fortify, Offensive Guard, and Coalescence may remain in the Skills table but are excluded from the default recommendation set.

---

## Candidate Validity

A candidate skill is valid if:

```
next_level <= max_level
```

And:

```
include_in_recommend = TRUE
```

If a skill is already maxed, it should not appear in the recommendation ranking.

---

## Simulation Logic

Each row in the recommendation table represents one simulation.

Example:

```
Current state:
攻击 = 3
看破 = 2
弱点特效 = 1
超会心 = 1
挑战者 = 3
```

For the `攻击` row:

```
sim_attack = 4
sim_crit_eye = 2
sim_wex = 1
sim_crit_boost = 1
sim_agitator = 3
```

For the `看破` row:

```
sim_attack = 3
sim_crit_eye = 3
sim_wex = 1
sim_crit_boost = 1
sim_agitator = 3
```

Only one skill changes per row.

---

## Recalculation Per Candidate

For each candidate row, the model recalculates:

- ATK_eff_new
- CR_eff_new
- CR_eff_capped_new
- CD_new
- ECM_new
- frames_eff_new
- ELE_eff_new
- D_phys_new
- D_ele_new
- D_total_new
- DPS_new
- gain

The same formula structure used by `Calc_Current` is reused with simulated skill levels.

---

## Attack Skill Behavior

Attack Boost affects the model through:

```
ATK_plus
CRT_plus
```

Attack Boost level 4 is especially important because it adds affinity in addition to raw attack.

This can make:

```
攻击 3→4
```

rank higher than surrounding Attack Boost levels.

---

## Affinity Skill Behavior

Affinity skills affect:

```
CR_eff_new
```

Then they affect physical damage through:

```
ECM_new
```

Affinity skills include:

- 看破
- 攻击 level 4+
- 弱点特效
- 挑战者
- 力量解放
- 精神抖擞

Affinity value decreases when current affinity approaches or exceeds 100%.

---

## Critical Boost Behavior

Critical Boost affects:

```
CD_new
```

Then CD affects:

```
ECM_new
```

Critical Boost value depends on effective affinity.

If:

```
CR_eff_new <= 0
```

Critical Boost has little or no value.

If:

```
CR_eff_new is high
```

Critical Boost becomes much stronger.

This is why Critical Boost should not be modeled as a fixed percentage gain.

---

## Weakness Exploit Behavior

Weakness Exploit depends on:

- physical hitzone
- tenderized state
- Weakness Exploit coverage

It can trigger only if:

```
H_phys >= 45
```

If the hitzone condition is not met:

```
Weakness Exploit gain = 0
```

This allows the recommender to show different skill priorities on different monster parts.

---

## Agitator Behavior

Agitator affects both:

```
ATK_plus
CRT_plus
```

Both are multiplied by rage coverage:

```
Agitator_ATK_expected = Agitator_ATK × agitator_coverage
Agitator_CRT_expected = Agitator_CRT × agitator_coverage
```

Therefore, Agitator value changes with the assumed monster rage uptime.

---

## Peak Performance and Resentment

Peak Performance uses full-health coverage:

```
Peak_ATK_expected = Peak_ATK × peak_coverage
```

Resentment uses red-health coverage:

```
Resentment_ATK_expected = Resentment_ATK × resentment_coverage
```

If coverage is 0%, the skill should have 0 gain.

---

## Focus Behavior

Focus does not change damage.

It changes:

```
frames_eff_new
```

Focus reduction:

```
focus_reduce_per_charge = ROUNDDOWN(charge_frame_per_charge × focus_reduce_pct, 0)
```

Effective frames:

```
frames_eff_new =
frames_base
- charge_count × focus_reduce_per_charge
- sheathe_count × sheathe_reduce_frames
```

If:

```
charge_count = 0
```

Focus gain should be 0.

---

## Quick Sheath Behavior

Quick Sheath does not change damage.

It changes:

```
frames_eff_new
```

Quick Sheath reduction:

```
sheathe_frame_reduction = sheathe_count × sheathe_reduce_frames
```

If:

```
sheathe_count = 0
```

Quick Sheath gain should be 0.

---

## Element Attack Behavior

Element Attack affects:

```
ELE_eff_new
```

Formula:

```
ELE_eff_new = (weapon_ele + ELE_plus) × ELE_multi
```

If:

```
weapon_ele = 0
```

Then:

```
ELE_eff_new = 0
```

This prevents Element Attack from creating element on a non-elemental weapon.

---

## Sorting

The final recommendation ranking sorts by:

```
gain descending
```

Recommended display columns:

| Column | Meaning |
| --- | --- |
| Rank | Sorted rank |
| Skill Upgrade | Candidate level change |
| New DPS | DPS after upgrade |
| Gain | Marginal DPS gain |
| Reason | Short explanation |

---

## Example Output

Example:

| Rank | Skill Upgrade | Gain | Interpretation |
| --- | --- | --- | --- |
| ---: | --- | ---: | --- |
| 1 | 攻击 3→4 | +1.96% | Adds raw and affinity |
| 2 | 看破 2→3 | +1.26% | Offsets negative affinity |
| 3 | 集中 0→1 | +1.02% | Reduces charge frames |
| 4 | 弱点特效 1→2 | +0.75% | Current hitzone can trigger WEX |
| 5 | 挑战者 3→4 | +0.69% | Adds raw and affinity under coverage |

---

## Future Skill Investment Curve

The recommender also supports a future investment curve.

Instead of simulating only +1 level, it simulates:

```
current level + 0
current level + 1
current level + 2
...
up to max level
```

The curve uses cumulative gain:

```
cumulative_gain = DPS_target_level / DPS_current - 1
```

This helps answer:

```
If I continue investing in this skill, how much more value can I expect?
```

---

## Marginal vs Cumulative

The bar chart uses next-point marginal gain:

```
next_point_gain = DPS_next_level / DPS_current - 1
```

The investment curve usually uses cumulative gain:

```
cumulative_gain = DPS_target_level / DPS_current - 1
```

If needed, per-point future marginal gain can be calculated as:

```
future_point_gain = DPS_target_level / DPS_previous_target_level - 1
```

---

## Why This Is a Recommender

The model is a recommender because it evaluates alternatives under the current state and ranks them.

It is not a recommender because it uses machine learning.

The recommendation is deterministic and formula-based.

This is appropriate for a numerical design tool because:

- the system is explainable
- every recommendation can be traced to formula components
- the result changes dynamically with player inputs
- the model can be validated by test cases

---

## Design Value

The recommender demonstrates:

- dynamic skill valuation
- conditional effect modeling
- breakpoint handling
- frame-based DPS modeling
- negative affinity support
- hitzone-dependent recommendation
- coverage-rate approximation
- clear dashboard communication

This is the core design value of the project.