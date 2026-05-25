# Arrow AntiCheat

## Build

- **Maven** (`pom.xml`), Java 16 target
- `mvn clean package` — shades with maven-shade-plugin, relocates `org.apache.commons.math3`, `it.unimi.dsi.fastutil`, `org.apache.commons.lang3` to `me.arrow.libs.*`
- No test suite (no `src/test/`)

## Entrypoints

- **plugin.yml** → `me.arrow.ArrowLoader` (JavaPlugin)
- `ArrowLoader.onEnable()` → creates `me.arrow.Arrow` (not a JavaPlugin, singleton via `getInstance()`) → calls `arrow.onEnable()`
- All initialization happens in `Arrow.onEnable()` — runs in a delayed task (10 ticks) after server start

## Version

- Hardcoded in `Arrow.java` (`version = "107-pre1"`) and `plugin.yml`

## Architecture

- **Per-player data**: `Profile` → `MovementData`, `CombatData`, `RotationData`, `ConnectionData`, `VelocityData`, etc. in `me.arrow.playerdata.data.impl.*`
- **Checks**: extend `me.arrow.checks.types.Check` (extends `AbstractCheck`), implement `handle(PacketReceiveEvent)` and `handle(PacketSendEvent)`
  - Annotations: `@Testing`, `@Experimental`, `@Development` — `fail()` silently returns for `@Development`
- **Packets**: PacketEvents 2.12.1 — `NetworkListener` handles incoming/outgoing, routes to per-profile checks
- **NMS abstraction**: `NmsManager` → `NmsInstance` interface → `InstanceDefault` (Bukkit API fallback). Implement per-version for performance
- **Config**: Custom `CommentedFileConfiguration` (YAML with comments). `Config.Setting` enum for `config.yml`, `Checks` for per-check config
- **Repeating tasks**: `TickTask` (50ms async), `ViolationTask` (VL reset timer), `LogsTask` (log archival every 5 min)
- **Threading**: `ThreadManager` manages per-player `ProfileThread`

## Key Dependencies (provided — must be on server)

- PacketEvents 2.12.1 (Spigot)
- Spigot API 1.21 (26.1.2)
- ViaVersion API (optional)
- Geyser/Floodgate API (optional — bedrock support)
- Velocity API 3.1.1 (optional)

## Commands

- `/arrow` — main command, registered in `CommandManager`
- `/stuck` — teleport to spawn (only in **test server mode**)

## Test Server Mode

- Config: `test_server_mode.enabled` — when on: max VL = 50, `/stuck` enabled, damage prevention, build zone, custom world

## Conventions

- `me.arrow` root package
- Checks use `buffer` (double) + VL (int) threshold system; punish when VL > maxVl
- Config via `Config.Setting.XYZ.getBoolean()`, `.getInt()`, `.getString()`, etc.
- Webhook integration for alerts/punishments (Discord)
- No bypass permissions (commented out in code)
- Shaded libs go to `me.arrow.libs.*`
