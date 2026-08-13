# rgarage_2step — Installation on VRPeX

---

## 1. Dependencies

Start these **before** `rgarage_2step` in `server.cfg`:

- `vrp` (VRPeX)
- `oxmysql`
- `ox_lib`

```cfg
ensure vrp
ensure oxmysql
ensure ox_lib
ensure rgarage_2step
```

## 2. Database

Run `2step_sql.sql` once. It creates `rgarage_2step_configs`, which holds the antilag
preset, flame size, colour, launch control and the three hotbar presets, per plate.

Missing columns are added automatically on start, so upgrading needs nothing else.

> Requires **MySQL 5.7+** or **MariaDB 10.2+**.

## 3. config.lua

```lua
Config.Framework = "vrpex"
Config.Language  = "pt_br"   -- en, pt_br, es, fr, de, it, ru, zh, ja, ko
```

## 4. fxmanifest.lua — required for VRPeX

Two lines must be **uncommented**:

```lua
shared_scripts {
    'config.lua',
    'Locales.lua',
    '@ox_lib/init.lua',
    '@vrp/lib/utils.lua'      -- <= REQUIRED
}

dependencies {
    'ox_lib',
    'oxmysql',
    'vrp'                      -- <= REQUIRED
}
```

Without them the bridge cannot reach vRP and the panel never opens.

> **Linux is case sensitive.** Some builds ship `lib/Utils.lua` with a capital U. Check
> your `vrp/lib/` folder and match the real filename.

## 5. Commands and permissions

```lua
Config.Commands = {
    enabled = true,
    name    = "2step",      -- /2step
    groups  = {}            -- empty = anyone can use
}
```

On VRPeX the bridge tries both `hasGroup` and `hasPermission`, and also splits dotted names, so all of these work:

```lua
groups = { ["Mechanic"] = 0, ["mechanic.antilag"] = 0, ["Admin"] = 0 }
```

With `Config.Installation.enabled = true` there is a second command,
`Config.Installation.command.name` (default `/antilag`), which only opens the panel on
vehicles that already have the antilag fitted.

## 6. The item

Suggested name: `antilag`. Two modes, set by `Config.Installation.enabled`:

- **`false`** — using the item opens the panel directly (player must be in the vehicle)
- **`true`** — the item **installs** the antilag on the car, with a progress bar

Add `antilag` to your itemlist, and fire this from its use handler:

```lua
TriggerEvent('rgarage_2step:server:useItem', source, 'antilag')
```

> **Do not consume the item yourself.** The resource removes it only after the install
> completes, so a cancelled install costs the player nothing.

## 7. Notify and progress bar

```lua
Config.Notify   = { enabled = true, type = "ryu" }
Config.Progress = { enabled = true, type = "event" }
```

`"ryu"` fires `ryu_hud:notification`; `"event"` fires `TriggerEvent("Progress", duration)`, the usual vRP pattern.

## 8. Worth knowing before going live

**Blank plates are refused.** A vehicle with no licence plate cannot be told apart from
any other plateless vehicle, so the resource declines to save rather than sharing one
database row between every car on the server. If players report *"this vehicle has no
licence plate"*, your spawn system is not assigning one — and that affects every
plate-keyed resource, not just this one.

---

## Troubleshooting

**The panel does not open**
`Config.Framework` is not `"vrpex"`, or `@vrp/lib/utils.lua` is still commented in the fxmanifest.

**Permission always denied**
The bridge tries group and permission. If both fail, the key is not the name your base uses. Case matters.

**Settings are not saved**
Check the MySQL version (5.7+ / MariaDB 10.2+) and that the vehicle actually has a plate.
