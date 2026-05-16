# Formula Notes

This document explains the core formulas used by the Great Sword Skill Priority Recommender.

The workbook is designed as an Excel / Google Sheets MVP. The formulas use continuous expected damage and combo-level aggregation. The purpose is to evaluate skill marginal value, not to exactly reproduce every in-game damage rounding step.

---

## 1. Global Frame Assumption

Monster Hunter: World Iceborne runs at 30 frames per second for the purpose of this model.

```
1 second = 30 frames
```

DPS is therefore calculated as:

```
DPS = D_total × 30 / frames_eff
```

---

## 2. Weapon Attack

The model uses true raw attack instead of displayed attack.

```
W_base = Display_ATK / weapon_coef
```

For Great Sword:

```
weapon_coef = 4.8
```

Example:

```
Display_ATK = 1632
weapon_coef = 4.8

W_base = 1632 / 4.8 = 340
```

The damage model always uses `W_base`, not displayed attack.

---

## 3. Effective Attack

Effective attack combines weapon true raw, additive buffs, additive skill bonuses, and attack multipliers.

```
ATK_eff = (W_base + buff_atk_plus + skill_atk_plus) × ATK_multi_eff
```

Where:

| Variable | Meaning |
| --- | --- |
| W_base | Weapon true raw |
| buff_atk_plus | Additive attack from food, charm, claw, drugs, seeds, powders, etc. |
| skill_atk_plus | Expected additive attack from skills |
| ATK_multi_eff | Expected attack multiplier |

---

## 4. Buff Attack

The MVP uses additive attack buffs.

Example buff values:

| Buff | ATK |
| --- | --- |
| --- | ---: |
| Attack Up Food L | +15 |
| Powercharm | +6 |
| Powertalon | +9 |
| Demondrug | +5 |
| Mega Demondrug | +7 |
| Might Seed | +10 |
| Demon Powder | +10 |
| Felyne Booster | +9 |

Example formula:

```
buff_atk_plus =
IF(food_l, 15, 0)
+ IF(powercharm, 6, 0)
+ IF(powertalon, 9, 0)
+ IF(demondrug, 5, 0)
+ IF(mega_demondrug, 7, 0)
+ IF(might_seed, 10, 0)
+ IF(demon_powder, 10, 0)
+ IF(felyne_booster, 9, 0)
```

---

## 5. Skill Attack Additives

Additive attack skills are added through expected coverage.

```
skill_atk_plus =
AttackBoost_ATK
+ Agitator_ATK × agitator_coverage
+ PeakPerformance_ATK × peak_coverage
+ Resentment_ATK × resentment_coverage
```

Example:

```
Agitator_ATK_expected = Agitator_ATK × agitator_coverage
```

If Agitator level 5 gives +20 attack and rage coverage is 75%:

```
Agitator_ATK_expected = 20 × 75% = 15
```

---

## 6. Attack Multipliers

Some skills can provide attack multipliers, such as Heroics or Offensive Guard.

For a multiplier skill with coverage, the expected multiplier should not be calculated as:

```
ATK_multi × coverage
```

That would incorrectly reduce attack below normal when coverage is less than 100%.

The correct coverage approximation is:

```
ATK_multi_eff = 1 + (ATK_multi - 1) × coverage
```

Example:

```
Offensive Guard multiplier = 1.15
coverage = 50%

ATK_multi_eff = 1 + (1.15 - 1) × 50%
ATK_multi_eff = 1.075
```

If multiple multiplier skills are enabled, they can be combined multiplicatively:

```
ATK_multi_eff = PRODUCT(1 + (ATK_multi_i - 1) × coverage_i)
```

In the MVP dashboard, high-condition multiplier skills are kept in the Skills table but are not included in the default recommendation set.

---

## 7. Effective Affinity

Effective affinity combines weapon affinity and skill-based affinity bonuses.

```
CR_eff = weapon_crt + CRT_plus
```

Where `CRT_plus` includes:

```
AttackBoost_CRT
+ CriticalEye_CRT
+ WeaknessExploit_CRT_expected
+ Agitator_CRT × agitator_coverage
+ LatentPower_CRT × latent_power_coverage
+ MaximumMight_CRT × maximum_might_coverage
```

The affinity used for expected critical calculation is capped:

```
CR_eff_capped = MAX(-100%, MIN(100%, CR_eff))
```

This allows the model to support both affinity overflow and negative affinity.

---

## 8. Weakness Exploit Logic

Weakness Exploit depends on hitzone state.

If the part is not tenderized:

```
H_phys = H_cut
```

If the part is tenderized:

```
H_phys = H_soft_cut
```

Weakness Exploit can trigger only when:

```
H_phys >= 45
```

Expected Weakness Exploit affinity is:

```
WEX_CRT_expected = WEX_CRT × wex_coverage × weak_ok
```

Where:

```
weak_ok = TRUE if H_phys >= 45
weak_ok = FALSE if H_phys < 45
```

If `weak_ok` is false, Weakness Exploit gain should be 0.

---

## 9. Tenderized Hitzone

Tenderized raw hitzone is calculated as:

```
H_soft = ROUNDDOWN(0.75 × H_raw + 25, 0)
```

Example:

```
H_raw = 35

H_soft = ROUNDDOWN(0.75 × 35 + 25, 0)
H_soft = ROUNDDOWN(51.25, 0)
H_soft = 51
```

This can change Weakness Exploit from inactive to active.

Elemental hitzone does not use tenderizing in this MVP.

---

## 10. Critical Damage

The model uses `CD` as the critical damage multiplier.

| Critical Boost Level | CD |
| --- | --- |
| ---: | ---: |
| 0 | 1.25 |
| 1 | 1.30 |
| 2 | 1.35 |
| 3 | 1.40 |

Important:

```
Critical Boost level 0 should use CD = 1.25, not 1.00.
```

The base positive critical hit multiplier in MHW is 1.25.

---

## 11. Expected Critical Multiplier

ECM stands for Expected Critical Multiplier.

It represents the average physical damage multiplier caused by affinity and critical damage.

For positive affinity:

```
ECM = 1 + CR_eff_capped × (CD - 1)
```

For negative affinity:

```
ECM = 1 + CR_eff_capped × 0.25
```

Negative affinity is supported because negative critical hits deal 0.75x physical damage.

Since 0.75x damage means a 25% loss:

```
negative_critical_loss = 0.25
```

Example:

```
CR_eff_capped = -8.75%

ECM = 1 + (-8.75%) × 0.25
ECM = 1 - 2.1875%
ECM = 0.978125
```

This means the current affinity state reduces expected physical damage to 97.8125%.

---

## 12. Physical Damage

The combo-level expected physical damage is:

```
D_phys = MV_total / 100 × ATK_eff × S_phy × H_phys / 100 × ECM × M_sweet
```

Where:

| Variable | Meaning |
| --- | --- |
| MV_total | Total physical motion value of the combo |
| ATK_eff | Effective true raw |
| S_phy | Physical sharpness multiplier |
| H_phys | Current physical hitzone |
| ECM | Expected Critical Multiplier |
| M_sweet | Sweet spot modifier |

Example structure:

```
D_phys =
MV_total / 100
× ATK_eff
× S_phy
× H_phys / 100
× ECM
× M_sweet
```

---

## 13. Elemental Damage

The combo-level expected elemental damage is:

```
D_ele = MV_ele_total × ELE_eff / 10 × S_ele × H_dragon / 100
```

Where:

| Variable | Meaning |
| --- | --- |
| MV_ele_total | Total elemental motion value of the combo |
| ELE_eff | Effective element value |
| S_ele | Elemental sharpness multiplier |
| H_dragon | Dragon elemental hitzone |

The MVP uses dragon element for the current Great Sword test cases.

---

## 14. Effective Element

The MVP supports elemental attack skills.

```
ELE_eff = (weapon_ele + ELE_plus) × ELE_multi
```

However, if the weapon has no element:

```
If weapon_ele = 0, then ELE_eff = 0
```

This prevents Element Attack from creating element on a non-elemental weapon.

Example:

```
IF weapon_ele = 0:
    ELE_eff = 0
ELSE:
    ELE_eff = (weapon_ele + ELE_plus) × ELE_multi
```

---

## 15. Total Damage

Total expected combo damage is:

```
D_total = D_phys + D_ele
```

The MVP does not apply per-hit flooring.

This is intentional because the model uses combo-level aggregation rather than per-hit simulation.

---

## 16. Effective Frames

Effective frames are calculated from base combo frames, Focus frame reduction, and Quick Sheath frame reduction.

```
frames_eff =
frames_base
- charge_count × focus_reduce_per_charge
- sheathe_count × sheathe_reduce_frames
```

---

## 17. Focus Frame Reduction

Focus is stored as a percentage in the Skills table.

| Focus Level | focus_reduce_pct |
| --- | --- |
| ---: | ---: |
| 0 | 0% |
| 1 | 5% |
| 2 | 10% |
| 3 | 15% |

For Great Sword, the charge frame value currently used is:

```
charge_frame_per_charge = 71
```

The per-charge Focus reduction is:

```
focus_reduce_per_charge = ROUNDDOWN(charge_frame_per_charge × focus_reduce_pct, 0)
```

Example:

```
Focus 2 = 10%

focus_reduce_per_charge = ROUNDDOWN(71 × 10%, 0)
focus_reduce_per_charge = ROUNDDOWN(7.1, 0)
focus_reduce_per_charge = 7
```

Focus changes DPS by reducing frames.

Focus does not change single-hit damage.

---

## 18. Quick Sheath Frame Reduction

Quick Sheath is stored as direct frame reduction.

| Quick Sheath Level | sheathe_reduce_frames |
| --- | --- |
| ---: | ---: |
| 0 | 0 |
| 1 | 1 |
| 2 | 3 |
| 3 | 6 |

Quick Sheath frame reduction is:

```
sheathe_frame_reduction = sheathe_count × sheathe_reduce_frames
```

Quick Sheath changes DPS by reducing frames.

Quick Sheath does not change single-hit damage.

---

## 19. DPS

The final DPS formula is:

```
DPS = D_total × 30 / frames_eff
```

Where:

```
D_total = D_phys + D_ele
```

The multiplier 30 converts frame-based combo duration into seconds.

---

## 20. Skill Marginal Gain

For each candidate skill, the recommender simulates increasing only that skill by one level.

```
gain = DPS_new / DPS_current - 1
```

Where:

| Variable | Meaning |
| --- | --- |
| DPS_current | DPS under current Input state |
| DPS_new | DPS after one candidate skill gains one level |
| gain | Marginal DPS gain |

The recommendation table sorts candidates by `gain` in descending order.

---

## 21. Future Skill Investment Curve

The future investment curve evaluates what happens if a skill receives more than one additional level.

For example, if current Attack Boost is level 3:

| add_level | target_level |
| --- | --- |
| ---: | ---: |
| 0 | 3 |
| 1 | 4 |
| 2 | 5 |
| 3 | 6 |
| 4 | 7 |

The plotted curve usually uses cumulative gain:

```
cumulative_gain = DPS_target_level / DPS_current - 1
```

This answers:

```
If I continue investing in this skill, how much total DPS gain can I expect from the current state?
```

---

## 22. Marginal vs Cumulative Gain

The recommender bar chart uses next-point marginal gain:

```
next_point_gain = DPS_next_level / DPS_current - 1
```

The investment curve can use cumulative gain:

```
cumulative_gain = DPS_target_level / DPS_current - 1
```

If a per-point curve is needed, the marginal gain between two future points can be calculated as:

```
per_point_gain = DPS_target_level / DPS_previous_target_level - 1
```

---

## 23. Important Model Behaviors

### Critical Boost is not a fixed-value skill

Critical Boost depends on current effective affinity.

If effective affinity is low or negative, Critical Boost has little or no value.

If effective affinity is high, Critical Boost becomes much stronger.

---

### Weakness Exploit is not always active

Weakness Exploit depends on:

- current hitzone
- tenderized state
- whether H_phys is at least 45
- Weakness Exploit coverage

If H_phys is below 45, Weakness Exploit gain should be 0.

---

### Focus only affects frame efficiency

Focus reduces charge frames.

It does not change:

- ATK_eff
- D_phys
- D_ele
- D_total

It only changes:

- frames_eff
- DPS

---

### Quick Sheath only affects frame efficiency

Quick Sheath reduces sheathing frames.

It does not change single-hit damage.

It only improves cycle DPS when `sheathe_count > 0`.

---

### Element Attack should not affect non-elemental weapons

If:

```
weapon_ele = 0
```

Then:

```
ELE_eff = 0
```

This means Element Attack gain should be 0.

---

## 24. MVP Limitations

The current formula model intentionally simplifies several details.

Current limitations:

- Uses continuous expected damage.
- Does not apply per-hit flooring.
- Uses combo-level aggregation.
- Uses coverage-rate approximation for conditional skills.
- Does not enumerate all combat states.
- Does not search armor, decorations, or slot constraints.
- Currently focuses on Great Sword.

These limitations are acceptable for the MVP because the purpose is to validate skill marginal value modeling and recommendation logic.

---

## 25. Future Formula Improvements

Possible future improvements:

- Per-hit physical and elemental damage floor.
- Per-hit critical simulation.
- State enumeration for conditional skills.
- Separate tenderized and non-tenderized uptime.
- Sharpness consumption modeling.
- Multi-part hit distribution.
- Armor skill and decoration constraint solver.
- Weapon-type-specific Focus and animation models.