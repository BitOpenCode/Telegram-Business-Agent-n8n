# Secretary Agent - Main Workflow

## 📋 Overview

The **Secretary Agent** is the main entry point workflow for your Telegram Business Bot. It receives all incoming messages from Telegram, routes them to appropriate sub-workflows based on command type, and handles webhook responses.

**This workflow must always be active!**

**Version:** 1.1

---

## 🏗️ Workflow Architecture

```
Telegram Business API → Webhook → Route Commands (Switch) → Sub-workflows → 200 OK Response
```

---

## 📁 File Information

| Property | Value |
|----------|-------|
| **Active Status** | `true` (always on) |
| **Version** | 1.1 |

---

## 🔧 Prerequisites

1. ✅ **n8n instance** (self-hosted or cloud)
2. ✅ **Telegram Bot** created via @BotFather
3. ✅ **Telegram Business** account
4. ✅ **Sub-workflows** imported

---

## 🚀 Quick Start Guide

### Step 1: Create Webhook

1. Open the **Telegram Webhook** node
2. Copy the **Path** value
3. You'll need this for the webhook URL

### Step 2: Configure Telegram Bot

#### BotFather Setup:
1. Create bot with [@BotFather](https://t.me/botfather)
2. Pick your bot → **Bot Settings** → **Secretary mode** → **ON**
3. Send /start to your bot

#### Telegram Business Setup:
1. **Settings** → **Telegram Business** → **Chatbots** → **Chat Automation**
2. Add your bot (username/link)
3. Set permissions: Read, Reply, Mark as Read

### Step 3: Set Webhook

Execute the **Set Telegram Webhook** node with:

```
https://api.telegram.org/botYOUR_TOKEN/setWebhook?url=https://n8n.YOUR_DOMAIN.com/webhook/YOUR_PATH
```

**Expected response:**
```json
{"ok": true, "result": true, "description": "Webhook was set"}
```

### Step 4: Activate

Toggle workflow to **Active** (green).

---

## 🔀 Route Commands (Switch Node)

| Output | Condition | Routes To |
|--------|-----------|-----------|
| `test` | `executionMode == "test"` | Test |
| `Give me crypto rates` | `text == "Дай курсы криптовалют"` | TOP 10 Crypto |
| `Give me rates` | `text == "Дай курсы валют"` | TOP 10 Currency Rates |
| `Rate` | `text starts with "Курс"` | Rate with Chart |
| `Music` | `text starts with "Music"` | Music Search |
| `Музыка` | `text starts with "Музыка"` | Music Search |
| `email sent by user` | `entities[0].type exists` | Email sent by user |
| `message` | `text exists` (default) | Message flow |
| `/calls button` | `callback_data == "/calls"` | Message /calls button |
| `More Music` | `callback_data starts with "dzmore|"` | More Music |
| `=callback_query` | `callback_query exists` | Callback PocketAI |
| `ART create` | `photo[0].file_id exists` | ART create PocketAI |

---

## 🔗 Sub-workflows

| Node | Purpose |
|------|---------|
| **Message flow** | AI secretary, call booking, urgent messages |
| **TOP 10 Crypto** | Cryptocurrency prices from Binance |
| **TOP 10 Currency Rates** | Forex rates from TradingView |
| **Rate with Chart** | Rate lookup with TradingView charts |
| **PocketLine Music Search** | Deezer music search |
| **Email sent by user** | Handle user email input |
| **Message /calls button** | Handle /calls callback |
| **More Music** | Load more music results |
| **Callback PocketAI** | Generic callback router |
| **ART create PocketAI** | Generate T-shirt mockups from photos |
| **Test** | Test workflow |

> **Important:** Update workflow IDs in each Execute Workflow node after importing.

---

## 📊 Node Reference

### Core Nodes

| Node | Type | Description |
|------|------|-------------|
| **Telegram Webhook** | Webhook | Receives POST from Telegram |
| **Route Commands** | Switch | Routes messages by command |
| **Respond to Webhook** | Respond | Returns 200 OK |
| **Set Telegram Webhook** | HTTP Request | Sets webhook URL |

### Optional Nodes (Disabled)

| Node | Purpose |
|------|---------|
| **is account owner?** | Filter messages from specific user |
| **No Operation, do nothing** | Placeholder |

---

## 📝 Setup Checklist

- [ ] Create bot with @BotFather
- [ ] Enable Secretary mode
- [ ] Configure Telegram Business → Chatbots
- [ ] Copy webhook path
- [ ] Set webhook URL
- [ ] Import all sub-workflows
- [ ] Update workflow IDs in Execute Workflow nodes
- [ ] Replace bot token
- [ ] Activate main workflow
- [ ] Test with: `Give me crypto rates`

---

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| Webhook returns 404 | Check path matches n8n config |
| Bot doesn't respond | Verify Secretary mode ON |
| Sub-workflow not found | Update workflow ID in node |
| 403 Forbidden | Check token is correct |

---

## 🔄 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.1 | 2026-06-29 | Added Music, ART, Callback routes |
| 1.0 | 2026-05-31 | Initial release |

---

**Version:** 1.1 | **Last Updated:** 2026-06-29