# Secretary - Telegram Business Secretary Bot

## 📖 What Is This Project?

**Secretary** is a professional virtual secretary bot for Telegram that helps manage communication, book calls, and provide automated responses. The bot acts as a filter between you and your contacts, saving your time while maintaining professionalism.

**Version:** 1.1

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| 🤖 **AI-Powered Secretary** | Uses OpenRouter AI (Hermes 405B) to understand and respond to messages |
| 📞 **Call Booking** | Integrates with Calendly for automated meeting scheduling |
| 💰 **Crypto & Forex Rates** | Real-time price checks from Binance and TradingView |
| 🎵 **Music Search** | Find and download tracks from Deezer |
| 🎨 **ART Generator** | Create 3D T-shirt mockups from photos |
| 🚨 **Urgent Message Handling** | Priority routing for important communications |
| 🧠 **Conversation Memory** | Remembers context per user |
| 🌍 **Multi-language** | Responds in the user's language |

---

## 🎯 What Problem Does It Solve?

| Problem | Solution |
|---------|----------|
| Too many messages | Bot filters and handles common questions |
| Scheduling calls | Automated Calendly booking |
| Checking crypto prices | Instant "Дай курсы криптовалют" command |
| Finding music | "Music {artist}" command |
| Creating merch | Send photo → get 3D T-shirt mockup |
| Urgent matters | Priority notification system |
| Time zone differences | Bot works 24/7 |

---

## 📁 Project Structure

Secretary/
├── Secretary Agent.json              # MAIN WORKFLOW (always active)
├── Message PocketAI.json             # AI secretary & call booking
├── TOP 10 Crypto.json                # Crypto dashboard
├── TOP 10 Currency Rates.json        # Forex dashboard (10 pairs)
├── Rate with Chart.json              # Specific rate + TradingView chart
├── PocketLine Music Search.json      # Deezer music search
├── PocketAI More Music.json          # Load more music results
├── Download track PocketAI.json      # Download audio from Deezer
├── ART create PocketAI.json          # 3D T-shirt mockup generator
├── Callback PocketAI.json            # Callback router (12 routes)
├── Message /calls button.json        # /calls button handler
├── Email sent by user.json           # Email input handler
└── README.md                         # This file

---

## 🚀 Quick Start

### Prerequisites

| Requirement | Notes |
|-------------|-------|
| n8n instance | Self-hosted or cloud (n8n.cloud, n8n.io) |
| Telegram account | Free or Premium |
| Telegram Bot | Create via @BotFather |
| Telegram Business | Premium required for bot to read personal messages |
| OpenRouter account | Free tier available |
| Calendly account | Free tier (optional) |
| ImgBB account | Free (for ART generator) |
| KIE AI account | Free (for ART generator) |

> ⚠️ **Note:** The bot can only read your personal messages if you have **Telegram Premium**. This feature is called "Telegram Business" and is currently available only for Premium subscribers.

---

## 📱 Telegram Business Setup

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
   - **Chat The Bot Can Access:** Only Selected Chats
   - **Included Chats:** Add your account or a friend
   - **Bot Permissions:** Read, Reply, Mark messages as Read

5. **Start Your Bot**
   - Open t.me/your_bot_name
   - Click "Start"

---

## 🛠️ Installation Guide

### Step 1: Import Workflows

1. Open n8n
2. Go to **Workflows** → **Import from File**
3. Import all JSON files in this order:
   - `Secretary Agent.json` (main)
   - `Message PocketAI.json`
   - `TOP 10 Crypto.json`
   - `TOP 10 Currency Rates.json`
   - `Rate with Chart.json`
   - `PocketLine Music Search.json`
   - `PocketAI More Music.json`
   - `Download track PocketAI.json`
   - `ART create PocketAI.json`
   - `Callback PocketAI.json`
   - `Message /calls button.json`
   - `Email sent by user.json`

### Step 2: Configure OpenRouter (AI)

1. Create account at https://openrouter.ai/
2. Go to **Keys** → **Create Key**
3. Copy API key
4. In n8n, open AI Agent nodes → Create credential → OpenRouter API → Paste key

### Step 3: Update Bot Token

In all HTTP Request nodes, replace `YOUR_BOT_TOKEN` with your actual token:

```
https://api.telegram.org/botYOUR_TOKEN_HERE/sendMessage
```

### Step 4: Set Webhook

1. Open **Secretary Agent** workflow
2. Find **Telegram Webhook** node → Copy the **Path**
3. Execute the **Set Telegram Webhook** node with:

```
https://api.telegram.org/botYOUR_TOKEN/setWebhook?url=https://YOUR_N8N_DOMAIN/webhook/YOUR_PATH
```

### Step 5: Activate Main Workflow

Toggle **Secretary Agent** workflow to **Active** (green).

---

## 🎮 Commands

| Command | What It Does |
|---------|--------------|
| `Дай курсы криптовалют` | Shows top 10 crypto prices |
| `Дай курсы валют` | Shows 10 forex pairs |
| `Курс BTC/USDT` | Specific crypto price + TradingView chart |
| `Курс USD/RUB` | Specific forex rate + chart |
| `Music Travis Scott` | Search and display tracks from Deezer |
| `Музыка Eminem` | Search and display tracks (Russian) |
| `звонок` | Sends Calendly booking button |
| `срочно` | Notifies creator immediately |
| `помощь` | Shows available commands |
| `/calls` | Call management menu |

### Bonus: Send a photo

Send any photo → the bot will generate a 3D T-shirt mockup with your design.

---

## 📊 Workflow Overview

### 1. Secretary Agent (Main)
- Entry point for all messages
- Routes to sub-workflows based on command
- **Must always be active**

### 2. Message PocketAI (AI Secretary)
- Handles general conversations
- AI-powered responses with memory
- Call booking flow
- Urgent message handling

### 3. TOP 10 Crypto
- Returns top 10 crypto dashboard
- Fetches 20+ coins from Binance

### 4. TOP 10 Currency Rates
- Returns 10 forex pairs dashboard
- USD, EUR, RUB, GBP, JPY, CNY, AED, THB, CHF, AUD

### 5. Rate with Chart
- Parses "Курс BTC/USDT" commands
- Fetches from Binance (crypto) or TradingView (forex)
- Returns price + TradingView chart link

### 6. Music Search
- Searches Deezer for tracks
- Returns 10 track buttons
- Supports "More…" pagination

### 7. Download Track
- Downloads audio from Deezer
- Sends MP3 with cover art
- Uses yt-dlp and Song.link API

### 8. ART create
- Generates 3D T-shirt mockups from photos
- Uses OpenRouter (Gemma/GPT-4.1) + KIE AI (Nano Banana 2)
- Uploads to ImgBB for URL processing

### 9. Callback Router
- Routes all callback queries (12 routes)
- Supports dz, dzmore, add_to_playlist, etc.

---

## 🔧 Required API Keys

| Service | What For | Where To Get |
|---------|----------|--------------|
| **OpenRouter** | AI models (Gemma, GPT-4.1) | https://openrouter.ai/ |
| **KIE AI** | Image generation (Nano Banana 2) | https://kie.ai/api-key |
| **ImgBB** | Image upload for ART generator | https://api.imgbb.com/ |
| **Calendly** | Call booking | https://calendly.com/ |
| **Telegram** | Bot token | @BotFather |
| **Deezer** | Music search | Public API (no key needed) |
| **Binance** | Crypto prices | Public API (no key needed) |
| **TradingView** | Forex rates | Public API (no key needed) |

---

## 🧪 Testing Your Bot

After setup, test these commands in Telegram:

| Test | Expected Result |
|------|-----------------|
| `Привет` | AI greeting |
| `Дай курсы криптовалют` | Shows BTC, ETH, etc. |
| `Дай курсы валют` | Shows USD/RUB, EUR/USD, etc. |
| `Курс BTC/USDT` | Shows Bitcoin price + chart link |
| `Music Eminem` | Shows 10 track buttons |
| `звонок` | Shows Calendly button |
| `срочно` | Notifies creator |
| Send a photo | 3D T-shirt mockup generated |

---

## ❌ Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Bot doesn't respond to personal messages | You need Telegram Premium for Business API |
| Webhook returns 404 | Check path matches n8n webhook URL |
| "chat_id is empty" | Bot not added as Business chatbot |
| AI doesn't answer | Check OpenRouter API key |
| Crypto rates not working | Binance API is public, no key needed |
| Music download fails | Install yt-dlp and ffmpeg on server |
| ART generator fails | Check KIE AI API key |
| Calendly button not showing | Update URL in the node |

---

## 📈 What Works Without PostgreSQL?

| Feature | Requires PostgreSQL? |
|---------|---------------------|
| AI responses | ❌ No |
| Crypto/forex rates | ❌ No |
| Music search/download | ❌ No |
| ART generator | ❌ No |
| Call booking button | ❌ No |
| Urgent messages | ❌ No |
| Call history tracking | ✅ Yes (optional) |

> The bot works 100% without a database. PostgreSQL is only for tracking who booked calls when.

---

## 🔐 Security Notes

- **Bot Token** — Keep private. Delete from examples before sharing.
- **Webhook Path** — Keep secret. Anyone with it can send messages to your bot.
- **OpenRouter Key** — Store in n8n credentials, not in nodes.
- **KIE AI Key** — Store in n8n credentials.

---

## 🔮 Future Improvements

- [ ] Voice message transcription
- [ ] File handling
- [ ] Group chat support
- [ ] Custom AI training
- [ ] WhatsApp integration
- [ ] Spotify integration

---

## 📄 License

This project is for educational purposes. Modify and use as needed.

---

## 🙏 Credits

- **n8n** — Workflow automation
- **OpenRouter** — AI models
- **Binance** — Crypto price API
- **TradingView** — Forex price API
- **Deezer** — Music API
- **KIE AI** — Image generation
- **Calendly** — Scheduling API
- **Telegram** — Messaging platform

---

**Made by @LilBitcoinVert**

**Previous versions:**
- **1.0** — 2026-05-31 — Initial release

**Version:** 1.1| **Last Updated:** 2026-06-29
