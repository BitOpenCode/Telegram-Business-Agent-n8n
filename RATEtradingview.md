# Rate with TradingView - Sub-workflow

## 📋 Overview

The **Rate with TradingView** workflow handles currency rate and cryptocurrency price requests. It parses user input (e.g., "Rate BTC/USDT"), determines the pair type (crypto or forex), fetches real-time data from the appropriate API, and returns formatted price information with a TradingView chart link.

**This workflow is triggered by the main Secretary Agent workflow.**

---

## 🏗️ Workflow Architecture

```
Input → Parse Request (Code) → is crypto? (IF) → [Crypto Path] or [Forex Path] → Format Response → Send Telegram Message
```

### Crypto Path:
```
Get Binance Ticker → Format Crypto Response → Send Telegram Message
```

### Forex Path:
```
Get Forex Rates → Format Forex Response → Send Telegram Message
```

---

## 📁 File Information

| Property | Value |
|----------|-------|
| **File Name** | `Rate with TradingView.json` |
| **Workflow ID** | `6U0KA12sJy2345oiU` |
| **Active Status** | `false` (triggered by main workflow) |
| **Tags** | `reddit` |
| **Trigger Type** | Execute Workflow Trigger |

---

## 🚀 How It Works

### Step 1: Parse Request (Code Node)

The first code node extracts and parses the user's message:

**Input format:** `Rate BTC/USDT`, `Rate ETHUSDT`, or `Rate USDRUB`

**Supported patterns:**

| Pattern Type | Example | Explanation |
|--------------|---------|-------------|
| Separated | `BTC/USDT` | Base/Quote with slash |
| Joined | `BTCUSDT` | Base + Quote combined |
| Single | `BTC` | Base currency only (defaults to USDT or USD) |

**Special crypto pairs handled:**
- `ETHUSDT`, `MATICUSDT`, `AVAXUSDT`, `DOTUSDT`, `LINKUSDT`, `TONUSDT`

**Output from Parse Request:**

```javascript
{
  chat_id: number,
  baseCurrency: string,        // e.g., "BTC"
  quoteCurrency: string,       // e.g., "USDT"
  displayPair: string,         // e.g., "BTC/USDT"
  exchangeType: "crypto" | "forex",
  tradingViewSymbol: string,   // e.g., "BINANCE:BTCUSDT" or "FX_IDC:USDRUB"
  binanceSymbol: string | null, // e.g., "BTCUSDT" (crypto only)
  chartLink: string,           // TradingView chart URL
  business_connection_id: string,
  reply_to_message_id: number
}
```

---

### Step 2: is crypto? (IF Node)

Routes the request based on `exchangeType`:

| Condition | Route |
|-----------|-------|
| `exchangeType == "crypto"` | → Get Binance Ticker |
| `exchangeType != "crypto"` (forex) | → Get Forex Rates |

---

## 🔷 Crypto Path (Binance)

### Get Binance Ticker

**API Endpoint:**
```
https://api.binance.com/api/v3/ticker/24hr?symbols=["{{ $json.binanceSymbol }}"]
```

**Example:** `https://api.binance.com/api/v3/ticker/24hr?symbols=["BTCUSDT"]`

**Response includes:**
- `lastPrice` - Current price
- `priceChangePercent` - 24h change percentage
- `highPrice` - 24h high
- `lowPrice` - 24h low

### Format Crypto Response (Code Node)

Formats the Binance data into a Telegram message:

```javascript
💰 *BTC/USDT*

📈 *Price:* 67500.00 USDT
📊 *24h change:* +2.35%
📈 *24h high:* 68000.00
📉 *24h low:* 67000.00

📈 [Open chart on TradingView](https://www.tradingview.com/chart/?symbol=BINANCE:BTCUSDT)
```

---

## 🔷 Forex Path (TradingView)

### Get Forex Rates

**API Endpoint:**
```
https://scanner.tradingview.com/forex/scan
```

**Request Body:**
```json
{
  "symbols": {
    "tickers": ["{{ $json.tradingViewSymbol }}"]
  },
  "columns": ["close", "change"]
}
```

**Example ticker:** `FX_IDC:USDRUB`

**Response format:**
```json
{
  "totalCount": 1,
  "data": [
    {
      "s": "FX_IDC:USDRUB",
      "d": [71.035, -0.07736671824448842]
    }
  ]
}
```
- `d[0]` = Current rate (close)
- `d[1]` = Change value

### Format Forex Response (Code Node)

Formats the TradingView data into a Telegram message:

```javascript
💱 *USD/RUB*

📉 *Rate:* 71.0350
📊 *Change:* -0.08%

📈 [Open chart on TradingView](https://www.tradingview.com/chart/?symbol=FX_IDC:USDRUB)
```

---

### Send Telegram Message

Both paths end with an HTTP Request node that sends the formatted message to Telegram:

**Endpoint:** `https://api.telegram.org/bot<TOKEN>/sendMessage`

**Method:** POST

**Body:** Formatted message with Markdown parse mode

---

## 📊 Node Reference

| Node | Type | Position | Description |
|------|------|----------|-------------|
| **When Executed by Another Workflow** | Trigger | (-704, 144) | Receives input from main workflow |
| **Parse Request** | Code | (-512, 144) | Parses user input, detects pair type |
| **is crypto?** | IF | (-352, 144) | Routes to crypto or forex path |
| **Get Binance Ticker** | HTTP Request | (-128, 16) | Fetches crypto data from Binance |
| **Format Crypto Response** | Code | (80, 16) | Formats crypto data for Telegram |
| **Send Telegram Message** | HTTP Request | (320, 16) | Sends crypto response to user |
| **Get Forex Rates** | HTTP Request | (-128, 208) | Fetches forex data from TradingView |
| **Format Forex Response** | Code | (80, 208) | Formats forex data for Telegram |
| **Send Telegram Message1** | HTTP Request | (320, 208) | Sends forex response to user |

---

## 📝 Supported Currency Pairs

### Cryptocurrencies (Crypto List)

| Symbol | Name |
|--------|------|
| BTC | Bitcoin |
| ETH | Ethereum |
| BNB | Binance Coin |
| SOL | Solana |
| XRP | Ripple |
| ADA | Cardano |
| DOGE | Dogecoin |
| TON | Toncoin |
| LINK | Chainlink |
| TRX | TRON |
| MATIC | Polygon |
| DOT | Polkadot |
| AVAX | Avalanche |

### Fiat Currencies (Forex List)

| Symbol | Name |
|--------|------|
| USD | US Dollar |
| RUB | Russian Ruble |
| EUR | Euro |
| GBP | British Pound |
| JPY | Japanese Yen |
| CNY | Chinese Yuan |
| CHF | Swiss Franc |
| AED | UAE Dirham |
| THB | Thai Baht |
| TRY | Turkish Lira |

---

## 💬 Example User Inputs

| User Types | Detected As | Fetches From |
|------------|-------------|--------------|
| `Rate BTC/USDT` | Crypto | Binance |
| `Rate ETHUSDT` | Crypto | Binance |
| `Rate BTC` | Crypto (default USDT) | Binance |
| `Rate USDRUB` | Forex | TradingView |
| `Rate EURUSD` | Forex | TradingView |
| `Rate GBPJPY` | Forex | TradingView |

---

## 📤 Output Examples

### Crypto Response (Bullish):
```
💰 *BTC/USDT*

📈 *Price:* 67500.00 USDT
📊 *24h change:* +2.35%
📈 *24h high:* 68000.00
📉 *24h low:* 67000.00

📈 [Open chart on TradingView](https://www.tradingview.com/chart/?symbol=BINANCE:BTCUSDT)
```

### Crypto Response (Bearish):
```
💰 *ETH/USDT*

📉 *Price:* 3450.50 USDT
📊 *24h change:* -1.25%
📈 *24h high:* 3520.00
📉 *24h low:* 3440.00

📈 [Open chart on TradingView](https://www.tradingview.com/chart/?symbol=BINANCE:ETHUSDT)
```

### Forex Response:
```
💱 *USD/RUB*

📉 *Rate:* 71.0350
📊 *Change:* -0.08%

📈 [Open chart on TradingView](https://www.tradingview.com/chart/?symbol=FX_IDC:USDRUB)
```

### Error Response:
```
❌ PAIR NOT FOUND. Examples:
• Rate BTC/USDT
• Rate ETHUSDT
• Rate USDRUB
```

---

## 🔧 Setup Instructions

### 1. Prerequisites

- ✅ Binance API (public, no key required)
- ✅ TradingView API (public, no key required)
- ✅ Telegram Bot Token configured

### 2. Update Bot Token

Replace the bot token in both **Send Telegram Message** nodes:

```
https://api.telegram.org/botYOUR_BOT_TOKEN_HERE/sendMessage
```

Current token in file: `8800811205:AAEKLbn5yJkIliXDEpJteRrCbngHgTwftf0`

### 3. Import Workflow

1. Copy the JSON file content
2. In n8n, go to **Workflows** → **Import from File**
3. Paste or upload the JSON
4. Save the workflow

### 4. Connect to Main Workflow

In the **Secretary Agent** main workflow, update the **Rate** Execute Workflow node with this workflow's ID: `6U0KACxsJymQRoiU`

---

## 🧪 Testing

### Test Crypto:
Send to your bot: `Rate BTC/USDT`

**Expected:** Price response with Binance data and TradingView chart link

### Test Forex:
Send to your bot: `Rate USDRUB`

**Expected:** Rate response with TradingView data and chart link

### Test Error Handling:
Send to your bot: `Rate XYZ/ABC`

**Expected:** Error message with example formats

---

## 📊 API Reference

### Binance API (Crypto)

| Parameter | Value |
|-----------|-------|
| **Base URL** | `https://api.binance.com/api/v3` |
| **Endpoint** | `/ticker/24hr` |
| **Method** | GET |
| **Parameter** | `symbols=["PAIR"]` |
| **Rate Limit** | 1200 requests/minute (IP) |

### TradingView Scanner API (Forex)

| Parameter | Value |
|-----------|-------|
| **URL** | `https://scanner.tradingview.com/forex/scan` |
| **Method** | POST |
| **Content-Type** | application/json |
| **Columns** | `close`, `change` |

---

## 🐛 Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| "PAIR NOT FOUND" | Invalid currency pair | Use format: BTC/USDT, ETHUSDT, or USDRUB |
| No response from Binance | Network issue or invalid symbol | Check Binance API status |
| No response from TradingView | Invalid ticker format | Symbol should be `FX_IDC:BASEQUOTE` |
| Telegram not sending | Invalid bot token | Update token in HTTP Request nodes |
| Wrong exchangeType | Currency not in lists | Add new currency to cryptoList or fiatList |

---

## 🔄 Adding New Currencies

### Add Cryptocurrency:

In **Parse Request** node, update the `cryptoList` array:

```javascript
const cryptoList = ['BTC', 'ETH', 'BNB', 'SOL', 'XRP', 'ADA', 'DOGE', 'TON', 'LINK', 'TRX', 'MATIC', 'DOT', 'AVAX', 'YOUR_NEW_COIN'];
```

### Add Fiat Currency:

In **Parse Request** node, update the `fiatList` array:

```javascript
const fiatList = ['USD', 'RUB', 'EUR', 'GBP', 'JPY', 'CNY', 'CHF', 'AED', 'THB', 'TRY', 'YOUR_NEW_FIAT'];
```

### Add Special Crypto Pair:

In **Parse Request** node, update the `specialCrypto` array:

```javascript
const specialCrypto = ['ETHUSDT', 'MATICUSDT', 'AVAXUSDT', 'DOTUSDT', 'LINKUSDT', 'TONUSDT', 'YOURPAIRUSDT'];
```

---

## 📋 Dependencies

| Workflow | Relationship |
|----------|--------------|
| Secretary Agent (Main) | Calls this workflow when user types "Rate..." |

---

## ⚠️ Notes

1. This workflow does NOT run standalone - it's triggered by the main workflow
2. Keep `active` status as `false` (triggered mode)
3. Binance API is public - no authentication required
4. TradingView scanner is public - no authentication required
5. Forex rates are returned with 4 decimal places
6. Crypto prices are returned with 2 decimal places

---

## 🔐 Security

- No user data is stored
- All API calls are read-only

---

**Version:** 1.0 | **Last Updated:** 2026-05-31