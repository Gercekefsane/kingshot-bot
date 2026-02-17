---
title: Kingshot Bot Technical Architecture - Python, PostgreSQL, Telegram, Discord
description: Technical documentation for Kingshot Bot covering system architecture, tech stack, Cloudflare bypass, dual-game engine, database schema, and API integration
keywords: kingshot bot architecture, python telegram bot, discord bot, postgresql, async python, cloudflare bypass, cloudscraper, kingshot api, century games bot, dual game bot, kingshot gift code api
---

# 👑 Kingshot Bot — Technical Architecture

## System Overview

```
┌───────────────────────────────────────────────────────────────┐
│                      WhiteBot Server                          │
│                                                               │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────────────┐ │
│  │  Telegram    │  │   Discord    │  │   Background Tasks   │ │
│  │  Bot API     │  │   Bot API   │  │                      │ │
│  │  (PTB v21)   │  │  (dpy v2)   │  │  • Gift Code Loop   │ │
│  │              │  │              │  │  • Control Loop      │ │
│  │  Commands    │  │  Slash Cmds  │  │  • Retry Queue       │ │
│  │  Callbacks   │  │  Events     │  │  • Event Scheduler   │ │
│  └──────┬───────┘  └──────┬──────┘  └──────────┬───────────┘ │
│         │                 │                     │             │
│         └────────────┬────┴─────────────────────┘             │
│                      │                                        │
│              ┌───────▼────────┐                               │
│              │   PostgreSQL   │                               │
│              │   Database     │                               │
│              └────────────────┘                               │
└───────────────────────────────────────────────────────────────┘
```

## Kingshot-Specific Architecture

### Cloudflare Bypass
The Kingshot gift code sources are protected by Cloudflare.
Standard HTTP libraries (`requests`, `aiohttp`) receive 403 errors.

**Solution**: `cloudscraper` library mimics browser TLS fingerprints, allowing the bot to access Cloudflare-protected endpoints reliably. Since `cloudscraper` is synchronous, it runs in a thread executor to maintain async compatibility:
```python
loop = asyncio.get_event_loop()
status, data = await loop.run_in_executor(None, _fetch_sync)
```

### API Integration
The bot communicates with Century Games' official Kingshot APIs:

| Function | Description |
|----------|------------|
| Player Lookup | Fetch player info by FID |
| Gift Code Redemption | Redeem codes per member |
| Code Discovery | Scan official channels for active codes |
| Request Signing | MD5-based signature for secure API calls |

### Dual-API Check
When registering a member, the bot queries **both** APIs concurrently:
```python
async def fetch_user_data_dual(fid, alliance_game_type):
    wos_task = asyncio.create_task(fetch_user_data(fid, GAME_WOS))
    ks_task = asyncio.create_task(fetch_user_data(fid, GAME_KS))
    wos_result, ks_result = await asyncio.gather(wos_task, ks_task)
    # Returns data from preferred game, or whichever succeeds
```

## Core Technologies

### Python 3.11+
- **Async/await** throughout the entire codebase
- `asyncio` for concurrent task management
- Thread executors for synchronous library integration

### python-telegram-bot v21
- Async-native Telegram Bot API wrapper
- `ConversationHandler` for multi-step flows
- `InlineKeyboardMarkup` for interactive menus
- HTML parse mode for rich message formatting

### discord.py v2.x
- Slash command integration (`app_commands`)
- `discord.Embed` for rich message display
- Cross-thread communication with `run_coroutine_threadsafe`

### PostgreSQL 15+
- Primary data store for all bot data
- Composite primary keys: `(fid, game_type)`
- `game_type` column on all relevant tables (`'wos'` / `'ks'`)
- `ON CONFLICT` upserts for idempotent operations

### cloudscraper
- Cloudflare TLS fingerprint bypass
- Critical for Kingshot gift code API access
- Run in thread executor for async compatibility

### aiohttp
- Async HTTP client for game API calls
- Concurrent API requests with `asyncio.gather`

## Module Architecture

```
whitebot/
├── bot.py                  # Main entry, handlers, dual-bot startup
├── config.py               # API keys, tokens, URLs (WOS + KS)
├── shared_state.py         # Cross-module bot references
├── modules/
│   ├── common.py           # Shared utils, dual-API, game configs
│   ├── registration.py     # /register, dual-API user registration
│   ├── members.py          # /addmember, /bulkadd, /profile
│   ├── alliance.py         # /setupalliance (game type selection)
│   ├── giftcode.py         # Code discovery, validation, redemption
│   ├── control.py          # Member monitoring, change detection
│   ├── idchannel.py        # ID channel auto-registration
│   └── member_tracking.py  # Premium tracking, rate limiting
├── locales/
│   ├── en.json             # English
│   ├── tr.json             # Turkish
│   └── ru.json             # Russian
└── db/
    └── migrate_game_type.py
```

## Key Design Decisions

### Game-Type Aware Database
Every table includes a `game_type` column (`'wos'` or `'ks'`):
- Single database serves both games
- Composite primary keys prevent FID collisions
- Queries are always scoped by `game_type`
- Gift codes are game-specific

### Single Process, Dual Bot
Both Telegram and Discord bots run in the same Python process:
- Telegram uses the main `asyncio` event loop
- Discord runs in a separate thread
- `shared_state.py` bridges between them

### Locale System
- JSON-based locale files (EN/TR/RU)
- `get_text(key, lang)` for string retrieval
- Per-group language settings
- Fallback to English if key missing

## External APIs

| API | Purpose | Auth | Bypass |
|-----|---------|------|--------|
| Kingshot Player API | Player info | Encrypt key | — |
| Kingshot Gift Code API | Redemption | Encrypt key | — |
| Official KS Channels | Code list | — | cloudscraper |
| WOS Player API | Player info | Sign + FID | — |
| WOS Gift Code API | Redemption | Sign + CAPTCHA | — |
| Official WOS Channels | Code list | — | aiohttp |
| CAPTCHA Service | CAPTCHA solving | API key | — |
