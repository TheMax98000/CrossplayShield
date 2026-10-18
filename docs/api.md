# CrossplayShield — Developer API (full edition)

Your plugin can see what CrossplayShield is about to block, and let it through.

## Setup

The API ships inside the full plugin jar. Put `crossplayshield-<version>.jar` in your project (for example under
`libs/`) and compile against it without shading it:

```kotlin
// build.gradle.kts
dependencies {
    compileOnly(files("libs/crossplayshield-1.0.0.jar"))
}
```

```xml
<!-- pom.xml -->
<dependency>
  <groupId>tomokisan</groupId>
  <artifactId>crossplayshield</artifactId>
  <version>1.0.0</version>
  <scope>system</scope>
  <systemPath>${project.basedir}/libs/crossplayshield-1.0.0.jar</systemPath>
</dependency>
```

```yaml
# plugin.yml
softdepend: [CrossplayShield]
```

The free edition has no API: with `softdepend`, check that the class exists before you register your listener:

```java
boolean api;
try {
    Class.forName("tomokisan.crossplayshield.CrossplayShieldBlockEvent");
    api = true;
} catch (ClassNotFoundException e) {
    api = false;
}
```

## `CrossplayShieldBlockEvent`

Fired when a protection is about to block something, **before** the log line, the alerts and the Discord message.
Cancelling it lets the action through, and nothing is logged or sent.

| Method | Description |
|---|---|
| `getPlayer()` | The player, or `null` when there is none (items dropping on the ground of a chunk) or the player is not in the game yet |
| `getProtection()` | `PacketLimit`, `CreativeItems`, `ChunkLimits` or `CommandFilter` |
| `getDetail()` | Human-readable detail, as shown in the alert, e.g. `CLICK_WINDOW 243/s > 162` |
| `isObserveOnly()` | `true` in dry-run or with the `log` action: nothing will be blocked anyway |
| `setCancelled(boolean)` | Let the action through |

For the same player and protection, the decision is kept for one second: a flood of 1,000 packets per second fires
one event per second, and cancelling it lets the whole second through.

**Threads:** `PacketLimit` events are **asynchronous** (they come from the network thread). The others are
synchronous, on the player's thread (main thread on Paper, the player's region thread on Folia). Keep your listener
short and thread-safe.

## Example

```java
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import tomokisan.crossplayshield.CrossplayShieldBlockEvent;

public final class StaffBypass implements Listener {

    @EventHandler
    public void onBlock(CrossplayShieldBlockEvent event) {
        // Staff in build mode may paste large creative items
        if (event.getProtection().equals("CreativeItems") && event.getPlayer() != null
                && event.getPlayer().hasPermission("myplugin.buildmode")) {
            event.setCancelled(true);
        }
    }
}
```
