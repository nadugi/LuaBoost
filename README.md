# ⚡ LuaBoost v1.1.0 (WotLK 3.3.5a)

**Lua runtime optimizer + SmartGC + SpeedyLoad for World of Warcraft 3.3.5a (build 12340)**
Author: **Suprematist**

LuaBoost improves addon performance by optimizing common Lua patterns, eliminating GC stutter with per-frame incremental garbage collection, and speeding up loading screens by suppressing noisy events.

Designed for **Warmane** and other 3.3.5a servers.

---

## ✅ Features

### Runtime Optimizations (automatic, always active)
- Faster `math.floor`, `math.ceil`, `math.abs` (pure Lua replacements)
- Faster `table.insert(t, v)` for append case
- `GetTimeCached()` — cached `GetTime()` value updated once per frame
- `LuaBoost_Throttle(id, interval)` — shared throttle helper for any addon
- Table pool: `LuaBoost_AcquireTable()` / `LuaBoost_ReleaseTable(t)` / `LuaBoost_GetPoolStats()`
- `GetDateCached(fmt)` — opt-in cached date helper (does **not** replace global `date()`)

### Smart GC Manager (configurable)
- Stops Lua auto-GC and performs **incremental GC steps every frame**
- **4-tier stepping**: loading → combat → idle → normal
  - **Loading**: aggressive cleanup (no rendering happening anyway)
  - **Combat**: minimal GC to protect frametime
  - **Idle**: aggressive cleanup while AFK
  - **Normal**: balanced stepping
- Emergency full GC when Lua memory exceeds threshold (outside combat)
- GUI: `ESC → Interface → AddOns → LuaBoost`

### SpeedyLoad — Fast Loading Screens
- Temporarily suppresses noisy events during loading screens
- Restores all events after loading completes (with event replay)
- **Two modes**:
  - **Safe** (11 events) — cosmetic events only, no risk
  - **Aggressive** (23 events) — includes actionbar/aura/inventory events, faster but slightly more aggressive
- Based on the approach from KPack/SpeedyLoad by Kader, with improvements:
  - Uses `GetFramesRegisteredForEvent()` for targeted suppression
  - Hooks `UnregisterEvent` to track removals during loading
  - Replays missed events after restore
  - Ensures LuaBoost's handler fires first via priority re-registration

### Protection Hooks (optional, enabled by default)
- **Intercept `collectgarbage()`** — blocks other addons from forcing full GC spikes
- **Block `UpdateAddOnMemoryUsage()`** — prevents CPU spikes from frequent memory scans

### DLL Integration (optional)
Works with **[wow_optimize.dll](https://github.com/suprepupre/wow-optimize)**:
- DLL does GC stepping from C (via Sleep hook) — zero Lua overhead
- DLL reads `LUABOOST_ADDON_COMBAT`, `LUABOOST_ADDON_IDLE`, `LUABOOST_ADDON_LOADING` globals
- DLL adjusts step size based on 4-tier state
- LuaBoost remains the UI/logic/settings layer
- Stats available via `LuaBoostC_GetStats()`

---

## ⚠️ Conflicts

### SmartGC
**Do NOT use SmartGC together with LuaBoost.**
SmartGC has been integrated into LuaBoost. Using two GC managers simultaneously will conflict.

SmartGC repo: https://github.com/suprepupre/SmartGC

### KPack SpeedyLoad
**Disable KPack's SpeedyLoad module if you enable LuaBoost's SpeedyLoad.**
Both addons suppress the same events during loading screens. Running both will cause double-suppression and events may not restore correctly.

To disable in KPack:  SpeedyLoad in KPack's module settings or addons folder and disable/remove it.

If you keep LuaBoost SpeedyLoad **disabled** (default), KPack SpeedyLoad can remain active — they won't conflict.

---

## 📦 Installation

### Recommended (early load order)
LuaBoost must be loaded first, so the `!` prefix is ​​used:
- Recommended combo: **LuaBoost + wow_optimize.dll**

Copy the addon folder:

Interface/AddOns/!LuaBoost/

├── !LuaBoost.toc

└── LuaBoost.lua


If you downloaded from GitHub, make sure the extracted folder is `!LuaBoost`.

Restart WoW or `/reload`.

---

## 🧰 Commands

| Command | Description |
|---------|-------------|
| `/lb` or `/luaboost` | Status overview |
| `/lb bench` | Run performance benchmark |
| `/lb gc` | GC stats + DLL stats (if present) |
| `/lb pool` | Table pool stats |
| `/lb toggle` | Enable/disable GC manager |
| `/lb force` | Force full GC now |
| `/lb sl` | Toggle SpeedyLoad on/off |
| `/lb sl safe` | Enable SpeedyLoad in safe mode (11 events) |
| `/lb sl agg` | Enable SpeedyLoad in aggressive mode (23 events) |
| `/lb settings` | Open GC settings panel |
| `/lb help` | Show all commands |

---

## 🛡️ Protection options (important)

LuaBoost includes two optional “protection” features enabled by default:

- **Intercept collectgarbage() calls**  
  Blocks other addons from forcing full GC spikes.
- **Throttle UpdateAddOnMemoryUsage()**  
  Reduces CPU spikes from frequent memory scans.

On some UI setups these hooks may cause “macro script blocked / taint” warnings.
If that happens, disable them in `Tools & Diagnostics` panel.

---

## 📊 SpeedyLoad Event Lists

### Safe Mode (11 events)

SPELLS_CHANGED, SPELL_UPDATE_USABLE, ACTIONBAR_SLOT_CHANGED,
USE_GLYPH, PLAYER_TALENT_UPDATE, PET_TALENT_UPDATE,
WORLD_MAP_UPDATE, UPDATE_WORLD_STATES, UPDATE_FACTION,
CRITERIA_UPDATE, RECEIVED_ACHIEVEMENT_LIST


### Aggressive Mode (23 events)
All safe events plus:

ACTIONBAR_UPDATE_STATE, ACTIONBAR_UPDATE_USABLE,
ACTIONBAR_UPDATE_COOLDOWN, SPELL_UPDATE_COOLDOWN,
UNIT_AURA, UNIT_INVENTORY_CHANGED, BAG_UPDATE,
QUEST_LOG_UPDATE, COMPANION_UPDATE, PET_BAR_UPDATE,
TRADE_SKILL_UPDATE, MERCHANT_UPDATE

## ✅ Compatibility

- **WoW**: 3.3.5a (Interface 30300)
- **Tested**: Warmane (Lordaeron / Icecrown)
- **Lua**: 5.1 (embedded in WoW client)
- **Works with**: DXVK, LAA patch, wow_optimize.dll
- **Conflicts with**: SmartGC addon, KPack SpeedyLoad module (if both SpeedyLoad features are enabled)

---

## 📜 License

MIT License — do whatever you want with it.