# Ryu Garage — 2Step

Two-step rev limiter and antilag for FiveM: exhaust flames, pops and bangs, launch
control and popcorn mode, all saved per plate.

**[▶ Buy on Tebex](TEBEX_LINK_HERE)** · **[📺 Video preview](VIDEO_LINK_HERE)**

> This repository holds the **documentation only**. The resource itself is delivered
> through Tebex.

---

## What it does

Hold the clutch, bounce off the limiter, and the exhaust answers. The player tunes the
intensity, the flame size and colour, the exhaust note and the launch control from a
tablet panel, and it all sticks to the plate.

## Features

### The bang

- **Three intensity levels**, each with its own cooldown, flame size, chance and speed
  window
- **Popcorn mode** — rapid-fire backfire bursts, with configurable chance and cooldown
- **Launch control** with adjustable RPM limiter and intensity
- **Silent mode** for flames with no sound, and **hide flames** for sound with no fire
- Multiple **exhaust sound types**, with custom audio shipped in the resource

### Looks

- **6 flame colours**, plus flame size per vehicle
- **Two UI themes** (Classic and Modern) and a free accent colour picker, saved per player
- Draggable panel

### Presets

**Three hotbar slots per vehicle**, swapped in-game without opening the menu.

## Requirements

- [`ox_lib`](https://github.com/overextended/ox_lib)
- [`oxmysql`](https://github.com/overextended/oxmysql)
- **MySQL 5.7+** or **MariaDB 10.2+**

The table is created automatically on first start, and missing columns are added on
upgrade. `2step_sql.sql` is included for a manual import.

## Frameworks

| Framework | Guide |
|---|---|
| vRP, Creative Network, Creative V5 | [INSTALL_VRP.md](INSTALL_VRP.md) |
| VRPeX | [INSTALL_VRPEX.md](INSTALL_VRPEX.md) |
| QBCore | [INSTALL_QB.md](INSTALL_QB.md) |
| ESX | [INSTALL_ESX.md](INSTALL_ESX.md) |
| Standalone | `Config.Framework = "standalone"` — no permission checks |

A dedicated guide for **ESX + Quasar Inventory** ships with the resource
(`QUASAR.md`), covering the item being consumed on install.

## Built for a live server

- **Per-plate persistence** with a `fake_plate` alias — a cloned plate never loses the
  antilag installed on the original
- **Rate limited server-side**, with every write validated and the plate read from the
  entity rather than trusted from the client
- **Blank plates are refused**: a vehicle with no plate cannot be told apart from any
  other plateless vehicle, so the resource declines to save instead of silently sharing
  one row between every car on the server
- **Sound and flames synced** to everyone nearby through statebags
- Threads idle high and only tighten while the effect is running

## Integration

```lua
exports['rgarage_2step']:OpenMenu()                     -- client
exports['rgarage_2step']:LoadOnSpawn(vehicle)           -- client, from your garage
exports['rgarage_2step']:SetBackfire(netId, state, sound, color)   -- server
exports['rgarage_2step']:SyncBackfireEffect(netId, exhaust, color, isTwoStep, hide, size)
```

## What is open

`config.lua`, `Locales.lua`, the SQL file, every framework bridge and the whole UI ship
unencrypted. Core logic is escrow protected.

## Support

Found a bug or need help with the install? Open an issue in this repository.
