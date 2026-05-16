# Damage Formula

This document explains the damage formula used in the Great Sword Skill Priority Recommender.

The model uses continuous expected damage and combo-level aggregation. It is designed for skill marginal value comparison, not exact per-hit in-game damage replication.

---

## Frame Assumption

The model uses:

```
1 second = 30 frames
```

Therefore:

```
DPS = D_total × 30 / frames_eff
```

---

## True Raw Attack

Monster Hunter displays inflated weapon attack values. The damage formula uses true raw attack.

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

The workbook uses `W_base`, not displayed attack.

---

## Effective Attack

Effective attack is:

```
ATK_eff = (W_base + buff_atk_plus + skill_atk_plus) × ATK_multi_eff
```

Where:

| Variable | Meaning |
| --- | --- |
| W_base | Weapon true raw |
| buff_atk_plus | Additive attack buffs |
| skill_atk_plus | Expected additive attack from skills |
| ATK_multi_eff | Expected attack multiplier |

---

## Buff Attack

Buff attack includes additive sources such as:

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

---

## Skill Attack Additives

Additive attack skills are calculated using coverage assumptions.

```
skill_atk_plus =
AttackBoost_ATK
+ Agitator_ATK × agitator_coverage
+ PeakPerformance_ATK × peak_coverage
+ Resentment_ATK × resentment_coverage
```

Example:

```
Agitator_ATK = 20
agitator_coverage = 75%

Agitator_ATK_expected = 20 × 75% = 15
```

---

## Attack Multipliers

Some skills provide attack multipliers.

A multiplier with coverage should use:

```
ATK_multi_eff = 1 + (ATK_multi - 1) × coverage
```

Incorrect approach:

```
ATK_multi_eff = ATK_multi × coverage
```

The incorrect approach can reduce attack below normal when coverage is less than 100%.

For multiple multiplier skills:

```
ATK_multi_eff = PRODUCT(1 + (ATK_multi_i - 1) × coverage_i)
```

---

## Effective Affinity

Effective affinity is:

```
CR_eff = weapon_crt + CRT_plus
```

`CRT_plus` includes:

```
AttackBoost_CRT
+ CriticalEye_CRT
+ WeaknessExploit_CRT_expected
+ Agitator_CRT × agitator_coverage
+ LatentPower_CRT × latent_power_coverage
+ MaximumMight_CRT × maximum_might_coverage
```

The model caps affinity:

```
CR_eff_capped = MAX(-100%, MIN(100%, CR_eff))
```

This supports both affinity overflow and negative affinity.

---

## Critical Damage

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

---

## Expected Critical Multiplier

ECM stands for Expected Critical Multiplier.

It combines affinity and critical damage into one expected physical damage multiplier.

For positive affinity:

```
ECM = 1 + CR_eff_capped × (CD - 1)
```

For negative affinity:

```
ECM = 1 + CR_eff_capped × 0.25
```

Negative affinity is supported because a negative critical hit deals 0.75x physical damage.

Example:

```
CR_eff_capped = -8.75%

ECM = 1 + (-8.75%) × 0.25
ECM = 0.978125
```

This means expected physical damage becomes 97.8125% of normal because of negative affinity.

---

## Hitzone and Tenderizing

If the part is not tenderized:

```
H_phys = H_cut
```

If the part is tenderized:

```
H_phys = H_soft_cut
```

Tenderized raw hitzone:

```
H_soft = ROUNDDOWN(0.75 × H_raw + 25, 0)
```

Elemental hitzone does not use tenderizing in this MVP.

---

## Weakness Exploit

Weakness Exploit can trigger only if:

```
H_phys >= 45
```

Expected Weakness Exploit affinity:

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

## Physical Damage

Combo-level expected physical damage:

```
D_phys = MV_total / 100 × ATK_eff × S_phy × H_phys / 100 × ECM × M_sweet
```

Where:

| Variable | Meaning |
| --- | --- |
| MV_total | Total physical motion value |
| ATK_eff | Effective true raw |
| S_phy | Physical sharpness multiplier |
| H_phys | Current raw hitzone |
| ECM | Expected Critical Multiplier |
| M_sweet | Sweet spot modifier |

---

## Effective Element

Effective element is:

```
ELE_eff = (weapon_ele + ELE_plus) × ELE_multi
```

If the weapon has no element:

```
If weapon_ele = 0:
    ELE_eff = 0
```

This prevents Element Attack from creating element on a non-elemental weapon.

---

## Elemental Damage

Combo-level expected elemental damage:

```
D_ele = MV_ele_total × ELE_eff / 10 × S_ele × H_dragon / 100
```

Where:

| Variable | Meaning |
| --- | --- |
| MV_ele_total | Total elemental motion value |
| ELE_eff | Effective element value |
| S_ele | Elemental sharpness multiplier |
| H_dragon | Dragon hitzone |

---

## Total Damage

Total expected combo damage:

```
D_total = D_phys + D_ele
```

The MVP does not apply per-hit flooring because the model uses combo-level aggregation.

---

## Effective Frames

Effective frames:

```
frames_eff =
frames_base
- charge_count × focus_reduce_per_charge
- sheathe_count × sheathe_reduce_frames
```

---

## Focus Frame Reduction

Focus is stored as a percentage.

| Focus Level | focus_reduce_pct |
| --- | --- |
| ---: | ---: |
| 0 | 0% |
| 1 | 5% |
| 2 | 10% |
| 3 | 15% |

For Great Sword:

```
charge_frame_per_charge = 71
```

Per-charge Focus reduction:

```
focus_reduce_per_charge = ROUNDDOWN(charge_frame_per_charge × focus_reduce_pct, 0)
```

Example:

```
Focus 2 = 10%
focus_reduce_per_charge = ROUNDDOWN(71 × 10%, 0)
focus_reduce_per_charge = 7
```

Focus affects `frames_eff` and DPS.

Focus does not affect `D_total`.

---

## Quick Sheath Frame Reduction

Quick Sheath uses direct frame reduction.

| Quick Sheath Level | sheathe_reduce_frames |
| --- | --- |
| ---: | ---: |
| 0 | 0 |
| 1 | 1 |
| 2 | 3 |
| 3 | 6 |

Quick Sheath reduction:

```
sheathe_frame_reduction = sheathe_count × sheathe_reduce_frames
```

Quick Sheath affects `frames_eff` and DPS.

Quick Sheath does not affect `D_total`.

---

## Final DPS

Final DPS:

```
DPS = D_total × 30 / frames_eff
```

The factor 30 converts frames into seconds.

---

## Marginal Gain

For skill recommendation:

```
gain = DPS_new / DPS_current - 1
```

The recommendation table sorts candidates by `gain`.

---

## MVP Simplifications

Current simplifications:

- continuous expected damage
- combo-level aggregation
- no per-hit flooring
- no per-hit critical roll simulation
- no sharpness consumption
- no full combat state enumeration
- no armor or decoration constraints

The model focuses on skill marginal value comparison rather than exact in-game damage replication.