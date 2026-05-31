# Telegram Secretary Mode - What You Need to Know

## 📢 Announcement (May 20, 2026)

Telegram released **Secretary Mode** - a feature allowing bot developers to connect AI agents to personal accounts, read incoming messages, and reply on behalf of the owner in selected chats.


---

## 🔧 How It Works

### Two independent activation steps:

**For Developer (BotFather):**
- Enable **Secretary Mode** in bot settings
- Required for BusinessConnection updates

**For User (Telegram Premium required):**
- Settings → Telegram Business → Chatbots → Chat Automation
- Add bot username
- Choose which chats bot can access
- Grant permissions (read, reply, media, payments)

### After Authorization:
- Bot receives `business_connection_id`
- Each message triggers `updateBotNewBusinessMessage`
- Bot replies via `messages.sendMessage` with business connection wrapper

> ⚠️ **One bot per account** - to switch bots, disconnect the previous one.

---

## ⏰ 24-Hour Activity Window

**Critical rule:** Bot can only reply in chats with real messages from the last 24 hours.

If user hasn't contacted someone in over 24 hours and bot tries to start a conversation:
→ Error: `BUSINESS_CHAT_INACTIVE`

**Why?** Prevents spam/abuse if someone steals a developer token.

**Impact:** Bot works great for customer replies, but CANNOT start cold outreach.

---

## 📡 API Methods Available (~20 methods)

| Category | Methods |
|----------|---------|
| Messaging | `sendMessage`, `editMessage`, `deleteMessages` |
| Media | `sendMedia`, `uploadMedia` |
| Stories | `sendStory`, `editStory` |
| Payments | `getPaymentForm`, `sendStarsForm` |
| Interactions | Reactions, inline buttons, gifts |

### Bot Permissions (BusinessBotRights):

- `can_read_messages` - read incoming
- `can_reply` - send replies
- `can_delete_outgoing` - delete bot's messages
- `can_view_gifts` - see gifts
- `can_transfer_stars` - send stars

> **Principle of least privilege** - request only what you need.

---

## 🎯 Use Cases

| Use Case | Description |
|----------|-------------|
| **Commercial Auto-Responder** | Answer basic questions (hours, prices, location) before owner responds |
| **Personal AI Secretary** | LLM integration (Claude, GPT, Gemini) - classify urgency, answer trivial questions |
| **Anti-Spam Filter** | Detect phishing/ads, auto-archive or delete |
| **Lead Management** | Collect contacts, classify sources, push to CRM via webhook |
| **Accessibility Assistant** | Voice commands → messages, or incoming → synthesized audio |

---

## 🔐 Privacy Implications

**Critical change:** When you authorize a secretary bot, you accept that a third party (bot developer, LLM provider) gains access to chat content.

- Secret Chats (E2EE) remain protected
- Regular chats lose server-side confidentiality promise

### For GDPR (European users):

Bot developer becomes a **data controller** and must:
- Publish privacy policy
- Obtain explicit consent
- Handle access, correction, and deletion requests

### Recommendations for users:

1. Read bot's privacy policy before authorizing
2. Limit access to necessary chats only
3. Exclude private chats with family/partners
4. Revoke access when bot is not needed

---

## 🛠️ Library Support

| Language | Library | Status |
|----------|---------|--------|
| Python | `python-telegram-bot` v21.5+ | ✅ Full support |
| JavaScript/Node | `grammY` | ✅ Full support |
| .NET | `Telegram.Bot` | 🔄 In progress |
| Go | `telebot` | 🔄 In progress |

---

## 📖 Official Resources

- [Telegram Blog: AI Bot Revolution](https://telegram.org/blog)
- [Core API: Secretary Bots](https://core.telegram.org/bots/features#secretary-bots)
- [Business Connection API](https://core.telegram.org/bots/api#businessconnection)

---

**Bottom line:** Telegram turned every personal account into a potential frontend for automated agents. Great for customer service and personal assistants. Not for cold outreach.

---

*Documentation for Secretary - Telegram Business Secretary Bot*

---
