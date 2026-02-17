---
title: Kingshot Bot Features - Gift Code Redemption, Alliance Management, Member Tracking
description: Complete feature documentation for Kingshot Bot including automatic gift code redemption, Cloudflare bypass, alliance member management, real-time monitoring, and dual-game support
keywords: kingshot bot features, kingshot gift code bot, automatic gift code redemption, kingshot alliance management, member tracking, furnace tracker, kingshot telegram bot, kingshot discord bot, century games bot, kingshot coupon bot
---

# 👑 Kingshot Bot — Feature Details

## 🎁 Gift Code System

### How Auto-Discovery Works
1. The bot scans **official Kingshot channels** every 60 minutes
2. Uses **advanced Cloudflare bypass** for uninterrupted access
3. New codes are stored in PostgreSQL with `pending` status
4. Codes are automatically redeemed for **every registered member**
5. Notifications are sent and pinned in alliance groups

### Code Lifecycle
```
Discovered → Pending → Validated → Auto-Used → Expired/Invalid
```

### Kingshot Redemption API
The bot uses Century Games' official gift code API:
- **Encrypted request signing** for secure communication
- **Player lookup** by FID via official API
- **Code redemption** with per-member tracking
- **Rate limiting** with built-in delays between requests

### Smart Retry System
- Failed redemptions are queued for retry
- Exponential backoff prevents rate limiting
- Maximum retry attempts configurable
- Detailed error tracking per member per code

---

## 👥 Member Management

### Dual-API Registration
When a member registers, the bot checks **both** game APIs concurrently:
1. WhiteOut Survival API
2. Kingshot API

The bot automatically determines which game the player belongs to based on:
- The alliance's configured game type (preferred)
- Which API returns valid player data

### Registration Methods
1. **Command**: `/register 123456789` — direct FID registration
2. **ID Channel**: Set a channel where typing any FID auto-registers the member
3. **Bulk Add**: `/bulkadd 111 222 333 TAG` — add multiple FIDs at once
4. **Admin Add**: `/addmember 123456789 TAG` — admin registers on behalf

### What Gets Tracked
| Field | Description | Change Notification |
|-------|-------------|-------------------|
| **Nickname** | In-game display name | ✅ Yes |
| **Furnace Level** | FC level (e.g., FC 30) | ✅ Yes |
| **State** | Server/state number | ✅ Yes |
| **Alliance** | Current in-game alliance | ✅ Yes |

---

## 📊 Monitoring & Control

### Control Loop
The control loop runs at configurable intervals (default: 60 minutes):
1. Fetches live data for each registered member from the Kingshot API
2. Compares with stored data in the database
3. Detects changes in nickname, furnace level, or state
4. Sends formatted notifications to the alliance's Telegram/Discord channel
5. Updates the database with new values

### Game-Type Aware
All operations are scoped by game type:
- Kingshot alliances only see Kingshot codes
- WhiteOut Survival alliances only see WOS codes
- The `/codes` command shows game icons (👑 / ❄️) per code

---

## 🌐 Multi-Language Support

### Supported Languages
| Language | Code | Coverage |
|----------|------|----------|
| English | `en` | 100% |
| Turkish | `tr` | 100% |
| Russian | `ru` | 100% |

### How It Works
- Each group/user can set their preferred language
- All bot messages, buttons, and notifications are localized
- Language can be changed with `/language` or `/setlang`

---

## 🔄 Cross-Platform

### Telegram Features
- Inline keyboards for navigation
- HTML-formatted messages
- Group and private chat support
- Message pinning for gift code notifications
- ID Channel for auto-registration

### Discord Features
- Slash commands (`/register`, `/help`, etc.)
- Embed messages with rich formatting
- Server and channel-based alliance linking
- Message pinning for gift code alerts
