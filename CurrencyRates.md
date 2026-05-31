# Currency Rates - Forex Rate Dashboard

## 📋 What This Workflow Does

This workflow handles the **"Currency rates"** command in Telegram. When a user types `Currency rates`, it fetches real-time forex rates for 10 major currency pairs from TradingView and returns a formatted dashboard.

**This workflow is triggered by the main Secretary Agent workflow.**

---

## Quick Overview

| User types | What happens |
|------------|--------------|
| `Currency rates` | Returns a list of 10 currency pairs with current rates and 24h changes |

---

## 📊 Currency Pairs Included

| # | Pair | TradingView Symbol |
|---|------|-------------------|
| 1 | EUR/USD | FX_IDC:EURUSD |
| 2 | GBP/USD | FX_IDC:GBPUSD |
| 3 | USD/JPY | FX_IDC:USDJPY |
| 4 | USD/RUB | FX_IDC:USDRUB |
| 5 | USD/CNY | FX_IDC:USDCNY |
| 6 | USD/THB | FX_IDC:USDTHB |
| 7 | USD/AED | FX_IDC:USDAED |
| 8 | CNY/RUB | FX_IDC:CNYRUB |
| 9 | USD/CHF | FX_IDC:USDCHF |
| 10 | AUD/USD | FX_IDC:AUDUSD |

---

## 📤 Example Output

```
💱 *Currency rates (Forex 24h)*

1. *EUR/USD*: 📉 1.0850 (-0.15%)
2. *GBP/USD*: 📈 1.2750 (+0.32%)
3. *USD/JPY*: 📈 148.50 (+0.45%)
4. *USD/RUB*: 📉 91.25 (-0.78%)
5. *USD/CNY*: 📉 7.1850 (-0.08%)
6. *USD/THB*: 📈 35.75 (+0.22%)
7. *USD/AED*: 📉 3.6725 (-0.01%)
8. *CNY/RUB*: 📈 12.70 (+0.55%)
9. *USD/CHF*: 📉 0.8950 (-0.18%)
10. *AUD/USD*: 📈 0.6550 (+0.25%)
```

**Emoji indicators:**
- 📈 = Price increased (positive change)
- 📉 = Price decreased (negative change)

---

## 🏗️ Workflow Architecture

```
User: "Currency rates"
    ↓
Main workflow routes to this sub-workflow
    ↓
Parse Currency Request (extracts chat_id, connection_id)
    ↓
Get Forex Rates (calls TradingView API for 10 pairs)
    ↓
Format Forex Response (builds dashboard message)
    ↓
Send Telegram Message (sends to user)
```

---

## 🔧 Node Reference

| Node | Type | What it does |
|------|------|--------------|
| **When Executed by Another Workflow** | Trigger | Receives input from main workflow |
| **Parse Currency Request** | Code | Extracts chat_id, business_connection_id, message_id |
| **Get Forex Rates** | HTTP Request | Fetches all 10 currency rates from TradingView |
| **Format Forex Response** | Code | Builds formatted dashboard message |
| **Send Telegram Message** | HTTP Request | Sends response to user |

---

## 📡 API Call Details

### TradingView Scanner API

**Endpoint:** `https://scanner.tradingview.com/forex/scan`

**Method:** POST

**Request Body:**
```json
{
  "symbols": {
    "tickers": [
      "FX_IDC:EURUSD",
      "FX_IDC:GBPUSD",
      "FX_IDC:USDJPY",
      "FX_IDC:USDRUB",
      "FX_IDC:USDCNY",
      "FX_IDC:USDTHB",
      "FX_IDC:USDAED",
      "FX_IDC:CNYRUB",
      "FX_IDC:USDCHF",
      "FX_IDC:AUDUSD"
    ]
  },
  "columns": ["close", "change"]
}
```

**Response Format:**
```json
{
  "data": [
    {
      "s": "FX_IDC:EURUSD",
      "d": [1.0850, -0.15]
    }
  ]
}
```
- `d[0]` = Current rate (close)
- `d[1]` = 24h change percentage

---

## 📝 Code Explanation

### Parse Currency Request
Extracts essential data from the incoming webhook:

```javascript
const body = $input.first().json.body;
const message = body.business_message;

return [{
  chat_id: message.chat.id,
  business_connection_id: message.business_connection_id,
  reply_to_message_id: message.message_id
}];
```

### Format Forex Response
1. Maps TradingView symbols to readable names
2. Extracts price and change from API response
3. Calculates trend emoji (📈/📉)
4. Builds formatted message with numbered list

---

## 🎯 How to Use

### In Telegram:
1. Type `Currency rates` in the chat with your bot
2. Bot responds with real-time rates for all 10 currency pairs

### In n8n:
1. Import this workflow
2. Update the bot token in **Send Telegram Message** node
3. Connect to main workflow (Secretary Agent)

---

## 🔄 Adding New Currency Pairs

To add more currency pairs, update two places in the **Format Forex Response** node:

### 1. Add to `currencyPairs` array:
```javascript
const currencyPairs = [
  // ... existing pairs ...
  { symbol: 'FX_IDC:USDCAD', name: 'USD/CAD' },  // Add new pair
];
```

### 2. Add to TradingView request body (Get Forex Rates node):
```json
{
  "symbols": {
    "tickers": [
      // ... existing symbols ...
      "FX_IDC:USDCAD"
    ]
  }
}
```

---

## 🛠️ Setup Instructions

### Step 1: Import Workflow
1. Copy the JSON file content
2. In n8n: **Workflows** → **Import from File**
3. Paste or upload the JSON
4. Save the workflow

### Step 2: Update Bot Token
Replace the token in **Send Telegram Message** node:

```
https://api.telegram.org/botYOUR_BOT_TOKEN_HERE/sendMessage
```

Current token in file: `8800811205:AAEKLbn5yJkIliXDEpJteRrCbngHgTwftf0`

### Step 3: Connect to Main Workflow
In the **Secretary Agent** main workflow, update the **Currency rates** Execute Workflow node with this workflow's ID: `uOM6FAQkWuXEkEVD`

### Step 4: Activate
- This workflow should be **inactive** (triggered by main workflow)
- Keep `active: false`

---

## 🧪 Testing

### Test Command:
Send to your bot: `Currency rates`

### Expected Result:
Dashboard with 10 currency pairs, each showing:
- Current rate (4 decimal places)
- 24h change percentage
- Trend emoji (📈 or 📉)

### Error Handling:
If the API returns no data:
```
❌ Not found.
```

---

## 📊 API Reference

| Parameter | Value |
|-----------|-------|
| **API Provider** | TradingView |
| **Endpoint** | `https://scanner.tradingview.com/forex/scan` |
| **Method** | POST |
| **Content-Type** | application/json |
| **Rate Limit** | None documented |
| **Authentication** | Not required |

---

## 🐛 Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| No response | Bot token incorrect | Update token in HTTP Request node |
| "Not found" | API returned empty data | Check TradingView API status |
| Missing pairs | Symbol not in request | Add symbol to both places |
| Wrong rate format | API response changed | Check TradingView API documentation |
| Workflow not triggered | Not connected to main workflow | Verify workflow ID in main workflow |

---

## 📁 File Information

| Property | Value |
|----------|-------|
| **File Name** | `Currency rates.json` |
| **Workflow Name** | Currency rates |
| **Workflow ID** | `uOM6F12kWuXEk564VD` |
| **Active Status** | `false` (triggered by main workflow) |
| **Tags** | `reddit` |

---

## 🔗 Dependencies

| Workflow | Relationship |
|----------|--------------|
| Secretary Agent (Main) | Calls this workflow when user types "Currency rates" |

---

## ⚡ Performance

- **API Calls:** 1 call per request
- **Data Points:** 10 currency pairs
- **Response Time:** ~1-2 seconds
- **Rate Limit:** None (public API)

---

## 📝 Notes

1. This workflow does NOT run standalone - triggered by main workflow
2. Rates are updated in real-time from TradingView
3. All prices are shown with 4 decimal places
4. Change percentages show with +/- sign
5. No API key required (public endpoint)

---

## 🔐 Security

- No user data stored
- Read-only API calls

---

**Version:** 1.0 | **Last Updated:** 2026-05-31