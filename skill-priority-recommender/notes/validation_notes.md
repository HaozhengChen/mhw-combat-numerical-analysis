# Validation Notes

This document records the validation scenarios used for the Great Sword Skill Priority Recommender.

The goal of validation is not to prove perfect in-game damage reproduction. The current workbook is an Excel / Google Sheets MVP that uses continuous expected damage and combo-level aggregation.

The goal is to verify that the model behaves correctly under important design cases:

- negative affinity
- Weakness Exploit trigger logic
- tenderized hitzones
- Focus frame reduction
- Quick Sheath frame reduction
- elemental skill behavior
- conditional skill coverage
- skill marginal gain ranking

---

## 1. Validation Principles

Each test case should check one modeling assumption.

A good validation case should have:

- a clear input setup
- an expected behavior
- an actual result from the workbook
- a pass / fail judgment
- a short explanation

Recommended validation table format:

| Field | Meaning |
| --- | --- |
| Case ID | Unique test case name |
| Purpose | What the case validates |
| Input Setup | Key workbook inputs |
| Expected Result | What should happen if the model is correct |
| Actual Result | Observed workbook output |
| Status | Pass / Fail |
| Notes | Explanation or follow-up |

---

## 2. Case 1: Fatalis Blade Standard Scenario

### Purpose

Validate the main recommendation flow under a realistic Great Sword test state.

This case checks:

- negative affinity handling
- Attack Boost marginal value
- Critical Eye marginal value
- Focus marginal value
- Critical Boost dependency on effective affinity

### Input Setup

```
weapon_id = GS_P2
weapon_name = 黑龙玄刃
combo_id = COMBO_SLINGER_TCS
combo_name = 直斩投射器真蓄
hitzone_id = TRAINING_POLE
tenderized = FALSE
sweet_spot_modifier = 1.03
```

Example skill state:

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

Example coverage state:

```
弱特覆盖率 = 75%
挑战者覆盖率 = 75%
无伤覆盖率 = 50%
怨恨覆盖率 = 0%
力量解放覆盖率 = 0%
精神抖擞覆盖率 = 0%
```

### Expected Result

Attack Boost 3→4 should rank highly because this level provides both:

- additive attack
- +5% affinity

Critical Eye should also have meaningful value because Fatalis Blade starts with negative affinity.

Critical Boost should have low or zero value if effective affinity is still negative or close to zero.

Focus should provide value if the selected combo contains charge segments.

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

This case confirms that Attack Boost 3→4 is correctly treated as a breakpoint level because it adds affinity in addition to raw attack.

It also confirms that Critical Boost is not treated as a fixed-value skill. Its value depends on current effective affinity.

---

## 3. Case 2: Draw Slash Combo and Focus

### Purpose

Validate that Focus only affects combos with charge segments.

Focus should not improve DPS if the selected combo has no charge segment.

### Input Setup

```
combo_id = COMBO_DRAW_SLASH
combo_name = 蹭刀
charge_count = 0
```

Keep other inputs unchanged from the standard test case.

### Expected Result

Focus gain should be 0 or near 0.

The formula should behave as:

```
frames_eff = frames_base - charge_count × focus_reduce_per_charge - sheathe_count × sheathe_reduce_frames
```

Since:

```
charge_count = 0
```

The Focus term becomes:

```
0 × focus_reduce_per_charge = 0
```

### Status

```
PASS if Focus gain = 0 or near 0
```

### Notes

This case validates that Focus is modeled as a frame-efficiency skill rather than a damage skill.

Focus should not change:

- ATK_eff
- D_phys
- D_ele
- D_total

It should only change:

- frames_eff
- DPS

---

## 4. Case 3: Quick Sheath and Sheathe Count

### Purpose

Validate that Quick Sheath only affects combos with sheathing actions.

### Input Setup

Use a combo with:

```
sheathe_count = 1
```

Then compare with a hypothetical or future combo where:

```
sheathe_count = 0
```

### Expected Result

If `sheathe_count = 1`, Quick Sheath should improve DPS by reducing effective frames.

If `sheathe_count = 0`, Quick Sheath gain should be 0.

The formula should behave as:

```
sheathe_frame_reduction = sheathe_count × sheathe_reduce_frames
```

### Status

```
PASS if Quick Sheath only has gain when sheathe_count > 0
```

### Notes

Quick Sheath should not change single-hit damage.

It only improves cycle DPS by reducing sheathing frames.

---

## 5. Case 4: Low Hitzone and Weakness Exploit

### Purpose

Validate that Weakness Exploit does not activate when the physical hitzone is below 45.

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

Expected behavior:

```
weak_ok = FALSE
Weakness Exploit gain = 0
```

### Status

```
PASS if Weakness Exploit gain = 0
```

### Notes

This case validates that Weakness Exploit is not treated as unconditional affinity.

Its value depends on the current hitzone.

---

## 6. Case 5: Tenderized Neck and Weakness Exploit

### Purpose

Validate that tenderizing can change Weakness Exploit from inactive to active.

### Input Setup A: Untenderized Neck

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

### Input Setup B: Tenderized Neck

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
PASS if Weakness Exploit gain increases after tenderizing
```

### Notes

This is one of the most important design validation cases.

It demonstrates this chain:

```
Tenderizing → higher hitzone → WEX trigger changes → skill priority changes
```

This is a strong portfolio example because it shows that skill value is context-dependent.

---

## 7. Case 6: Non-elemental Weapon and Element Attack

### Purpose

Validate that Element Attack does not create elemental damage on a non-elemental weapon.

### Input Setup

```
weapon_id = GS_P1
weapon_name = 防卫队炎刃5
weapon_ele = 0
```

### Expected Result

The formula should force:

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
PASS if Element Attack gain = 0
```

### Notes

This case prevents an important modeling bug.

Element Attack should only improve weapons that already have matching element.

---

## 8. Case 7: Positive Affinity and Critical Boost

### Purpose

Validate that Critical Boost becomes valuable when effective affinity is high.

### Input Setup

Use a state with high effective affinity, for example:

```
CR_eff > 50%
```

This can be achieved through:

- Critical Eye
- Weakness Exploit
- Agitator
- Attack Boost level 4+
- positive coverage assumptions

### Expected Result

Critical Boost should gain value as effective affinity increases.

Expected relationship:

```
Higher CR_eff → higher Critical Boost value
```

### Status

```
PASS if Critical Boost ranks higher under high-affinity states
```

### Notes

This confirms that Critical Boost is correctly modeled through ECM.

Critical Boost should not be a fixed-value skill.

---

## 9. Case 8: Negative Affinity and ECM

### Purpose

Validate that negative affinity reduces expected physical damage.

### Input Setup

Use Fatalis Blade with insufficient affinity compensation:

```
weapon_crt = -30%
CR_eff < 0
```

Example:

```
CR_eff = -8.75%
```

### Expected Result

The negative affinity formula should be used:

```
ECM = 1 + CR_eff × 0.25
```

Example:

```
ECM = 1 + (-8.75%) × 0.25
ECM = 0.978125
```

### Status

```
PASS if ECM < 1 when CR_eff < 0
```

### Notes

This case validates that negative affinity is not incorrectly clamped to 0 before ECM.

---

## 10. Case 9: Affinity Cap

### Purpose

Validate that affinity is capped between -100% and 100%.

### Input Setup

Create a high-affinity state, for example:

```
CR_eff > 100%
```

### Expected Result

The model should calculate:

```
CR_eff_capped = 100%
```

ECM should use `CR_eff_capped`, not raw `CR_eff`.

### Status

```
PASS if ECM uses capped affinity
```

### Notes

This prevents inflated value from overcapping affinity.

This also helps explain why additional affinity skills lose value near or above 100% affinity.

---

## 11. Case 10: Coverage Rate Sensitivity

### Purpose

Validate that conditional skills respond to coverage rates.

### Input Setup

Test a conditional skill such as Agitator.

Compare:

```
agitator_coverage = 0%
agitator_coverage = 50%
agitator_coverage = 100%
```

### Expected Result

Agitator gain should increase as coverage increases.

Expected relationship:

```
0% coverage → no gain
50% coverage → partial gain
100% coverage → full gain
```

### Status

```
PASS if conditional skill gain scales with coverage
```

### Notes

This validates the MVP's coverage-rate approximation model.

---

## 12. Validation Summary Table

Recommended summary table:

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

## 13. Portfolio Interpretation

The validation cases are designed to show that the recommender is not using fixed skill values.

Instead, skill priority changes based on:

- weapon stats
- current affinity
- selected combo
- charge count
- sheathe count
- hitzone
- tenderized state
- conditional coverage
- current skill levels

This is the core design value of the Skill Priority Recommender.

---

## 14. Known Validation Limits

The current validation does not prove exact in-game damage parity.

Known limits:

- no per-hit flooring
- no per-hit critical roll simulation
- no sharpness consumption
- no monster state timeline
- no full armor or decoration constraints
- no multi-part hit distribution

These are acceptable for the MVP stage.

The validation focuses on whether the model logic and skill priority behavior are directionally correct.

---

## 15. Recommended Next Validation Steps

Future validation improvements:

- Compare selected cases against training area damage tests.
- Add per-hit damage floor validation.
- Add separate validation for physical and elemental damage components.
- Add state-enumeration tests for conditional skills.
- Add more monster hitzones.
- Add more combo variants.
- Add weapon-type-specific validation cases.