# Secretary Agent - Main Workflow

## 📋 Overview

The **Secretary Agent** is the main entry point workflow for your Telegram Business Bot. It receives all incoming messages from Telegram, routes them to appropriate sub-workflows based on command type, and handles webhook responses.

**This workflow must always be active!**

---

## 🏗️ Workflow Architecture

```
Telegram Business API → Webhook → Route Commands (Switch) → Sub-workflows → 200 OK Response
```

---

## 📁 File Information

| Property | Value |
|----------|-------|
| **File Name** | `Secretary Agent.json` |
| **Workflow ID** | `EG10MYoF0z129f21k` |
| **Active Status** | `true` (always on) |
| **Tags** | `reddit` |

---

## 🔧 Prerequisites

Before importing this workflow, make sure you have:

1. ✅ **n8n instance** (self-hosted or cloud)
2. ✅ **Telegram Bot** created via @BotFather
3. ✅ **Telegram Business** account
4. ✅ **Sub-workflows** ready (imported in n8n)

---

## 🚀 Quick Start Guide

### Step 1: Create Webhook in n8n

1. Open the **Webhook** node in this workflow
2. Copy the **Path** value (example: `7fc9fe99-4832-401b-a3aa-4a802089d750`)
3. You'll need this for the Telegram webhook URL

> ⚠️ **Security**: Keep your webhook path private

---

### Step 2: Configure Telegram Bot

#### BotFather Setup:

1. Create bot with [@BotFather](https://t.me/botfather)
2. Pick your bot → **Bot Settings**
3. Choose **Secretary mode** → Turn **ON**
4. Send /start to your bot in private message (with bot)

#### Telegram Business Setup:

1. Open **Telegram Settings**
2. Tap **Telegram Business**
3. Select **Chatbots** → **Chat Automation**

More on Sticky Notes (full guide)

#### Add Your Bot:

| Field | Value |
|-------|-------|
| **Bot Username/Link** | `t.me/your_bot_name` |
| **Chat The Bot Can Access** | Only Selected Chats |
| **Included Chats** | Your friend or another account |

#### Bot Permissions:

For testing, choose:
- ✅ **Read**
- ✅ **Reply**
- ✅ **Mark messages as Read**

> **Note**: Don't touch Manage Profile, Manage Gifts and Stars, or Manage Stories

---

### Step 3: Set Telegram Webhook

Execute the **Set Telegram Webhook** node with this URL format:

```
https://api.telegram.org/botYOUR_BOT_TOKEN/setWebhook?url=https://n8n.YOUR_DOMAIN.com/webhook/YOUR_WEBHOOK_PATH
```

#### Example:

```
https://api.telegram.org/bot8800811205:AAEKLbn5yJkIliXDEpJteRrCbngHgTwftf0/setWebhook?url=https://n8n.bitcoinlimb.com/webhook/7fc9fe99-4832-401b-a3aa-4a802089d750
```

#### Expected Response:

```json
[
  {
    "ok": true,
    "result": true,
    "description": "Webhook was set"
  }
]
```

---

### Step 4: Activate Workflow

1. Toggle the workflow to **Active** (green)
2. The workflow will now listen for incoming messages

---

## 🔀 Route Commands (Switch Node)

The **Route Commands** node routes messages to different sub-workflows:

| Condition | Route To | Output Key |
|-----------|----------|------------|
| `text == "Crypto rates"` | Crypto rates workflow | `Crypto rates` |
| `text == "Currency rates"` | Currency rates workflow | `Currency rates` |
| `text starts with "Rate"` | Rate workflow (TradingView) | `Rate` |
| `text exists` (default) | Messages workflow | `message` |

---

## 🔗 Sub-workflows

This workflow executes 4 sub-workflows:

| Node Name | Workflow ID | Purpose |
|-----------|-------------|---------|
| **Messages** | `w6aQHR9Rmb6UdlrR` | AI secretary, call booking, urgent messages |
| **Crypto rates** | `vJkb8BRltug2ThRv` | Fetch cryptocurrency prices from Binance |
| **Currency rates** | `uOM6FAQkWuXEkEVD` | Fetch forex rates from TradingView |
| **Rate** | `6U0KACxsJymQRoiU` | Generic rate lookup with TradingView charts |

> **Important**: These workflow IDs must match the IDs in your n8n instance. Update them after importing.

---

## 📊 Node Reference

### Core Nodes

| Node | Type | Position | Description |
|------|------|----------|-------------|
| **Webhook** | Webhook | (-928, 704) | Receives POST requests from Telegram |
| **Route Commands** | Switch | (-480, 672) | Routes messages to appropriate sub-workflows |
| **Respond to Webhook** | Respond to Webhook | (384, 720) | Returns 200 OK to Telegram |

### Sub-workflow Executors

| Node | Position | Connected From |
|------|----------|----------------|
| **Crypto rates** | (-80, 400) | Route Commands [0] |
| **Currency rates** | (-80, 560) | Route Commands [1] |
| **Rate** | (-64, 720) | Route Commands [2] |
| **Messages** | (-64, 928) | Route Commands [3] |

### Optional Nodes (Disabled)

| Node | Purpose | Status |
|------|---------|--------|
| **is account owner?** | Filter messages from specific Telegram user | Disabled |
| **No Operation, do nothing** | Placeholder for owner filter | Disabled |

---

## 🎯 Sticky Notes (Documentation)

The workflow includes visual documentation notes:

| Note | Position | Content |
|------|----------|---------|
| Sticky Note1 | (-1472, 576) | Bot setup instructions |
| Sticky Note2 | (-1232, 576) | Webhook creation guide |
| Sticky Note | (-1232, 864) | Telegram webhook URL format |
| Sticky Note3 | (-816, 1072) | Expected webhook response |
| Sticky Note4 | (-1232, 1456) | Activate workflow reminder |
| Sticky Note5 | (-208, 848) | Messages workflow info |
| Sticky Note6 | (-784, 64) | Secretary Agent trigger note |
| Sticky Note7 | (-1232, 1552) | Telegram setup details |
| Sticky Note8 | (-208, 240) | Crypto rates workflow info |
| Sticky Note9 | (-960, 576) | POST response note |

---

## 🔐 Security Notes

1. **Webhook Path**: Keep your webhook path secret
2. **Bot Token**: Replace `8800811205:AAEKLbn5yJkIliXDEpJteRrCbngHgTwftf0` with your own token
3. **Domain**: Update `n8n.bitcoinlimb.com` with your n8n domain
4. **Owner Filter**: Enable the disabled IF node to restrict access to your Telegram ID only

---

## ⚠️ Before Importing

Replace these values in the JSON file:

```javascript
// Replace with your values:
YOUR_BOT_TOKEN        // From @BotFather
YOUR_N8N_DOMAIN       // Your n8n instance URL
YOUR_WEBHOOK_PATH     // From Webhook node
WORKFLOW_IDS          // Update sub-workflow IDs to match your instance
```

---

## 📝 Setup Checklist

- [ ] Create bot with @BotFather
- [ ] Enable Secretary mode in BotFather
- [ ] Configure Telegram Business → Chatbots
- [ ] Add bot with correct permissions
- [ ] Copy webhook path from n8n
- [ ] Set Telegram webhook URL
- [ ] Import all 4 sub-workflows
- [ ] Update workflow IDs in Execute Workflow nodes
- [ ] Replace bot token with your own
- [ ] Activate main workflow
- [ ] Test by sending "Crypto rates" to your bot

---

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| Webhook returns 404 | Check webhook path matches n8n configuration |
| Bot doesn't respond | Verify Secretary mode is ON in BotFather |
| "chat_id is empty" error | Check that message contains business_connection_id |
| Sub-workflow not found | Update workflow ID in Execute Workflow node |
| 403 Forbidden | Check bot token is correct and bot is started |

---

## 📂 Related Files

| File | Description |
|------|-------------|
| `Messages.json` | AI secretary workflow |
| `Crypto rates.json` | Binance price fetcher |
| `Currency rates.json` | Forex rate fetcher |
| `Rate TradingView.json` | TradingView chart generator |

---

## 🔄 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-05-31 | Initial release |

---

**Remember**: This workflow must remain **active** for your bot to function. Sub-workflows can be updated without restarting the main workflow.
