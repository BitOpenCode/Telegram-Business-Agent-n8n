# Secretary - Telegram Business Secretary Bot

## 📖 What Is This Project?

**Secretary** is a professional virtual secretary bot for Telegram that helps manage communication, book calls, and provide automated responses. The bot acts as a filter between you and your contacts, saving your time while maintaining professionalism.

### Key Features:

- 🤖 **AI-Powered Secretary** - Uses OpenRouter AI (Hermes 405B) to understand and respond to messages
- 📞 **Call Booking** - Integrates with Calendly for automated meeting scheduling
- 💰 **Crypto & Forex Rates** - Real-time price checks from Binance and TradingView
- 🚨 **Urgent Message Handling** - Priority routing for important communications
- 🧠 **Conversation Memory** - Remembers context per user
- 🌍 **Multi-language** - Responds in the user's language

---

## 🎯 What Problem Does It Solve?

| Problem | Solution |
|---------|----------|
| Too many messages | Bot filters and handles common questions |
| Scheduling calls | Automated Calendly booking |
| Checking crypto prices | Instant "Rate BTC/USDT" command |
| Urgent matters | Priority notification system |
| Time zone differences | Bot works 24/7 |

---

## 📁 Project Structure

```
Secretary/
├── Secretary Agent.json          # MAIN WORKFLOW (always active)
├── Messages.json                 # AI secretary & call booking
├── Rate with TradingView.json    # Crypto & forex rates
├── Crypto rates.json             # Top 10 crypto dashboard
├── Currency rates.json           # Forex dashboard (10 pairs)
└── README.md                     # This file
```

---

## 🚀 Quick Start

### Prerequisites

| Requirement | Notes |
|-------------|-------|
| n8n instance | Self-hosted or cloud (n8n.cloud, n8n.io) |
| Telegram account | Free or Premium |
| Telegram Bot | Create via @BotFather |
| Telegram Business | **Premium required** for bot to read personal messages |
| OpenRouter account | Free tier available |
| Calendly account | Free tier (optional) |


Main n8n https://n8n.io/ also okay

---

## 📱 Telegram Business Setup (Important!)

> ⚠️ **Note:** The bot can only read your personal messages if you have **Telegram Premium**. This feature is called "Telegram Business" and is currently available only for Premium subscribers.

### Step-by-Step:

1. **Create Bot with @BotFather**
   - Send `/newbot` → Choose name → Get API token

2. **Enable Secretary Mode**
   - @BotFather → Your bot → Bot Settings → Secretary Mode → Turn ON

3. **Configure Telegram Business**
   - Open Telegram App
   - Settings → Telegram Business → Chatbots
   - Click "Chat Automation"
   - Add your bot (username or link)

4. **Set Bot Permissions**
   - Chat The Bot Can Access: **Only Selected Chats**
   - Included Chats: Add your account or a friend
   - Bot Permissions: **Read, Reply, Mark messages as Read**

5. **Start Your Bot**
   - Open `t.me/your_bot_name`
   - Click "Start"

---

## 🛠️ Installation Guide

### Step 1: Import Workflows

1. Open n8n
2. Go to **Workflows** → **Import from File**
3. Import all 5 JSON files in this order:
   - `Secretary Agent.json` (main)
   - `Messages.json`
   - `Rate with TradingView.json`
   - `Crypto rates.json`
   - `Currency rates.json`

### Step 2: Configure OpenRouter (AI)

1. Create account at https://openrouter.ai/
2. Go to **Keys** → **Create Key**
3. Copy API key
4. In n8n, open any AI Agent node
5. Create credential → OpenRouter API → Paste key

### Step 3: Update Bot Token

In all HTTP Request nodes (search for `8800811205:AAEKLbn5...`), replace with your bot token:

```
https://api.telegram.org/botYOUR_TOKEN_HERE/sendMessage
```

### Step 4: Set Webhook

1. Open **Secretary Agent** workflow
2. Find **Webhook** node → Copy the **Path** (e.g., `7fc9fe99-4832-401b-a3aa-4a802089d750`)
3. Execute the **Set Telegram Webhook** node with URL:

```
https://api.telegram.org/botYOUR_TOKEN/setWebhook?url=https://YOUR_N8N_DOMAIN/webhook/YOUR_PATH
```

### Step 5: Activate Main Workflow

- Toggle **Secretary Agent** workflow to **Active** (green)

---

## 📞 Calendly Integration (Optional)

### What You Get:
- Users can book calls with you via a button
- Automatic Google Meet links
- Calendar sync

### Setup:

1. Go to https://developer.calendly.com/
2. Register → My Apps → **+ Create new app**
3. Name: `Secretary`
4. Redirect URI: `https://n8n.com/rest/oauth2-credential/callback`
5. Copy **Client ID** and **Client Secret**
6. In n8n: Create Calendly OAuth2 credential
7. Create Event Type in Calendly: `Secretary`
8. Copy invite link: `https://calendly.com/yourname/secretary`
9. Update the URL in **Send Telegram message with button** node

---

## 🗄️ PostgreSQL Database (Optional)

> **Note:** PostgreSQL is NOT required for the bot to work. The database is only for tracking call history and is completely optional. The bot works perfectly without it.

### If You Want Call History:

**Table Schema:**
```sql
CREATE TABLE "calls" (
    id SERIAL PRIMARY KEY,
    user_name VARCHAR(200),
    user_email VARCHAR(200) NOT NULL,
    call_type VARCHAR(100),
    call_date DATE,
    call_time TIME,
    status VARCHAR(50) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Enable nodes:**
- `Calendly Trigger1` (invitee.created)
- `Calendly Cancel` (invitee.canceled)
- `insert call info`
- `calls Create table`

> ⚠️ **All database nodes are DISABLED by default.** Enable only if you have PostgreSQL configured.

---

## 🎮 Commands

| Command | What It Does |
|---------|---------------|
| `Crypto rates` | Shows top 10 crypto prices |
| `Currency rates` | Shows 10 forex pairs |
| `Rate BTC/USDT` | Specific crypto price + chart |
| `Rate USDRUB` | Specific forex rate + chart |
| `call` (or any booking intent) | Sends Calendly button |
| `urgent` | Notifies creator immediately |
| `help` | Shows available commands |
| `/calls` | Call management (coming soon) |

---

## 📊 Workflow Overview

### 1. Secretary Agent (Main)
- Entry point for all messages
- Routes to sub-workflows based on command
- **Must always be active**

### 2. Messages (AI Secretary)
- Handles general conversations
- AI-powered responses with memory
- Call booking flow

### 3. Rate with TradingView
- Parses "Rate BTC/USDT" commands
- Fetches from Binance (crypto) or TradingView (forex)

### 4. Crypto rates
- Returns top 10 crypto dashboard
- Fetches 20+ coins from Binance

### 5. Currency rates
- Returns 10 forex pairs dashboard
- USD, EUR, RUB, GBP, JPY, CNY, AED, THB, CHF, AUD

---

## 🔑 Environment Variables

| Variable | Where to Change |
|----------|----------------|
| Bot Token | All HTTP Request nodes |
| Webhook Path | Set Telegram Webhook node |
| Calendly URL | Send Telegram message with button node |
| OpenRouter API Key | AI Agent credentials |

---

## 🧪 Testing Your Bot

After setup, test these commands in Telegram:

```
1. "Hello" → Should get greeting
2. "Crypto rates" → Shows BTC, ETH, etc.
3. "Currency rates" → Shows USD/RUB, EUR/USD, etc.
4. "Rate BTC/USDT" → Shows Bitcoin price
5. "I want to book a call" → Shows Calendly button
6. "urgent, help me" → Notifies creator
```

---

## ❌ Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Bot doesn't respond to personal messages | You need **Telegram Premium** for Business API |
| Webhook returns 404 | Check path matches n8n webhook URL |
| "chat_id is empty" | Bot not added as Business chatbot |
| AI doesn't answer | Check OpenRouter API key |
| Crypto rates not working | Binance API is public, no key needed |
| Calendly button not showing | Update URL in the node |

---

## 📈 What Works Without PostgreSQL?

| Feature | Requires PostgreSQL? |
|---------|---------------------|
| AI responses | ❌ No |
| Crypto/forex rates | ❌ No |
| Call booking button | ❌ No |
| Urgent messages | ❌ No |
| Call history tracking | ✅ Yes (optional) |

**The bot works 100% without a database.** PostgreSQL is only for tracking who booked calls when.

---

## 🔐 Security Notes

1. **Bot Token** - Keep private. Delete from examples before sharing.
2. **Webhook Path** - Keep secret. Anyone with it can send messages to your bot.
3. **OpenRouter Key** - Store in n8n credentials, not in nodes.
4. **Calendly OAuth** - Use n8n credentials, not hardcoded.

---

## 📝 File Summary

| File | Purpose | Active |
|------|---------|--------|
| `Secretary Agent.json` | Main router | ✅ Always ON |
| `Messages.json` | AI secretary | ❌ Triggered |
| `Rate with TradingView.json` | Price lookup | ❌ Triggered |
| `Crypto rates.json` | Crypto dashboard | ❌ Triggered |
| `Currency rates.json` | Forex dashboard | ❌ Triggered |

---

## 🎓 Understanding the Flow

```
Telegram User
     │
     ▼
Telegram Business API
     │
     ▼
Secretary Agent (Main Workflow) ─────► Always Active
     │
     ├── "Crypto rates" ─────────────► Crypto rates workflow
     ├── "Currency rates" ───────────► Currency rates workflow
     ├── "Rate BTC/USDT" ────────────► Rate with TradingView workflow
     └── any other message ──────────► Messages workflow (AI secretary)
```

---

## 📞 Support

For issues:
1. Check that **Telegram Premium** is active (for Business API)
2. Verify webhook is set correctly
3. Check n8n execution logs
4. Ensure main workflow is **active** (green)

---

## 🚧 Limitations

| Limitation | Workaround |
|------------|------------|
| Telegram Business requires Premium | Use group chats instead |
| Free AI models have rate limits | Upgrade to paid model |
| No built-in database | Add PostgreSQL optionally |
| Single bot token | Create multiple bots for different purposes |

---

## 🔮 Future Improvements

- [ ] Multi-language support (already partially implemented)
- [ ] Voice message transcription
- [ ] File handling
- [ ] Group chat support
- [ ] Custom AI training with user data
- [ ] WhatsApp integration
- [ ] Music downloads

---

## 📄 License

This project is for educational purposes. Modify and use as needed.

---

## 🙏 Credits

- **n8n** - Workflow automation
- **OpenRouter** - AI models
- **Binance** - Crypto price API
- **TradingView** - Forex price API
- **Calendly** - Scheduling API
- **Telegram** - Messaging platform

---

**Made with ❤️ for @LilBitcoinVert**

---

*Last Updated: May 31, 2026*
