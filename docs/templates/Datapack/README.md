

The downloadable template replaces CobbleCoop's five included `basic.json` pools. Place the ZIP directly in the world's `datapacks` folder, edit it, and run `/reload`.

## Included tiers

CobbleCoop includes one basic pool for each tier:
```text
data/cobblecoop/loot_table/cobblecoop/rewards/
|-- common/basic.json
|-- uncommon/basic.json
|-- rare/basic.json
|-- epic/basic.json
`-- legendary/basic.json
```

The word `basic` is only the file name. You may replace it or add files with clearer names such as `berries.json`, `training_items.json`, or `building_materials.json`.

## Automatic discovery

Every JSON below one of these folders is detected automatically:

```text
data/<namespace>/loot_table/cobblecoop/rewards/common/
data/<namespace>/loot_table/cobblecoop/rewards/uncommon/
data/<namespace>/loot_table/cobblecoop/rewards/rare/
data/<namespace>/loot_table/cobblecoop/rewards/epic/
data/<namespace>/loot_table/cobblecoop/rewards/legendary/
```

Subfolders are also supported. This is valid:

```text
data/myserver/loot_table/cobblecoop/rewards/rare/training/vitamins.json
```

Run `/reload` after adding or editing datapack files. CobbleCoop builds an in-memory index during the reload. Opening a bag does not scan folders or write pool information to the world.

## Selection rules

If a tier contains several JSON files, each file has the same chance of being selected.

Example:

```text
common/basic.json
common/berries.json
common/balls.json
```

Each file has a one-in-three chance. Item `weight` values inside `basic.json` only compare entries from that Minecraft loot table; they do not change the chance of selecting another file.

For the easiest setup, keep one `basic.json` file per tier and place all item entries and weights inside it. Add several files only when you intentionally want equally likely themed pools.

## Adding a custom pool

Create a datapack with this structure:

```text
MyServerRewards/
|-- pack.mcmeta
`-- data/
    `-- myserver/
        `-- loot_table/
            `-- cobblecoop/
                `-- rewards/
                    `-- common/
                        `-- basic.json
```

No TOML registration is required. The example above creates this loot table ID:

```text
myserver:cobblecoop/rewards/common/basic
```

It is automatically added to the Common Reward Bag after `/reload`.

## Complete basic pool example

```json
{
  "type": "minecraft:chest",
  "pools": [
    {
      "rolls": 2,
      "entries": [
        {
          "type": "minecraft:item",
          "name": "cobblemon:poke_ball",
          "weight": 20,
          "functions": [
            {
              "function": "minecraft:set_count",
              "count": {
                "min": 3,
                "max": 7
              }
            }
          ]
        },
        {
          "type": "minecraft:item",
          "name": "minecraft:iron_ingot",
          "weight": 10,
          "functions": [
            {
              "function": "minecraft:set_count",
              "count": {
                "min": 2,
                "max": 5
              }
            }
          ]
        }
      ]
    }
  ]
}
```

`rolls` controls how many entries are rolled. `weight` controls the relative chance of entries inside that pool. `set_count` controls the amount of the selected item.

No `random_sequence` field is necessary. Bags use the server random generator when they are opened and do not store a seed per item.

## Replacing the included basic pool

Use the same namespace and path as CobbleCoop:

```text
data/cobblecoop/loot_table/cobblecoop/rewards/common/basic.json
```

Because the resource ID is identical, the higher-priority datapack replaces the included Common pool instead of adding a second candidate.

The same rule applies to the other four tiers.

## Adding another pool instead of replacing one

Use your own namespace or another file name:

```text
data/myserver/loot_table/cobblecoop/rewards/common/berries.json
```

The included `cobblecoop:.../common/basic` and custom `myserver:.../common/berries` tables will then be equally likely.

## Reward bag item IDs

```text
cobblecoop:common_reward_bag
cobblecoop:uncommon_reward_bag
cobblecoop:rare_reward_bag
cobblecoop:epic_reward_bag
cobblecoop:legendary_reward_bag
```

Give a bag for testing:

```text
/give @s cobblecoop:common_reward_bag
```

Test a loot table directly:

```text
/loot give @s loot myserver:cobblecoop/rewards/common/basic
```

## rewards.toml

The generated file is located at:

```text
config/cobblecoop/rewards/rewards.toml
```

It controls scoring, tier chances, bonus experience, and anti-farm protection. Reward items still remain entirely inside datapack loot tables.

The complete commented file is available at `docs/rewards/rewards.toml.example`.

## Handicap reward score

Every handicap has one configurable maximum point value. Boolean choices award zero or the complete value. Custom number arrays are interpreted in the order shown in the menu: the first active position receives a fraction and the last active position receives the configured maximum.

For example, with `teamReduction = 12` points:

```text
Configured values: [0, 3, 12, 25]
Selected 0:   0 points
Selected 3:   4 points
Selected 12:  8 points
Selected 25: 12 points
```

Adding more intermediate values does not increase the maximum score. Keep the neutral value first and order active values from least to most difficult. The default move list is therefore `[0, 3, 2, 1]`.

Exact Generation receives a fixed score because generation number is not a reliable difficulty scale. Up To Generation becomes more valuable as fewer generations remain available. Randomizers award zero by default because random results can make a trainer weaker.

Player Level and Trainer Level are finalized differently when the battle starts. Their menu score is replaced by points calculated from the real selected-team levels, so choosing Same Level without actually reducing anything does not generate free reward points.

## Random tier selection

The original effective average of the trainer team chooses one of four configurable weight bands. Reward score then grows higher-tier weights exponentially. All weights are normalized before the tier is rolled.

Default weights are ordered as Common, Uncommon, Rare, Epic, and Legendary:

```toml
[tierWeights]
level1To25 = [7500, 2000, 480, 19, 1]
level26To50 = [3500, 4500, 1700, 290, 10]
level51To75 = [1000, 3500, 4000, 1400, 100]
level76To100 = [300, 1700, 4000, 3200, 800]
```

A zero remains zero after handicap boosts. The defaults use very small non-zero Epic and Legendary weights in the lower bands. They are almost invisible in an easy battle but can grow into meaningful chances when a low-level team accepts many real handicaps. Existing files containing the exact old default arrays are migrated to this curve at runtime; custom arrays are preserved.

## Personal soft pity

Each player has separate Epic and Legendary pity progress for every trainer level band. An eligible tier roll that misses Epic increases the Epic counter; Epic or Legendary resets it. Every eligible non-Legendary result increases the Legendary counter, and only Legendary resets it.

Pity changes probability gradually and never guarantees a tier. The defaults begin Epic growth after 4 eligible misses and reach its maximum bonus at 20. Legendary begins after 10 and reaches its maximum at 40. Low-level trainers keep deliberately small caps, and progress earned in one level band cannot be carried into another.

```toml
[pity]
epicRange = [4, 20]
legendaryRange = [10, 40]
epicCap = [1.0, 7.0, 22.0, 40.0]
legendaryCap = [0.1, 0.75, 3.0, 12.0]
```

Cap arrays are ordered as Low, Mid, High, and Endgame percentages. If handicap-adjusted probability already exceeds a configured cap, pity does not lower it. A configured zero tier also remains locked.

Only a full, non-repeated reward roll advances pity. Debug battles, overleveled farming, repeat-protected encounters, failed bag checks, losses, and cancelled battles provide no progress. Receiving a high tier from a reduced repeat roll still resets the matching counter so pity cannot be retained after obtaining that reward.

Pity uses compact vanilla world `SavedData`. Only players with non-zero progress are stored, using two small four-band counter arrays per UUID.

## Bonus experience

`experience.percentPerPoint` adds a configurable percentage to real battle EXP for every reward point. The bonus is capped by `maximumBonusPercent` and reduced by the same anti-farm protection used for bags. It works on both temporary battle teams and direct original-team battles.

## Anti-farm protection

Level relevance compares the rounded average of every Pokémon selected by both allies against the corresponding trainer-team average. Both sides apply level handicaps first. A carry safeguard keeps each effective level within `levels.carryGap` of its strongest member, preventing a level 100 Pokémon from being hidden behind several level 1 fillers.

The original effective average is also recorded. Every real group of levels removed by Same Level or Player Level grants a bounded reward point, while every configured group of levels that the trainer remains above the adjusted team grants another bounded point. A high-level team that genuinely scales itself down is rewarded; a low-level team receives the same opportunity through natural disadvantage and other handicaps.

The level settings intentionally use short names:

| Setting | Meaning |
| --- | --- |
| `levels.carryGap` | Maximum distance between the strongest member and the protected average. |
| `levels.reductionStep` | Real levels removed for one Player Level reward point. |
| `levels.challengeStep` | Trainer advantage levels for one Trainer Level reward point. |
| `announcements.announceHighTiers` | Announces Epic and Legendary bags when enabled. |

Victories are also recorded per player and stable trainer ID. With the defaults:

```text
First victory:  100% bag chance
Second victory: 35% bag chance
Third victory:  12.25% bag chance
Further wins:   0% until reset
Reset:          24 real-world hours without another victory
```

The history is stored as small vanilla world SavedData. Each entry contains only player UUID, trainer encounter ID, victory count, and last victory time. Debug battles do not create entries.

## Setup menu forecast

The Handicap tab shows a server-synchronized table on the left with exact Common, Uncommon, Rare, Epic, and Legendary tier percentages. Additional Information remains on the right. The server sends the real point values, tier curve, and current player's pity when the menu opens, so dedicated-server settings are not replaced by the client's local configuration.

The white percentage includes every current modifier. When pity contributes, a cyan `(+x%)` appears beside it and explains the personal bonus on hover. Final Player Level and Trainer Level points still require both selected teams and are recalculated when the battle launches. Repeat and overlevel protection affect whether a bag is awarded, not the displayed tier distribution.

The small vanilla-style information button in the upper-right of the table opens the in-game reward guide. It explains team levels, handicaps, soft pity, protection, bags, and Luck, then returns to the unfinished setup without cancelling the session.

## High-tier announcements

When `announcements.announceHighTiers` is enabled, every online player receives a colored message when somebody is awarded an Epic or Legendary bag. Set it to `false` for private or quieter servers.

## Using modded items

Any registered item may be used:

```json
{
  "type": "minecraft:item",
  "name": "cobblemon:ability_capsule",
  "weight": 5
}
```

The owning mod must be installed when the datapack loads. A missing item ID can prevent that loot table from loading.

## Player Luck inside a bag

Bag tier is rolled from trainer level and handicap score. Afterward, the player opening the bag supplies normal Minecraft Luck to the selected loot table. Add `quality` to entries that should react to it:

```json
{
  "type": "minecraft:item",
  "name": "minecraft:iron_ingot",
  "weight": 20,
  "quality": -1
},
{
  "type": "minecraft:item",
  "name": "minecraft:diamond",
  "weight": 3,
  "quality": 4
}
```

Positive quality makes an entry more likely with positive Luck. Negative quality gradually removes basic entries from the effective weight. The five included `basic.json` tables already use this system.

## Troubleshooting

If a custom file is not detected:

1. Confirm the folder is `loot_table`, singular.
2. Confirm the path contains `cobblecoop/rewards/<tier>/`.
3. Use only `common`, `uncommon`, `rare`, `epic`, or `legendary` as the tier folder.
4. Check the JSON syntax and all item IDs.
5. Confirm the datapack appears in `/datapack list`.
6. Run `/reload` and check the server log for `Loaded ... automatic reward pool(s)`.

`/cobblecoop reload` reloads TOML configuration. Standard `/reload` reloads datapacks and rebuilds the automatic reward index.
