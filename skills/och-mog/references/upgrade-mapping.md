# Upgrade Mapping

Use this file when the task requires mapping player-facing upgrade names to technical IDs or interpreting buffs in payloads.

## Scope And Confidence

These mappings come from observed notes and payloads. Most are strong enough to use operationally, but if a live run exposes conflicting labels, prefer the live game data.

## Player-Facing Name -> Technical ID

- `vigor I` -> `max_energy_small`
- `vigor II` -> `max_energy_large`
- `sharp blade` -> `attack_up`
- `golden touch` -> `treasure_bonus`
- `bone breaker` -> `bone_breaker`
- `lucky find` -> `lucky_find`
- `magnetic pull` -> `magnet`
- `quick step` -> `quick_step`
- `light feet` -> `swift_steps`
- `sprint` -> `sprint`
- `scavenger` -> `scavenger`
- `soft traps` -> `soft_traps`
- `mimic sense` -> `mimic_sense`
- `poison blade` -> `poison_blade`
- `treasure hunter` -> `treasure_hunter`
- `tough hide` -> `tough_hide`
- `trap sight` -> `trap_sight`
- `energy wave` -> `heal_large`
- `energy surge` -> `heal_small`
- `barrier` -> `shield`
- `crit strike` -> `critical_strike`
- `eagle eye` -> `vision_boost`
- `momentum` -> `momentum`
- `cleave` -> `cleave`
- `thorns` -> `thorns`
- `armor plating` -> `armor_plating`

## Observed Effects Summary

- `max_energy_small`: `+15` max energy, passive
- `max_energy_large`: larger max-energy upgrade, passive
- `attack_up`: `+1` attack
- `treasure_bonus`: treasure increase effect
- `lucky_find`: improved loot drops
- `scavenger`: pots and crates drop loot more reliably
- `quick_step`: free movement over a bounded duration
- `swift_steps`: stronger temporary free-movement effect
- `sprint`: finite pool of free moves
- `bone_breaker`: extra damage vs skeletons
- `mimic_sense`: reveal hidden mimics
- `soft_traps`: reduce trap damage
- `trap_sight`: reveal traps in explored areas
- `treasure_hunter`: extra item from chests
- `heal_small` / `heal_large`: instant energy restoration

## Buff Field Hints

Observed buff fields that help interpret upgrade state:

- `attackBonus`
- `boneBreaker`
- `critChance`
- `magnetRange`
- `sprintMoves`
- `swiftStepsTurns`
- `poisonBladeTurns`
- `luckyFindBonus`
- `treasureBonus`
- `visionBoost`
- `armorPlating`
- `shieldHits`
- `hasQuickStep`
- `hasScavenger`
- `hasToughHide`
- `hasTrapSight`
- `hasTreasureHunter`
- `hasRetaliation`
- `hasPortalAdept`
- `hasMimicSense`
- `hasSecondWind`
- `momentumMoves`
- `passiveBuffDurations`

## Enemy Name Hints

Confirmed or strongly supported technical names:

- `bat`: standard bat, likely the "black bat" the player referred to
- `skullbat`: distinct, higher-tier bat variant
- `slime`: standard slime
- `redslime`: distinct slime variant, not the same as Big Slime
- `kingslime`: Slime King / Big Slime / SK
- `king_slime_split`: split form of Slime King
- `maomi`: premium enemy hint observed in logs
- `ghost`: non-standard attack logic, avoid assuming normal melee behavior

## Jackpot-Related Notes

- `Sir Jackalot` is confirmed by system event text, but a direct run payload mapping has not yet been captured.
- `jackpotWon` exists in run payloads.
- `skDefeated` exists in run payloads and is likely tied to Slime King / SK progression.

