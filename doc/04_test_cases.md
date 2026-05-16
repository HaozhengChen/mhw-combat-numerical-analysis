# Test Cases

This document describes test cases for validating the Great Sword Skill Priority Recommender.

The purpose of these tests is to verify model behavior under important combat states.

The current MVP uses continuous expected damage and combo-level aggregation. The tests focus on directional correctness and model logic.

---

## Test Case Format

Each test case should include:

| Field | Meaning |
| --- | --- |
| Case ID | Unique test case name |
| Purpose | What the test validates |
| Input Setup | Key workbook inputs |
| Expected Result | Expected model behavior |
| Actual Result | Observed workbook result |
| Status | PASS / FAIL / TBD |
| Notes | Explanation |

---

## Case 1: Fatalis Blade Standard Scenario

### Purpose

Validate the main recommendation flow under a realistic Great Sword state.

This case checks:

- negative affinity handling
- Attack Boost breakpoint behavior
- Critical Eye value
- Focus value
- Critical Boost dependency on affinity

### Input Setup

```
weapon_id = GS_P2
weapon_name = 黑龙玄刃
combo_id = COMBO_SLINGER_TCS
combo_name = 直斩投射器真蓄
hitzone_id = TRAINING_POLE
target = 训练场木桩
tenderized = FALSE
sweet_spot_modifier = 1.03
```

Example skill levels:

```
攻击 = 3
看破 = 2
弱点特效 = 1
超会心 = 1
挑战者 = 3
无伤 = 1
怨恨 = 0
集中 = 0
纳刀术 = 0
属性强化 = 0
```

Example coverage:

```
弱特覆盖率 = 75%
挑战者覆盖率 = 75%
无伤覆盖率 = 50%
怨恨覆盖率 = 0%
力量解放覆盖率 = 0%
精神抖擞覆盖率 = 0%
```

### Expected Result

Attack Boost 3→4 should rank highly because it provides:

- raw attack
- +5% affinity

Critical Eye should also provide value because Fatalis Blade starts with negative affinity.

Focus should provide value because the combo contains a charge segment.

Critical Boost should have low value if effective affinity is still negative or low.

### Example Actual Result

| Rank | Skill Upgrade | Gain |
| --- | --- | --- |
| ---: | --- | ---: |
| 1 | 攻击 3→4 | +1.96% |
| 2 | 看破 2→3 | +1.26% |
| 3 | 集中 0→1 | +1.02% |
| 4 | 弱点特效 1→2 | +0.75% |
| 5 | 挑战者 3→4 | +0.69% |

### Status

```
PASS
```

### Notes

This confirms that the recommender recognizes Attack Boost 3→4 as a breakpoint.

It also confirms that Critical Boost is not treated as a fixed-value skill.

---

## Case 2: Draw Slash Focus Test

### Purpose

Validate that Focus only affects combos with charge segments.

### Input Setup

```
combo_id = COMBO_DRAW_SLASH
combo_name = 蹭刀
charge_count = 0
```

### Expected Result

Focus gain should be 0 or near 0.

Because:

```
charge_count = 0
```

The Focus term becomes:

```
charge_count × focus_reduce_per_charge = 0
```

### Status

```
TBD
```

### Notes

Focus should not change:

- ATK_eff
- D_phys
- D_ele
- D_total

It should only affect:

- frames_eff
- DPS

If there is no charge segment, Focus should have no DPS effect.

---

## Case 3: Quick Sheath Test

### Purpose

Validate that Quick Sheath only affects combos with sheathing actions.

### Input Setup A

```
sheathe_count = 1
```

### Expected Result A

Quick Sheath should improve DPS by reducing effective frames.

### Input Setup B

```
sheathe_count = 0
```

### Expected Result B

Quick Sheath gain should be 0.

### Status

```
TBD
```

### Notes

Quick Sheath should not change combo damage.

It should only affect cycle frames.

---

## Case 4: Low Hitzone Weakness Exploit Test

### Purpose

Validate that Weakness Exploit does not activate if the physical hitzone is below 45.

### Input Setup

```
hitzone_id = RATH_BODY
target = 火龙
part = 胴
tenderized = FALSE
H_cut = 25
```

### Expected Result

Since:

```
H_phys = 25
H_phys < 45
```

Weakness Exploit should not trigger.

Expected output:

```
weak_ok = FALSE
Weakness Exploit gain = 0
```

### Status

```
TBD
```

### Notes

This confirms that Weakness Exploit is not modeled as unconditional affinity.

---

## Case 5: Tenderized Neck Weakness Exploit Test

### Purpose

Validate that tenderizing can change Weakness Exploit from inactive to active.

### Input Setup A: Untenderized

```
hitzone_id = RATH_NECK
part = 颈
tenderized = FALSE
H_cut = 35
```

### Expected Result A

```
H_phys = 35
H_phys < 45
weak_ok = FALSE
Weakness Exploit gain = 0
```

### Input Setup B: Tenderized

```
hitzone_id = RATH_NECK
part = 颈
tenderized = TRUE
H_soft_cut = 51
```

### Expected Result B

```
H_phys = 51
H_phys >= 45
weak_ok = TRUE
Weakness Exploit gain > 0
```

### Status

```
TBD
```

### Notes

This test is important because it demonstrates the relationship:

```
Tenderizing → higher hitzone → Weakness Exploit trigger → changed skill priority
```

This is a strong portfolio example.

---

## Case 6: Non-elemental Weapon Element Attack Test

### Purpose

Validate that Element Attack does not create element on a non-elemental weapon.

### Input Setup

```
weapon_id = GS_P1
weapon_name = 防卫队炎刃5
weapon_ele = 0
```

### Expected Result

The model should force:

```
ELE_eff = 0
```

Therefore:

```
D_ele = 0
Element Attack gain = 0
```

### Status

```
TBD
```

### Notes

This prevents a common modeling mistake.

Element Attack should only affect weapons with matching element.

---

## Case 7: Positive Affinity Critical Boost Test

### Purpose

Validate that Critical Boost becomes valuable when effective affinity is high.

### Input Setup

Create a high-affinity state.

Example:

```
CR_eff > 50%
```

Possible ways to reach this:

- increase Critical Eye
- increase Weakness Exploit
- increase Agitator coverage
- use tenderized weak hitzone

### Expected Result

Critical Boost gain should increase as effective affinity increases.

Expected relationship:

```
Higher CR_eff → Higher Critical Boost value
```

### Status

```
TBD
```

### Notes

This confirms that Critical Boost is correctly modeled through ECM.

---

## Case 8: Negative Affinity ECM Test

### Purpose

Validate that negative affinity reduces expected physical damage.

### Input Setup

Use Fatalis Blade with insufficient affinity compensation.

Example:

```
weapon_crt = -30%
CR_eff = -8.75%
```

### Expected Result

Negative affinity formula should be used:

```
ECM = 1 + CR_eff × 0.25
```

Example:

```
ECM = 1 + (-8.75%) × 0.25
ECM = 0.978125
```

Expected behavior:

```
ECM < 1
```

### Status

```
PASS
```

### Notes

This confirms that negative affinity is not incorrectly treated as 0.

---

## Case 9: Affinity Cap Test

### Purpose

Validate that affinity is capped between -100% and 100%.

### Input Setup

Create a high-affinity state:

```
CR_eff > 100%
```

### Expected Result

The model should use:

```
CR_eff_capped = 100%
```

ECM should be calculated using `CR_eff_capped`, not raw `CR_eff`.

### Status

```
TBD
```

### Notes

This prevents inflated value from overcapped affinity.

It also explains why additional affinity loses value after reaching 100%.

---

## Case 10: Coverage Sensitivity Test

### Purpose

Validate that conditional skills respond to coverage rates.

### Input Setup

Test Agitator under three coverage settings:

```
agitator_coverage = 0%
agitator_coverage = 50%
agitator_coverage = 100%
```

### Expected Result

Agitator gain should scale with coverage.

Expected relationship:

```
0% coverage → no gain
50% coverage → partial gain
100% coverage → full gain
```

### Status

```
TBD
```

### Notes

This validates the coverage-rate approximation model.

---

## Summary Table

| Case ID | Purpose | Expected Result | Status |
| --- | --- | --- | --- |
| Case 1 | Fatalis Blade standard scenario | Attack, Critical Eye, and Focus behave reasonably | PASS |
| Case 2 | Draw Slash Focus test | Focus gain is 0 when charge_count = 0 | TBD |
| Case 3 | Quick Sheath test | Quick Sheath gain depends on sheathe_count | TBD |
| Case 4 | Low hitzone WEX test | WEX gain is 0 when H_phys < 45 | TBD |
| Case 5 | Tenderized neck WEX test | WEX gain increases after tenderizing | TBD |
| Case 6 | Non-elemental weapon test | Element Attack gain is 0 | TBD |
| Case 7 | High affinity Critical Boost test | Critical Boost value increases | TBD |
| Case 8 | Negative affinity ECM test | ECM < 1 when CR_eff < 0 | PASS |
| Case 9 | Affinity cap test | ECM uses capped affinity | TBD |
| Case 10 | Coverage sensitivity test | Conditional gain scales with coverage | TBD |

---

## Portfolio Interpretation

These tests show that the recommender is not using fixed skill values.

Skill priority changes based on:

- current weapon stats
- current affinity
- combo selection
- charge count
- sheathe count
- hitzone
- tenderized state
- skill coverage
- current skill levels

This is the main design value of the recommender.

---

## Known Validation Limits

The current validation does not prove exact in-game parity.

Known limits:

- no per-hit flooring
- no per-hit critical roll simulation
- no sharpness consumption
- no monster timeline
- no armor or decoration constraints
- no multi-part hit distribution

The current tests focus on whether the model logic is directionally correct.

---

## Future Validation Work

Future validation improvements:

- compare selected cases against training area damage tests
- add per-hit damage floor validation
- add separate physical and elemental validation
- add state enumeration tests
- add more monster hitzones
- add more combo variants
- add weapon-type-specific validation cases