# Roblox Game Development Rules & Security Standards

When developing or reviewing Roblox games and Luau code:

## 1. Network & Security Guardrails (Never Trust the Client)
- **Zero Client Trust**: RemoteEvent and RemoteFunction inputs are attacker-controlled. Always sanitize, type-check, and boundary-check parameters on the server.
- **Server Authority**: Critical state (player currency, inventory, health, cooldowns, XP, level) must calculate and reside solely on the server. Client scripts are strictly views and controllers.
- **Rate Limiting**: Implement debounce/rate-limiting per player on all incoming remote requests to prevent remote spam or exploitation.
- **Physics & Movement**: Never trust client positions for teleportation or sensitive triggers. Validate raycasts and distance server-side.

## 2. Data Persistence (DataStore)
- **ProfileService / Session Locking**: Always use atomic session-locking persistence patterns (e.g., ProfileService/ProfileStore) to avoid race conditions and item dupe bugs. Never use raw `SetAsync()` on player data.
- **Graceful Failure**: Always wrap DataStore calls (`GetAsync`, `UpdateAsync`) in `pcall()`. Implement exponential backoff retries when encountering service throttles.
- **ProcessReceipt**: For developer products, ensure idempotent receipt handling. Record purchases to DataStore before returning `ProductPurchaseDecision.PurchaseGranted`. If saving fails, return `ProductPurchaseDecision.NotProcessedYet`.

## 3. Architecture & Service Hierarchy
- `ServerScriptService`: Server-only logic, authoritative services, DataStores, anti-cheat.
- `ReplicatedStorage`: Shared ModuleScripts, configurations, RemoteEvents/RemoteFunctions, shared definitions.
- `StarterPlayer.StarterPlayerScripts`: Client controllers, input handlers, UI controllers, rendering logic.
- `StarterGui`: UI ScreenGuis (auto-cloned to PlayerGui).
- `ServerStorage`: Server-only templates, weapons/tools spawned into workspace or backpack on demand.
- `Workspace`: Keep the 3D scene clean and streamable with `StreamingEnabled` on.

## 4. Memory & Performance Optimization
- **Connection Hygiene**: Store all `:Connect()` connections and cleanly `:Disconnect()` them on object destruction or player leaving. Use `Maid` or `Trove` patterns.
- **Loop Yielding**: Never run tight loops without explicit yields (`task.wait()`). Prefer event-driven architecture over polling loops.
- **Replication Efficiency**: Avoid replicating high-frequency bulk updates. Transmit delta updates or state flags instead of continuous coordinates.
