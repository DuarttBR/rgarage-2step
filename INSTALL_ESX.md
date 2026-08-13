# rgarage_2step — Installation on ESX

Covers the **ESX built-in inventory** and **ox_inventory**. For Quasar Inventory
see `QUASAR.md`, shipped with the resource.

---

## 1. Dependencies

Start these **before** `rgarage_2step` in `server.cfg`:

- `es_extended`
- `oxmysql`
- `ox_lib`

```cfg
ensure es_extended
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
Config.Framework = "esx"
Config.Language  = "en"   -- en, pt_br, es, fr, de, it, ru, zh, ja, ko
```

## 4. fxmanifest.lua — REQUIRED on ESX

Two lines must be **commented out**:

```lua
shared_scripts {
    'config.lua',
    'Locales.lua',
    '@ox_lib/init.lua',
    -- '@vrp/lib/utils.lua'   -- <= MUST stay commented
}

dependencies {
    'ox_lib',
    'oxmysql',
    -- 'vrp'                   -- <= MUST stay commented
}
```

There is no `vrp` resource on your server, so those paths are invalid and the resource
will not start. This is the most common reason it fails to boot.

## 5. Commands and permissions

```lua
Config.Commands = {
    enabled = true,
    name    = "2step",      -- /2step
    groups  = {}            -- empty = anyone can use
}
```

On ESX the key is matched against the permission group (`getGroup()`) **or** the job name — either one passing is enough:

```lua
groups = { ["mechanic"] = 0, ["admin"] = 0 }
```

With `Config.Installation.enabled = true` there is a second command,
`Config.Installation.command.name` (default `/antilag`), which only opens the panel on
vehicles that already have the antilag fitted.

## 6. The item

Suggested name: `antilag`. Two modes, set by `Config.Installation.enabled`:

- **`false`** — using the item opens the panel directly (player must be in the vehicle)
- **`true`** — the item **installs** the antilag on the car, with a progress bar

### ox_inventory

In `ox_inventory/data/items.lua`:

```lua
['antilag'] = {
    label = 'Antilag / 2-Step Kit',
    weight = 500,
    stack = true,
    close = true,
    description = 'Install or open 2-step on your vehicle',
    server = { export = 'rgarage_2step.useAntilagItem' }
},
```

### ESX built-in inventory

Add `antilag` to your `items` table, then in your own server file:

```lua
ESX = exports['es_extended']:getSharedObject()

ESX.RegisterUsableItem('antilag', function(src)
    TriggerEvent('rgarage_2step:server:useItem', src, 'antilag')
end)
```

> **Do not consume the item yourself.** The resource removes it only after the install
> completes, so a cancelled install costs the player nothing.

## 7. Notify and progress bar

```lua
Config.Notify   = { enabled = true, type = "ox_lib" }
Config.Progress = { enabled = true, type = "ox_lib" }
```

`ox_lib` is the right default on ESX for both.

## 8. Worth knowing before going live

**Blank plates are refused.** A vehicle with no licence plate cannot be told apart from
any other plateless vehicle, so the resource declines to save rather than sharing one
database row between every car on the server. If players report *"this vehicle has no
licence plate"*, your spawn system is not assigning one — and that affects every
plate-keyed resource, not just this one.

---

## Troubleshooting

**The panel does not open**
`@vrp/lib/utils.lua` or `'vrp'` is still active in the fxmanifest — both must be commented out on ESX.

**Permission always denied**
The key has to be the job name or the permission group, spelled as your server returns it.

**Settings are not saved**
Check the MySQL version (5.7+ / MariaDB 10.2+) and that the vehicle actually has a plate.
