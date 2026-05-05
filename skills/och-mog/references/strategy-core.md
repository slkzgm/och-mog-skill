# Strategy Core

Use this file for practical play policy, floor-phase goals, chest and pot priorities, sustain routing, items, portals, and premium target handling.

## Scope And Confidence

This file combines:

- observed public API and payload behavior
- public client interaction rules
- guidance from a strong player

Treat these as high-value heuristics, not perfect proofs of optimal play. The server state is always authoritative.

## Primary Mode Framing

The strongest general framing is:

- early floors are setup and efficient farming
- later floors are where combat EV and premium targets matter more
- the run should be managed so floor `7+` is still playable, not merely reachable

## Floor Phases

### Floors 1-4

Default objective:

- kill all enemies that can be fought safely
- open all pots
- route through nearby chests when efficient

Interpretation of "clear":

- enemy full-clear is the core requirement
- pots are part of the baseline value loop
- crates and nearby chests are secondary, but often worthwhile if routing is efficient

### Floors 5-6

Default objective:

- clear only if the run remains healthy
- avoid spending too much energy for mediocre EV
- start preserving tactical control for floor `7+`

### Floors 7+

Default objective:

- maximize profitable combat
- preserve tactical control
- prioritize premium enemies and dangerous followers over generic side loot

This is the phase where:

- attack-oriented upgrades matter much more
- rerolls become more defensible
- premium targets like `kingslime` and jackpot-related enemies become strategically relevant

## Energy Discipline

The player guidance is explicit:

- reaching floor `7` with as much energy as possible is highly valuable

Practical interpretation:

- floors `1-4` should usually be cleared hard
- floors `5-6` should be cleared only if they do not compromise the floor `7+` phase
- low-energy salvage is better than dying while trying to overclear

## Enemy Priority

At higher floors, prioritize enemies that can follow and pressure the player.

Explicit player guidance:

- `skeleton`
- `bat`

Implication:

- do not over-prioritize passive side objectives while a follower can collapse onto your position
- combat pressure can be more important than visible chest routing

## Chest, Pot, Crate, Rock

### Chests

Chests are movement-blocking but auto-open when the player moves adjacent.

Use:

- path adjacent to visible chests when the route is efficient
- remember dormant mimics can be displayed as chests unless mimic-sense state is active
- once opened, chest loot scatters around the chest and may require follow-up movement

### Pots

Pots are strategically important:

- they can drop sustain, resources, and hidden consumable items
- they often fit naturally into room-clear routing
- use explicit `break`

### Crates

Crates are still valuable, but the strongest player signal emphasized pots more than crates.

Use explicit `break`.

### Rocks

Rocks block movement. Do not treat them as ordinary breakable objects unless a future public state explicitly introduces a valid rock action.

## Fountains

Fountains are one-time sustain nodes.

Observed rule:

- moving onto a fountain the first time can restore up to `10` energy

Practical use:

- route through fountains when energy is low and path cost is reasonable
- treat fountains as sustain tools, especially before or during the high-floor phase
- do not ignore a nearby fountain if it materially stabilizes the run

## Items

Inventory is limited, so item decisions matter.

Known useful patterns:

- use `gas_pedal` or movement support to escape lethal pressure or improve premium-target routing
- use targeted shot items on high-value or dangerous enemies when target rules allow
- use `pocket_portal` or `escape_rope` for floor control when the run is unstable
- preserve survival items for lethal or near-lethal moments
- `discard_item` is free and can be correct when holding low-value inventory blocks future item access

Do not let item optimization distract from immediate lethal threats.

## Portals

Portals are movement-based first. Teleport only after the state indicates the player is at a portal prompt.

Teleport cost follows reroll-style progression. `portal_adept` improves the cost profile.

Use teleport when:

- current floor position is poor
- a premium target or stairs route is much better after teleport
- the cost is justified by survival or EV

Avoid teleport when:

- the current local route is already strong
- the run cannot afford the resource cost
- upgrade selection is pending

## Combat Micro: Do Not Always Step Into Adjacency

Important micro-heuristic:

- some enemies hit when the player enters an adjacent tile
- when such an enemy deals more than `1` damage, it can be better to wait and let the enemy come closer instead of stepping into range first

This is not universal.

Only use this patience rule when:

- the enemy pattern will actually move toward the player
- waiting does not create a worse multi-enemy position
- the delay improves the trade materially

Do not:

- spam `pass` while waiting for a non-converging enemy
- assume every enemy will approach

## Multi-Enemy Risk

The player's strongest "common mistakes" warning is:

- do not fight multiple enemies at the same time if avoidable
- do not path through the middle of a room unnecessarily

Practical implication:

- prefer routing that keeps edge control
- avoid exposing the player to two-sided pressure
- value clean sequencing of fights over greedy central movement

## Premium Targets And Rarity

### Slime King / SK

Technical mapping:

- `kingslime`
- split form: `king_slime_split`

Strategic implication:

- high-value enemy from floor `8+`
- can be worth routing toward in higher floors
- be ready for split-form cleanup and positioning risk

### Sir Jackalot

Concept:

- jackpot enemy
- currently appears in the floor `7-15` band when eligible
- current normal-run spawn pre-roll is `8%`
- defeating or hitting it can produce jackpot claimable value depending on server event outcome
- after being resolved once, it should not appear again later in the same run
- the run continues after the fight

Technical mapping:

- `skeletonking`
- usually appears with fleeing behavior

Strategic implication:

- unlike follower enemies such as bats or skeletons, Sir Jackalot should be treated as a premium chase target
- do not apply the generic "wait for the enemy to come to you" micro to him by default if the payload continues to show fleeing behavior

### Mimics

Technical mapping:

- `mimic`
- dormant mimics can appear as closed chests in client-safe state
- mimic-sense state reveals dormant mimics when present in the returned state

Strategic implication:

- chest routing has hidden combat risk
- do not treat every visible chest as risk-free value

## Break-Even Interpretation

Current best interpretation of the player guidance:

- full-clearing enemies on floors `1-4` is an important part of break-even logic
- from floor `7+`, the best profitable combat you can manage matters more than rigid full-clear behavior

This should not be interpreted as:

- "always full-clear every floor no matter the state"

It should be interpreted as:

- early floors are clear-heavy by default
- higher floors are EV-heavy and condition-dependent
