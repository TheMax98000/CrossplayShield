# Commands and permissions

`/cs` is an alias of `/crossplayshield`.

| Command | Description |
|---|---|
| `/cs audit` | Check the Paper, ViaVersion and Floodgate settings, with the line to change for each problem |
| `/cs alerts` | Toggle in-game alerts for yourself (full edition) |
| `/cs info <player>` | Java or Bedrock (with the device), current packets per second, last blocks |
| `/cs exempt <player> [seconds]` | Exempt a player from every protection (default: 60 s), e.g. while you test a plugin |
| `/cs reload` | Reload `config.yml` and the language files |

| Permission | Default | Description |
|---|---|---|
| `crossplayshield.admin` | op | Use `/cs`; get a message on join when a new version is out |
| `crossplayshield.alerts` | op | Receive alerts in game (full edition) |
| `crossplayshield.bypass.creative` | nobody | Creative items are not cleaned (full edition) |
| `crossplayshield.bypass.commands` | nobody | Commands are not filtered (full edition) |
