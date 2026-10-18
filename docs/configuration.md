# Configuration

`plugins/CrossplayShield/config.yml`. Apply changes with `/cs reload`. New keys are added automatically to existing
files when you update the plugin. Wrong values are reported in the server log: an unknown action becomes `block`,
an unknown packet type, a limit that is not a positive number or an invalid regular expression is ignored.

Each protection has an `action`:

| Action | Effect |
|---|---|
| `block` | Block (default) |
| `log` | Only write a line in the server log |
| `kick` | Block and kick the player |

A protection writes at most one log line (and one alert) per player per second, even during a flood.

## General

| Key | Default | Description |
|---|---|---|
| `language` | `en` | Language of the server log, and of players whose game language has no text file: `en`, `fr`, or a language you added in `plugins/CrossplayShield/lang/` |
| `debug` | `false` | Every 5 s, log the peak packet rates of each player and the cost of the packet listener. For calibration only |
| `update-check` | `true` | At startup, ask GitHub whether a newer version exists and tell admins when they join. One request per startup, nothing is sent about your server |
| `dry-run` | `false` | `true`: nothing is blocked, everything is logged with a `[dry-run]` prefix. Leave it on for a day after installing |
| `bedrock.multiplier` | `3.0` | Packet limits are multiplied by this for Bedrock players (detected by Floodgate). Values below 1 are replaced by 1 |

## Packet limits

| Key | Default | Description |
|---|---|---|
| `packet-limits.enabled` | `true` | Limit packets per type |
| `packet-limits.action` | `block` | `block` drops the packets above the limit, `log` only logs them, `kick` kicks at the first excess |
| `packet-limits.kick-on-flood` | `true` | With `block`: kick a player who keeps sending more than twice the limit for 5 seconds |
| `packet-limits.per-second` | see below | Maximum packets per second of each type for Java players |

The window is one second, sliding. Packets up to the limit always go through; only the extra ones are dropped.

| Type ([PacketEvents name](https://github.com/retrooper/packetevents)) | Default | Java action |
|---|---|---|
| `CLICK_WINDOW` | 162 | Inventory click |
| `CREATIVE_INVENTORY_ACTION` | 60 | Item taken from the creative inventory |
| `EDIT_BOOK` | 2 | Book edit (Paper already kicks at the second edit within one second) |
| `UPDATE_SIGN` | 4 | Sign edit |
| `NAME_ITEM` | 20 | Anvil rename (each typed letter) |
| `PLUGIN_MESSAGE` | 20 | Plugin channel message |
| `SELECT_BUNDLE_ITEM` | 20 | Bundle item selection |
| `PLAYER_DIGGING` | 60 | Block breaking, item drop, hand swap |
| `PLAYER_BLOCK_PLACEMENT` | 40 | Block placing, item use on a block |

You can add any other client-to-server packet type by its PacketEvents name. Paper already limits all packets
together (500 per second over 7 seconds by default); these limits are per type. `/cs audit` checks that Paper's
limiter is on, and that ViaVersion's own limiter does not kick Bedrock players while they sleep.

## Full edition

### Discord

| Key | Default | Description |
|---|---|---|
| `discord.webhook-url` | `""` | Discord webhook URL (channel → Edit → Integrations → Webhooks). Empty = disabled |
| `discord.min-interval-seconds` | `30` | Minimum delay between two messages for the same player and protection |

In-game alerts go to players with `crossplayshield.alerts`; each one can mute them with `/cs alerts`.

### Creative items

Items taken from the creative inventory by players without `crossplayshield.bypass.creative` are replaced by a
cleaned copy.

| Key | Default | Description |
|---|---|---|
| `creative-items.enabled` | `true` | Clean creative items |
| `creative-items.action` | `block` | `block` replaces the item, `log` only logs, `kick` replaces it and kicks the player |
| `creative-items.max-name-length` | `256` | Longer names are cut (in characters, an emoji counts as one) |
| `creative-items.max-lore-lines` | `50` | Extra lore lines are removed |
| `creative-items.strip-attributes` | `true` | Remove attribute modifiers |
| `creative-items.strip-nested-contents` | `true` | Empty the containers stored inside a container item (shulker box in a shulker box, bundle in a bundle…) |
| `creative-items.max-item-bytes` | `32768` | Items bigger than this once serialized are replaced by a plain item of the same type |

Enchantments above their vanilla maximum level are always brought back to it.

### Chunk limits

Counted when something arrives in the chunk. Nothing that is already there is ever removed.

| Key | Default | Description |
|---|---|---|
| `chunk-limits.enabled` | `true` | Limit per chunk |
| `chunk-limits.action` | `block` | `block` refuses the new item (an item dropping on the ground is then lost), `log` only logs |
| `chunk-limits.max-ground-items` | `200` | Items lying on the ground in one chunk |
| `chunk-limits.max-written-books` | `100` | Written books in the containers of one chunk (chests, barrels, shulker boxes placed as blocks…) |
| `chunk-limits.max-container-bytes` | `1048576` | Total size of the shulker boxes and bundles stored in the containers of one chunk, uncompressed (repetitive text that compresses well still counts in full) |

Player clicks, drags and hoppers are checked. `0` disables a limit.

### Command filter

Applies to players without `crossplayshield.bypass.commands`, Java and Bedrock alike.

| Key | Default | Description |
|---|---|---|
| `command-filter.enabled` | `true` | Filter commands |
| `command-filter.action` | `block` | `block` refuses the command, `log` only logs, `kick` refuses it and kicks the player |
| `command-filter.max-length` | `256` | Longer commands are refused (`0` = no limit) |
| `command-filter.block-namespaced` | `true` | Refuse `/plugin:command` forms, which skip other plugins' command checks, and hide them from the command autocompletion |
| `command-filter.allowed-namespaces` | `[]` | Namespaces still allowed, e.g. `[essentials]` |
| `command-filter.blocked` | `[]` | Regular expressions matched against the command without `/`, e.g. `"^(?i)pl(ugins)?\\b"` |

## Languages

`plugins/CrossplayShield/lang/en.yml` and `fr.yml` hold every text. Edit them, or copy one to `<code>.yml` (for
example `de.yml`) and translate it: players whose game is in that language get it. Missing keys fall back to
English. Colors use `§` codes.
