# Upgrade Mapping

Use this file when the task requires mapping player-facing upgrade names to technical IDs or interpreting buffs in payloads.

## Scope And Confidence

These mappings come from public payload behavior and product-facing names. If a live run exposes conflicting labels, prefer the live `pendingUpgradeOptions` data.

## Player-Facing Name -> Technical ID

Core and common upgrades:

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

Higher-impact or rarer upgrades:

- `second wind` -> `second_wind`
- `portal adept` -> `portal_adept`
- `retaliation` -> `retaliation`
- `spirit ward` -> `spirit_ward`
- `fatal edge` -> `fatal_edge`
- `executioner` -> `executioner`
- `invisibility` -> `invisibility`
- `phoenix` -> `phoenix`

## Observed Effects Summary

- `max_energy_small`: max energy increase
- `max_energy_large`: larger max-energy increase
- `attack_up`: attack increase
- `treasure_bonus`: treasure increase effect
- `lucky_find`: improved loot drops
- `scavenger`: pots and crates drop loot more reliably
- `quick_step`: temporary free movement behavior
- `swift_steps`: stronger temporary free-movement effect
- `sprint`: finite pool of free moves
- `bone_breaker`: extra damage vs skeletons
- `soft_traps`: reduce trap damage
- `trap_sight`: reveal traps in explored areas
- `treasure_hunter`: extra item from chests
- `heal_small` / `heal_large`: instant energy restoration
- `portal_adept`: improves teleport cost profile
- `spirit_ward`: changes ghost interaction
- `fatal_edge`: powerful low-health execute-style effect
- `second_wind`: survival recovery effect

Many passive upgrades have bounded floor durations. Check `passiveBuffDurations` and live buff fields before assuming an effect is permanent.

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

## Upgrade Selection Guidance

Default high-priority picks:

- economy and loot: `treasure_hunter`, `lucky_find`, `scavenger`, `magnet`
- movement and routing: `quick_step`, `swift_steps`, `sprint`, `portal_adept`
- survivability: `second_wind`, `tough_hide`, `shield`, `soft_traps`
- combat: `attack_up`, `fatal_edge`, `critical_strike`, `cleave`, `retaliation`

Legendary or rare behavior can be gated per run. Do not force a hardcoded legendary pick if it is not present in live options.

## Enemy Name Hints

Confirmed or strongly supported technical names:

- `slime`: standard slime
- `redslime`: distinct slime variant
- `bat`: standard bat
- `skullbat`: distinct, higher-tier bat variant
- `skeleton`: standard skeleton
- `skeleton2`: higher-tier skeleton
- `ghost`: invincible or special-interaction ghost
- `ghost2`, `ghost3`: damaging ghost variants
- `shroom`: stationary ranged enemy
- `kingslime`: Slime King / Big Slime / SK
- `king_slime_split`: split form of Slime King
- `mimic`: hidden chest enemy
- `skeletonking`: Sir Jackalot
- `pengu`: event enemy
- `capfull`: event enemy
- `maomi`: premium enemy hint observed in logs

## Jackpot-Related Notes

- `Sir Jackalot` is the jackpot enemy.
- Public run payloads can expose `jackpotWon`.
- Public run payloads can expose `skDefeated`, likely tied to Slime King / SK progression.
- Use current events and enemy payloads to decide whether to chase or fight; do not rely only on name matching.
