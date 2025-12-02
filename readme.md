
- Game Source: https://github.com/yiyuezhuo/Late-Qing-Naval-Combat-Demo
- Steam: https://store.steampowered.com/app/3996220/First_SinoJapanese_War/
- Discussion: https://discord.gg/2yqbyGwsdQ

# Naval Tactical Mode

## Battery

Battery related resolution is based on SEEKRIEG 5 rule (though not always identical).  Reading this manual should be sufficient to understand the related mechanism; however, for further insight, it can be instructive to consult the original SK5 rulebook, though it is behind a paywall.

Ship Log's state items are based on the data in the Ship Class. The mapping relationship looks like this:

<img src="images/Battery.svg">
(FCS is mapped to "Fire Control Positions").

The target is set at the mount level. If Auto Firing is enabled (the default setting), the WTA Solver will guide the battery to fire at a target. The battery will then attempt to use the FCS and any valid mount (with a satisfied firing angle) to engage the target.

When an FCS tracks a unit, it increases its **TK Seconds (Tracking Seconds)** for that target. The tracking state will **upgrade** from "Begin Tracking" to "Tracking," resulting in a higher accuracy bonus. The FCS modifiers are as follows:

- Local Control (Target is not tracked by any FCS): Large negative modifier
- Begin Tracking: Small negative modifier
- Tracking: Neutral
- Hitting (Target was hit recently): Small Positive modifier

Similarly, a mount accumulates **Processing Seconds**. When the processing seconds reach the "seconds per round" value (determined by the Rate of Fire, which is based on the Penetration Table and range), the process seconds reset to 0, and a firing round is resolved.

<img src="images/ROF_Base_Fire_Control.svg">

The above image also show how the base fire control value is derived:

- The **Range Band** (determined by distance in the penetration table) and the **Target Aspect** determine the column.
- The **Target Speed** determines the row.

This fire control value is then adjusted by multiple factors:

- Visibility
- Evasive Action
- Fire Control System
- Over-centration
- Target Size
- Fire Control Radar (Although the game era doesn't cover radar (SK5 covered it), this value can be used to denote range-finder or other useful damageable device.)

The detailed rule can be checked in source code.

Note: Flying shell is not modeled in the game. The flying time is part of "processing time" in the game. 

### Hit Resolution

When a hit is scored, following procedure is performed:

- Roll for Hit Location & Vertical/Horizontal hit according Range Band (Short/Medium/Long/Extreme) and target aspect (Narrow/Broad).
- Compare Vertical/Horizontal penetration value (for given range) to effective armor value of the hit location. Three outcomes are possible:
	- Pass-Through: Less Effective
	- Penetrates And Detonates: Most Effective
	- Not Penetrate: Least Effective
- According to penetrate type, and use Ammo Type (AP/SAP/COM/HE), roll for Damage Effect and Damage Point inflicted.
- In increase in Damage Point may generate additional Damage Effects, or instant sink the ship if it become too high for a turn (but 100% DP doesn't automatically sink the ship). Damage Effect themselves may also generate further Damage Effects.

Note: Damage Effect DE XXX is the name of SK5 name, the effect can be checked in source code or SK5 material (not always identical though).
## Torpedo

Torpedoes do not follow the SK5 rule, because the compromises made for tabletop play would result in a poorer system while not being any easier to implement compared to a system based on actual collision detection handled by the game engine’s physics.

### Ship Collider

To improve recognizability, the size of the icon/model can be increased in the settings (just like how miniature wargames often use oversized models). However, the real collider size is maintained. Internally in the game engine, it looks like this:

<img src="images/Collider vs Icon.png">
(The box represents the real collider. The LOS detection tool is based on the actual LOS, so it naturally handles curvature as well.)

### Interception Point

WIP

## Weapon Target Assignment

WIP

# Strategic Mode

## Strategic Group & Land Unit

WIP

## Movement

WIP

## Supply

WIP

## Naval Transfer

Navy Transfer must be carried out through a naval mission. The Non-Fleet Strategic Groups to be transported via Navy Transfer and the Fleet Group providing the transportation need to be assigned to the mission. During the assembly phase, they will arrive at the assembly point (the first hex of the waypoint). The groups to be transported are generally divided into several parts and loaded onto individual; transport ships, with large formation gradually being emptied until they become small enough to be entirely loaded onto a single transport ship. Upon reaching the destination, the groups will, by default, automatically reorganize into their original formation. The navy transfer may require multiple rounds trips. Once the mission is completed, the transport fleet will remain stationed at the assembly point (it's a good start point for a persistent supply mission).

<img src="images/naval transfer mission allocation.svg">
<img src="images/naval transfer mission example.svg">

## Repair

The repair system is inspired by WITP (War in the Pacific). Specifically, damage from SK5 is mapped to a simplified WITP repair model for resolution. (Sample mapping: 1 SK5 Flooding Hit ≈ 20 WITP Flooding Damage Points, or 20% flooding.)

### Tactical Naval Combat damage effect Trim

When a ship’s state is synced to the strategic game, certain adjustments are applied to damage effects. Non-permanent and some specific damage effects are removed.

### Repairable damage

Following state could be counted as a damage:

<img src="images/repairable damage.svg">

For example, each max speed offset is counted as separate damage and can be repaired individually to restore the ship's original speed.

Different damage types have different repair point costs and priorities, and are classified as either normal damage or major damage. Major damage can only be repaired by a repair provider that meets the displacement requirement.

### Repair Device

Damage can be repaired by ports and repair shipyards.

<img src="images/Port_Repair_Shipyard.png">

| Type | Repair Point Generated | Repair Upper Limit (tons) |
| ---- | ---------------------- | ------------------------- |
| Port (per level) | 100 | 0 |
| Repair Shipyard (per level) | 100 | 1000 |

Repair ships have been added to the game, but their repair functionality will be implemented in a future update.

### Mapped Damage Point

In the SK5 system, the Taken Damage Point for an unsunk ship is sometimes a _good thing_, since a high damage point indicates that the ship’s “instant-kill potential” has been expended—for example, its magazine has been destroyed relatively safely. Therefore, damage points are not listed as a repairable item in the repair system. Instead, they are _forcibly_ restored according to the ratio of the repaired damage’s repair cost to the ship’s total existing repair cost. The forcibly removed damage points are called **“mapped damage points”** in the log.

## Command Structure

Each strategic group has a command point value cost:

- Each land unit under the command will cost the original value.
- Each strategic group under the command will cost 1/3 of its own command point.

Strategic group have a fixed command capacity of 1500. If command point cost is lower than the capacity, the group will be subject to a constant penalty based on commander's Land Operational ability:

| Level | Command Penalty | Tactical Modifier |
| ----- | --------------- | ---------------- |
| Barely Competent | 40% | -10% |
| Average (Unknown) | 30% | 0% |
| Above Average | 20% | 10% |
| Outstanding | 10% | 20% |
| Gifted | 0% | 30% |

In the land battle resolution, a unit's "Chance Cost" (basically cost to "activate" this unit) is modified by every layer of engaged commanding layer:

<img src="images/command penalty.svg">

In the above example, if the 9th Combined Bridge is the leading group (If only 1 independent group is engaged, that group is the leading group. If more than one group is engaged in the land battle, weaker groups would attached to the strongest group temporarily. So there's still only 1 group on the top and other attached group will be subject to extra command layer penalty.). This layer will yield a command penalty of 20% since command point is lower than capacity and commander's Land Operational ability is "above average". On the other hand, 11th Inf Regt layer will generate 32% penalty. So 2nd Bn, 11th Inf Regt and 3rd Bn, 11th Inf Regt would be subject to 52% penalty.

If 5th Div is engaged instead of 9th Combined Bridge, the penalty would be stacked with an extra layer introduced by extra command layer.

This design achieves the following effects:

- Bigger land units are hard to activate, so it's less-effective than smaller unit, that's how I implement the "soft combat width".
- Every engaged operational commander in the commander chain make proper effect, instead of strange positive buf stack by enforced layer (EX: HOI) or just no effect on higher layer (EX: ATG).

The penalty system is designed to mathematically ensure that "flatter" command structures generally incur higher penalties. Thus, Japan's modernized hierarchical command structure will typically perform better than China's pre-Napoleonic structure, which lacked permanent formations above the battalion level.

The tactical modifier, however, only affects directly commanded land units and does not apply through the hierarchy.

## Land Battle Resolution

### Sub Combat

Each turn, every unit generates a "chance" value representing its tactical potential. Two chance values would be consumed in a sequence of generated sub-combat for this turn.

For each sub-combat, the two sides roll with their total chance as weight to determine the initiative side. The initiative side becomes the attacker of the sub-combat, while the opposing side becomes the defender. An attacker unit is rolled according to the power weighting and a defender unit is selected based on strength weighting (so low-effective unit -- those with high suppression, fatigue and low morale) would be more likely to be chosen defender than attacker in sub-combat.

The two engaged units then determine their commitment percentage, which is based on a "reference ratio" (e.g., a 300-man unit attacking a 100-man unit with a 1:1 reference ratio would result in the attacker committing 100 men and the defender committing 100 men). The determination of the reference ratio is heavily influenced by the current situation value, which represents the loss or gain of critical advantageous positions. The situation only shifts when the attacker in the sub-combat achieves a significant advantage by the end of the sub-combat (attacker is not suppressed, defender is suppressed, attacker has higher morale and lower fatigue). This represents a tactical breakthrough that may force the defender to retreat from their current position. Conversely, if the defender achieves a "victory," the situation does not shift, aside from potentially a favorable attrition ratio.

Generating a sub-combat consumes chance from the initiative side based on the unit's state and its commitment percentage.

In an execution of land battle, which may last several turns, the side with the advantage gradually pushes the situation in its favor, gaining increasingly better exchange ratios. The disadvantaged side may launch counterattacks to retake the situation, but may ultimately be unable to sustain the deteriorating exchange ratio and choose to retreat from the hex. They may then retreat to a new hex and start a new battle, where the situation resets — allowing this cycle to potentially repeat.

Thus, sub-combats give the initiative side an upper hand, and having more initiative is almost always beneficial. This can be achieved by:

- More units — generates higher base chance values.
- A more effective command structure and higher commander ability — reduces the chance cost for each sub-combat generated.

For the resolution of a land battle within a single strategic turn (hour), chance is only consumed up to a given percentage (e.g., 90%) before the resolution ends. This threshold can be adjusted to control battle intensity.

<img src="images/Land Battle.svg">

### Abstract Firing

A sub combat is consisted of 3 firing exchange. Each firing inflict strength, suppression, morale and fatigue hit. Two side also increase some fatigue.

The 3 non-"physical" effectiveness attribute, suppression, morale and fatigue is defined as:

- Suppression: An effect is somewhat longer than true "fire suppression", but even it's 100% suppressed, most of the value would be restored in the new turn (not 100% is due to some suppression is assumed to happen in the end of last turn). 
- Morale: morale is like suppression, but slower to restore and harder to inflict.
- Fatigue: fatigue is like morale, but slower to restore and harder to inflict. And some fatigue would be imposed without loss.

These value with strength modify the "lethality" (a concept taken from JTS Squad Battle), which effect inflict hits and if a "breakthrough" (situation push) is achieved.