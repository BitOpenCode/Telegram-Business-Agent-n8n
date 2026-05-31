# Messages - AI Secretary Workflow

## 📋 What This Workflow Does

The **Messages** workflow is the core AI secretary that handles all general conversations, call bookings, greetings, and message forwarding. It uses OpenRouter AI (Hermes 405B) to understand user intent and respond appropriately.

**This workflow is triggered by the main Secretary Agent workflow when no specific command matches.**

---

## 🏗️ Workflow Architecture

```
User Message → Parse Intent → Route by Intent → AI Response → Send to User
```

### Intent Routing:

| Intent | Handler |
|--------|---------|
| `callBooking` | Send Calendly button |
| `question` | AI Agent (with memory) |
| `greeting` | Greeting Agent |
| Default | AI Agent |

---

## 📁 File Information

| Property | Value |
|----------|-------|
| **File Name** | `Messages.json` |
| **Workflow Name** | Messages |
| **Workflow ID** | `w6aQHR9Rmb6UdlrR` |
| **Active Status** | `false` (triggered by main workflow) |
| **Tags** | `reddit` |

---

## 🔧 Main Nodes

### 1. Parse text command2 (Code Node)
Parses incoming message and detects intent using regex patterns:

**Supported intents:**
- `callBooking` - "call", "book", "schedule", "appointment"
- `greeting` - "hi", "hello", "good morning"
- `question` - "how", "why", "what", "when"
- `reminder`, `task`, `schedule`, `report`, `help`
- `negative`, `positive`, `farewell`, `identity`

**Output includes:**
- Original message data
- Intent with confidence score
- Call context (date, time, topic, urgency)
- Calendly link (if booking)

---

### 2. Switch Node
Routes messages based on `intent.primary`:

| Route | Destination |
|-------|-------------|
| `callBooking` | Send Telegram message with Calendly button |
| `question` | AI Agent (with memory) |
| `greeting` | Greeting Agent |
| Default | AI Agent |

---

## 🤖 AI Agents

### Agent 1: Main Secretary (AI Agent1)

**Model:** `nousresearch/hermes-3-llama-3.1-405b:free` via OpenRouter

**Memory:** Buffer window (5 messages) per `business_connection_id`

**System Prompt:** Professional secretary for @LilBitcoinVert

**Capabilities:**
- Answer questions about the creator
- Forward messages
- Handle urgent requests
- Provide help and commands
- Handle complaints/feedback

**Response Rules:**
- Max 3-4 sentences
- No markdown/HTML
- Same language as user
- Always offer solution (type "call")

---

### Agent 2: Greeting Agent (AI Agent2)

**Model:** `nousresearch/hermes-3-llama-3.1-405b:free` via OpenRouter

**Memory:** Buffer window per `business_connection_id`

**System Prompt:** Professional receptionist - ONLY handles greetings and farewells

**Capabilities:**
- Welcome users warmly
- Say goodbye
- Redirect complex requests to main secretary

**Does NOT do:**
- Book calls
- Answer questions
- Handle urgent messages
- Provide command lists

---

## 📞 Call Booking Flow

When user expresses intent to book a call:

1. **Switch** routes to `callBooking`
2. **Send Telegram message with button** sends:
   ```
   You can book a convenient time by clicking the button below.
   [📅 Call booking] - Link to Calendly
   ```

3. **Calendly webhook (optional, disabled)** can capture booking events

---

## 🗄️ Calendly Integration (Optional)

### Setup Steps:

1. **Create Calendly App:**
   - Go to https://developer.calendly.com/
   - Register → My Apps → + Create new app
   - Name: `Secretary`
   - Redirect URI: `https://n8n.com/rest/oauth2-credential/callback`

2. **Get Credentials:**
   - Client ID
   - Client Secret
   - Webhook signing key

3. **Configure in n8n:**
   - Create Calendly OAuth2 credential
   - Enter Client ID and Secret
   - Connect account

4. **Create Event Type:**
   - Go to Calendly → Event Types
   - Create "One on One" meeting
   - Name: `Secretary`
   - Location: Google Meet
   - Copy invite link: `https://calendly.com/nameyour/secretary`

5. **Enable Webhooks (optional):**
   - Enable `Calendly Trigger` nodes
   - They will store bookings in PostgreSQL

---

## 🗄️ PostgreSQL Database (Optional)

### Table Schema for Call Tracking:

```sql
CREATE TABLE "calls" (
    id SERIAL PRIMARY KEY,
    user_name VARCHAR(200),
    user_email VARCHAR(200) NOT NULL,
    chat_id BIGINT,
    call_type VARCHAR(100),
    call_date DATE,
    call_time TIME,
    call_start TIMESTAMP,
    call_end TIMESTAMP,
    timezone VARCHAR(50),
    join_url TEXT,
    cancel_url TEXT,
    reschedule_url TEXT,
    calendly_event_uri TEXT,
    calendly_invitee_uri TEXT,
    status VARCHAR(50) DEFAULT 'active',
    raw_payload JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Triggers (Disabled by Default):

| Node | Purpose |
|------|---------|
| `Calendly Trigger1` | Captures new bookings |
| `Calendly Cancel` | Captures cancelled bookings |
| `insert call info` | Saves to database |
| `calls Create table` | Creates table schema |

---

## 🌐 OpenRouter Setup

### Step 1: Create Account
1. Go to https://openrouter.ai/
2. Register/Login
3. Go to Workspaces → Default → Keys

### Step 2: Create API Key
1. Click **+ New Key**
2. Name: `Router` (or anything)
3. Credit limit: Optional
4. Expiration: No Expiration
5. Copy the API key

### Step 3: Configure in n8n
1. Open any AI Agent node
2. Create new credential → OpenRouter API
3. Paste your API key
4. Save

**Model used:** `nousresearch/hermes-3-llama-3.1-405b:free`

---

## 🔄 Two AI Versions

This workflow contains **two different AI setups**:

| Path | Nodes | Purpose |
|------|-------|---------|
| **Old Path** | Parse text command → Convert for AI → Basic LLM Chain | Simple responses (no memory) |
| **New Path** | Parse text command3 → AI Agent → Simple Memory | Full secretary with memory |

**Note:** The workflow currently uses the **new path** with `Parse text command3` feeding into `AI Agent`.

---

## 📊 Node Reference

| Node | Type | Position | Purpose |
|------|------|----------|---------|
| **When Executed by Another Workflow** | Trigger | (-1008, 3600) | Receives from main workflow |
| **Parse text command2** | Code | (-704, 3600) | Intent detection (English patterns) |
| **Switch** | Switch | (-496, 3584) | Routes by intent |
| **Send Telegram message with button** | HTTP | (-112, 3264) | Sends Calendly button |
| **AI Agent1** | Agent | (-128, 3600) | Main secretary (questions) |
| **AI Agent2** | Agent | (-128, 3952) | Greeting agent |
| **Code1** | Code | (224, 3600) | Checks for "/calls" in response |
| **Is it about calls schedule?** | IF | (432, 3600) | Routes "/calls" to button |
| **Code** | Code | (704, 3504) | Builds call management keyboard |
| **Send Telegram Message** | HTTP | (896, 3504) | Sends call management UI |
| **Parse text command3** | Code | (-80, 2160) | Alternative intent parser |
| **AI Agent** | Agent | (96, 2160) | Fallback AI (old path) |

---

## 🎯 Example Conversations

### Greeting:
```
User: "Hi"
Agent: "Hello! I'm PocketAI, the receptionist for @LilBitcoinVert. Our main secretary will help you with call bookings and messages. What would you like to do?"
```

### Question:
```
User: "What does @LilBitcoinVert do?"
Agent: "@LilBitcoinVert is currently busy working on projects. I'll pass along your question. If urgent — type urgent."
```

### Call Booking:
```
User: "I want to book a call"
Agent: [Sends Calendly button]
```

### Urgent:
```
User: "urgent, the bot is not working"
Agent: "I've noticed the technical issue! I've immediately notified @LilBitcoinVert. He'll fix it shortly. Thanks for the report!"
```

### Farewell:
```
User: "bye"
Agent: "Goodbye! If you need to book a call or reach @LilBitcoinVert later, just say 'call' or 'help'. Take care!"
```

---

## 🛠️ Setup Instructions

### Step 1: OpenRouter Account
1. Create account at https://openrouter.ai/
2. Generate API key
3. Create credential in n8n

### Step 2: Update Bot Token
Replace `8800811205:AAEKLbn5yJkIliXDEpJteRrCbngHgTwftf0` in all HTTP nodes with your token

### Step 3: Import Workflow
1. Copy JSON content
2. In n8n: Workflows → Import from File
3. Save workflow

### Step 4: Connect to Main Workflow
In **Secretary Agent** main workflow, update the **Messages** Execute Workflow node with ID: `w6a123R954b6U66dlrR`

### Step 5: Configure Calendly (Optional)
- Create Calendly OAuth2 credential
- Update Calendly link in button node
- Enable webhook nodes if using PostgreSQL

### Step 6: Activate
- Keep this workflow **inactive** (triggered by main workflow)
- Main workflow must be active

---

## 🔑 Intent Patterns (English)

| Intent | Regex Pattern |
|--------|---------------|
| callBooking | `/(book\|schedule\|call\|consultation\|meet\|call me\|let's talk\|appointment\|reserve\|set up a call\|booking)/i` |
| greeting | `/^(hi\|hello\|hey\|good morning\|good afternoon\|good evening\|howdy\|sup\|greetings\|yo)/i` |
| question | `/[?]\|\b(how\|why\|where\|when\|who\|what\|which\|how many\|tell me\|explain)\b/i` |
| farewell | `/(bye\|goodbye\|see you\|farewell\|take care)/i` |

---

## 🧪 Testing

### Test Greeting:
```
User: "Hello"
Expected: Warm greeting + redirect
```

### Test Question:
```
User: "What can you do?"
Expected: List of capabilities
```

### Test Call Booking:
```
User: "I want to book a call"
Expected: Calendly button
```

### Test Urgent:
```
User: "urgent, I need help"
Expected: Notification message
```

---

## ⚠️ Important Notes

1. **Two AI paths exist** - Newer path uses `Parse text command3` → `AI Agent` with memory
2. **Memory per user** - Uses `business_connection_id` as session key
3. **Calendly nodes disabled** - Enable only if using PostgreSQL
4. **Bot token visible** - Consider using n8n credentials
5. **Free AI model** - Hermes 405B free tier (rate limits may apply)

---

## 🔗 Dependencies

| Workflow | Relationship |
|----------|--------------|
| Secretary Agent (Main) | Calls this workflow for general messages |

---

## 🐛 Troubleshooting

| Issue | Solution |
|-------|----------|
| No AI response | Check OpenRouter API key and credits |
| Wrong intent | Review regex patterns in Parse text command2 |
| Memory not working | Verify business_connection_id is passed |
| Calendly not showing | Update URL in button node |
| Bot token error | Replace token in all HTTP nodes |

---

## 📝 Customization

### Change AI Model:
Edit AI Agent nodes → Change model to any OpenRouter model:
- `google/gemini-2.0-flash-exp:free`
- `meta-llama/llama-3.3-70b-instruct:free`
- `microsoft/phi-3.5-mini-128k-instruct:free`

### Add Custom Responses:
Edit system prompt in AI Agent1 node

### Add More Intents:
Update `intentPatterns` in Parse text command2 node

---

**Version:** 1.0 | **Last Updated:** 2026-05-31