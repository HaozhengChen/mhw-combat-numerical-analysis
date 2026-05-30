# Design Reflection

This document reflects on what the Monster Hunter combat numerical analysis project suggests for future numerical design work.

The goal of this reflection is to connect the spreadsheet model, damage formulas, skill marginal gain, and boss design analysis back to practical game design thinking.

---

## 1. From Formula to Player Behavior

A damage formula is not only a calculation tool.

It also shapes player behavior by defining what the game rewards.

For example:

- If weak point hitzones give much higher damage, players learn positioning.
- If affinity and Critical Boost interact multiplicatively, players learn build synergy.
- If Focus reduces charge frames, Great Sword players learn the value of timing and openings.
- If a monster's head has the best hitzone but is dangerous to approach, players must weigh risk and reward.

This means numerical design is not separate from gameplay design.

A number in a formula eventually becomes a player decision.

---

## 2. Skill Value Is State-Dependent

One of the main lessons from the Skill Priority Recommender is:

```
A skill does not have one fixed value.
```

Skill value depends on the current combat state.

Examples:

- Critical Boost is weak when effective affinity is low.
- Critical Boost becomes strong when effective affinity is high.
- Weakness Exploit has no value if the hitzone condition is not met.
- Focus has no value if the combo has no charge segment.
- Quick Sheath has no value if the loop has no sheathing action.
- Element Attack has no value on a non-elemental weapon.

This suggests that designers should be careful with static tier lists or isolated skill evaluations.

A better design workflow is to evaluate skills across multiple realistic scenarios.

---

## 3. Marginal Value as a Balancing Tool

The recommender is built around marginal gain:

```
gain = DPS_new / DPS_current - 1
```

This is useful because designers often need to answer practical balancing questions:

- Is this skill level too strong?
- Is this level a meaningful breakpoint?
- Does this upgrade feel worth the investment?
- Does this skill become useless after a certain threshold?
- Does a skill create an unhealthy must-pick point?

For example, Attack Boost 3→4 can rank highly because it adds both raw attack and affinity.

This reveals a breakpoint.

In a production environment, marginal value analysis can help identify:

- over-efficient levels
- weak levels
- breakpoint spikes
- diminishing returns
- overcapping issues

This is more useful than only looking at total build DPS.

---

## 4. Conditional Coverage and Real Combat Value

Many Monster Hunter skills are conditional.

Examples:

- Agitator depends on monster rage uptime.
- Weakness Exploit depends on hitzone and weak point uptime.
- Peak Performance depends on full-health uptime.
- Resentment depends on red-health uptime.
- Maximum Might depends on stamina state.
- Latent Power depends on trigger conditions.

The MVP uses coverage-rate approximation:

```
expected_value = full_value × coverage
```

This is a simplification, but it is useful because it forces the designer to ask:

```
How often is this skill actually active?
```

This is important for real balancing work.

A skill with a high theoretical value but low uptime may be healthy.

A skill with a moderate value but near-permanent uptime may become dominant.

Therefore, conditional skills should be balanced around expected value, not only maximum value.

---

## 5. Frame Economy in Action Games

Action games are not only balanced by attack and defense values.

They are also balanced by time.

The DPS formula makes this explicit:

```
DPS = D_total × 30 / frames_eff
```

This means frame cost is part of the numerical system.

Focus and Quick Sheath demonstrate this clearly:

- Focus does not increase damage per hit.
- Quick Sheath does not increase damage per hit.
- Both can still increase DPS by reducing time cost.

This is a key lesson for action game numerical design:

```
Animation duration, recovery, charge time, and sheathing time are numerical balance levers.
```

A combat designer and numerical designer should therefore evaluate damage and animation timing together.

---

## 6. Boss Design and Numerical Design Are Connected

The boss design article shows that numerical systems and encounter systems are deeply connected.

A skill may look strong in a spreadsheet but underperform in real hunts if:

- the monster rarely exposes weak hitzones
- the player cannot safely complete the combo
- the monster moves too much
- the skill condition has low uptime
- the best hitzone is too dangerous to attack
- the weapon's animation commitment does not fit the opening size

This means a numerical model should include assumptions such as:

- hitzone
- combo
- tenderized state
- coverage
- frame duration
- weak point uptime
- rage uptime

The spreadsheet does not replace encounter analysis.

It supports it.

---

## 7. Tooling Mindset for Numerical Design

A useful numerical model should be:

- explainable
- adjustable
- reusable
- easy to validate
- easy to present

This is why the workbook separates:

- raw data
- formulas
- recommendation logic
- dashboard output
- validation checks

In a team environment, a numerical tool is useful only if other designers can understand and adjust it.

The Dashboard is therefore not just decoration.

It is part of the tool's communication layer.

A good design tool should help answer:

```
What changed?
Why did it change?
Is the result reasonable?
What assumption caused the result?
```

---

## 8. What This Project Suggests for Production Work

In a production environment, this type of model could support:

- skill tuning
- weapon balance review
- build diversity analysis
- boss hitzone tuning
- combat pacing analysis
- player build recommendation
- patch impact prediction

For example:

### Skill tuning

If one skill appears as the best next-point option across too many scenarios, it may be over-efficient.

### Hitzone tuning

If weak point values make Weakness Exploit mandatory, monster hitzones may be too generous or too centralized.

### Frame tuning

If one combo dominates because of its frame efficiency, animation timing may need review.

### Build diversity

If different scenarios recommend different skills, the system supports build diversity.

---

## 9. What I Would Improve in a Real Project

If this were developed further for production-style use, I would improve it in several directions.

### Add per-hit calculation

The MVP uses combo-level aggregation.

A more accurate model would calculate each hit separately and apply per-hit flooring.

### Add state enumeration

Instead of using only coverage-rate approximation, the model could enumerate states:

```
rage on / off
tenderized / not tenderized
weak point uptime
full health / not full health
```

### Add build cost

The current model treats each skill level as one comparable point.

A future model should include slot cost, armor constraints, and decoration availability.

### Add validation automation

Important rules could become automated tests:

- Focus gain is 0 when `charge_count = 0`.
- Element Attack gain is 0 when `weapon_ele = 0`.
- Weakness Exploit gain is 0 when `H_phys < 45`.
- ECM is below 1 when affinity is negative.

### Add scenario comparison

The tool could compare recommendations across multiple monsters, hitzones, and combos.

This would better support balance decisions.

---

## 10. Main Takeaways

The main lessons from this project are:

1. Skill value should be evaluated in context.
2. Marginal value is useful for balancing skill levels.
3. Conditional skills should be evaluated by expected uptime.
4. Frame cost is part of the numerical system.
5. Boss design and numerical design should be analyzed together.
6. A good numerical tool should be explainable and reusable.
7. Numbers shape player behavior, not just damage output.

---

## 11. Final Reflection

This project started as a spreadsheet model for Great Sword skill value.

However, the process showed that a numerical model is most valuable when it connects back to design questions.

The important question is not only:

```
What is the DPS?
```

The more useful design questions are:

```
Why is this skill valuable here?
What player behavior does this number reward?
Does this create meaningful choice?
Does this support build diversity?
Is the value understandable and fair?
```

For future numerical design work, I would treat formulas, tools, and encounter analysis as connected parts of the same design process.

A strong numerical designer should not only calculate balance.

A strong numerical designer should also understand what player behavior the balance creates.