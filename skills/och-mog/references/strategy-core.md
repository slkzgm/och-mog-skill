# Strategy Core

Use this file for practical play policy, floor-phase goals, chest and pot priorities, sustain routing, and premium target handling.

## Scope And Confidence

This file combines:

- observed API and payload behavior
- custom-client interaction rules
- guidance from a strong player

Treat these as high-value heuristics, not perfect proofs of optimal play.

## Primary Mode Framing

The strongest general framing is:

- early floors are setup and efficient farming
- later floors are where combat EV and premium targets matter more
- the run should be managed so floor `7+` is still playable, not merely reachable

## Floor Phases

### Floors 1-4

Default objective:

- kill all enemies
- open all pots

Interpretation of "clear":

- enemy full-clear is the core requirement
- pots are part of the baseline value loop
- crates and nearby chests are secondary to that baseline, but still often worthwhile if routing is efficient

### Floors 5-6

Default objective:

- clear only if the run remains healthy
- do not spend too much energy for mediocre EV

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

## Chest Priority

Early and neutral state:

- if a chest is visible, its room can be favored when the route is still efficient

Higher-floor / jackpot phase:

- premium enemies and dangerous followers take priority over chest routing
- do not let chest greed drag the run into low-energy or multi-enemy situations

Additional caution:

- mimics exist
- chest value becomes less important when jackpot or premium-combat EV is on the table

## Pots, Crates, And Sustain

### Pots

Pots are strategically important enough to be part of early-floor "clear" logic.

Reason:

- they can generate sustain and loot
- they often fit naturally into room-clear routing

### Crates

Crates are still valuable, but the strongest player signal emphasized pots more than crates.

### Fountains

Fountains are one-time sustain nodes.

Observed rule:

- moving onto a fountain the first time can restore energy

Practical use:

- route through fountains when energy is low and path cost is reasonable
- treat fountains as sustain tools, especially before or during the high-floor phase
- do not ignore a nearby fountain if it materially stabilizes the run

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

The player’s strongest "common mistakes" warning is:

- do not fight multiple enemies at the same time if avoidable
- do not path through the middle of a room unnecessarily

Practical implication:

- prefer routing that keeps edge control
- avoid exposing the player to two-sided pressure
- value clean sequencing of fights over greedy central movement

## Rerolls And Build Timing

Preferred pattern:

- avoid rerolling heavily before floor `7` if possible
- save reroll budget for the floors where combat EV matters more

But:

- if an early reroll materially saves a run, it can still be correct
- treasure spent on stability can be positive EV over many runs

## Premium Targets And Rarity

### Slime King / SK

Confirmed technical mapping:

- `kingslime`
- split form: `king_slime_split`

### Sir Jackalot

Confirmed conceptually:

- this is the jackpot enemy
- defeating it can trigger jackpot payout

Technical caveat:

- a direct run payload for Sir Jackalot has not yet been captured in our data
- treat it as real but rare

### Maomi

`maomi` appears in logs and should be treated as a premium-value enemy hint until more precise EV data is collected.

## Break-Even Interpretation

Current best interpretation of the player guidance:

- full-clearing enemies on floors `1-4` is an important part of break-even logic
- from floor `7+`, the best profitable combat you can manage matters more than rigid full-clear behavior

This should not be interpreted as:

- "always full-clear every floor no matter the state"

It should be interpreted as:

- early floors are clear-heavy by default
- higher floors are EV-heavy and condition-dependent

