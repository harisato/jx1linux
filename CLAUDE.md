# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

JX1Linux (JxOffline 1) is an offline game server and client for the MMORPG "Vo Lam Truyen Ky 1" (剑侠情缘 / Sword Heroes' Fate), version 8.x. The project is maintained by the Vietnamese community "Hoi Quan Vo Lam". All game logic is written in **Lua** scripts. The repository contains no build system, test runner, or linter — development is done by editing Lua scripts and INI/TXT configuration files directly.

## Architecture

The codebase is a **client-server** game system with two main components:

### Client (`client/`)
- **Game.exe / game_y.exe**: Windows game client executables (precompiled binaries, not built from source here)
- **script/**: Client-side Lua scripts (~22 files)
  - `protocol.lua`: Defines client-server script protocol enums and registration
  - `skill/`: Per-faction skill definitions (huashan, shaolin, emei, gaibang, wudang, wudu, tangmen, kunlun, tianwang, tianren, cuiyan, xiaoyao)
  - `global/meridian/`: Meridian (kinh mach) system client logic
  - `event/prize/`: Client-side event/prize scripts
  - `ui/`: Client UI scripts (icon bar, etc.)
  - `miniskill/`: Mini skill UI
- **settings/**: Game configuration data (items, NPCs, NPC resources, meridian, pets, partner/companion)
- **ui/**: UI layout definitions (`.ini` files, Chinese-encoded filenames)
- **spr/**: Sprite/graphic resources (items, NPC appearances, UI elements)
- **maps/**: Map data files organized by map name in `map_publish/`
- **lang/vn/**: Vietnamese localization (string tables, NPC/object name replacements)
- **config.ini**: Client connection config (server port 5622, graphics settings)
- **Auto/**: Auto-play/bot tools (Auto.exe, AutoPK.exe)

### Server (`server/jxser/`)

#### Gateway (`server/jxser/gateway/`)
The network gateway layer handling player connections, account management, and routing:
- **bishop.cfg**: Gateway network config — connects Account server (port 5002), Role server (port 5001), client port (5622), game server port (5632)
- **goddess_y / bishop_y**: Gateway daemon binaries
- **KG_SyncD / BishopConn / backupdaemon**: Support daemons for sync, connectivity, and backup
- **s3relay/**: S3 relay server components
- **rolevalue_setting/**: Role value ladder/ranking configuration

#### Game Server (`server/jxser/server1/`)
The main game logic server (runs on CentOS):
- **server_start**: Game server binary
- **servercfg.ini**: Server network config (gateway, database, transfer, chat, tong ports — all on localhost)
- **script/**: Server-side Lua scripts (~6900 files) — this is where most game logic lives
  - **lib/**: Core utility libraries (`include.lua` defines the `Include()` module loader, `basic.lua`, `common.lua`, `player.lua`, `say.lua`, `award.lua`, `file.lua`, `log.lua`, `mem.lua`, `compose.lua`, etc.)
  - **global/**: Auto-executed initialization scripts (`autoexec.lua` is the main entry point that loads everything via `Include()` and runs `main()`)
  - **skill/**: Server-side skill system scripts
  - **item/**: Item use scripts (one file per item type)
  - **task/**: Quest/task system scripts
  - **missions/**: Mission system (tianchimijing, tongwar, leaguematch, citydefence, etc.)
  - **maps/**: Map-specific scripts and world scripts
  - **ai/**: NPC AI scripts (fighter behavior)
  - **battles/**: Battle system scripts
  - **shop/**: Shop system
  - **tong/**: Guild (Tong) system
  - **petsys/**: Pet system
  - **partner/**: Companion system
  - **event/**: Event scripts (seasonal events, festivals)
  - **activitysys/**: Activity/event system with NPC dialogs, timers, awards
  - **vng_feature/**: Vietnam-specific features (skill training, lenhbai/quests, battles, item management)
  - **vng_lib/**: Vietnam-specific utility libraries (vngapi, vngaward, vngtranslog, extpoint, bittask)
  - **vng_event/**: Vietnam-specific events
  - **class/**: Class/faction-related scripts
  - **giftcode/**: Gift code redemption system
  - **honor/**: Honor system
  - **worldrank/**: World ranking system
  - Region-specific scripts in Chinese-encoded directory names (东北区, 江南区, 两湖区, etc.)
- **vng_script/**: Additional Vietnam scripts (activitysys configs by ID like 1021-1036, events with award data)
- **settings/**: Game balance data (items, NPCs, drops, factions, maps, battles, skills, shops, missions, weather, meridian, etc.)
- **maps/**: Map configuration (mapgs_01.ini through mapgs_08.ini, worldset.ini)
- **lang/**: Localization files (both `zh/` Chinese and `vn/` Vietnamese)
- **data/**: Runtime data (lottery, gift codes)

## Key Technical Details

### Lua Script System
- Scripts use a custom `Include()` function (defined in `script/lib/include.lua`) that resolves paths relative to the script root using backslash (`\`) Windows-style paths
- Entry point: `script/global/autoexec.lua` — loads all subsystems and defines `main()`
- `DynamicExecute()` is used to invoke scripts at runtime
- Region detection: `GetProductRegion()` returns `"vn"` for Vietnam-specific code paths
- Vietnamese text in scripts uses **TCVN3** encoding (not UTF-8). Use JXStudio or JXLuaEditor to edit without breaking Chinese characters

### Configuration Files
- `.ini` files: Server/client network config, game settings, account setup
- `.txt` files: Data lists (NPC lists, skill lists, quest configs, event subscribers)
- `.cfg` files: Gateway/daemon configuration
- Map files use Chinese GB2312/GBK encoding for directory names

### Network Architecture
- Client connects to Gateway on port **5622**
- Gateway (`bishop`) routes to: Account server (5002), Role/DB server (5001), Transfer (5003), Chat (5004), Tong/Guild (5005)
- Game server listens on port **6666**, connected via Gateway port **5632**
- Default server IP: **192.168.1.100** (VMware virtual network)

## Branch Naming Convention

Branches and PR names use lowercase without diacritics, separated by dots (`.`), with a prefix:
- **doc**: Documentation changes
- **bin**: Binary/tool changes in client or server
- **script**: Script and config file (`.lua`, `.ini`, `.txt`) changes
- **feat**: Large features spanning multiple categories

Example: `script.them-kim-ma-lenh-khi-danh-quai-9x`

## Deployment

The server runs inside VMware virtual machines:
1. **CentOS VM**: Runs the game server (`server/jxser/server1/server_start`) and related daemons
2. **Windows XP VM**: Provides the management interface (SecureCRT shortcuts for starting services)
3. Upload `server/jxser/` to the CentOS VM via WinSCP, then run the upgrade shortcut to apply changes
4. Start services in order: steps 1, 2, 3, then S1 via SecureCRT shortcuts
