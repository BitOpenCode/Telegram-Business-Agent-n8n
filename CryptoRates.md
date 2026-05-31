# Crypto Rates - Cryptocurrency Dashboard

## 📋 What This Workflow Does

This workflow handles the **"Crypto rates"** command in Telegram. When a user types `Crypto rates`, it fetches real-time prices for the top 10 cryptocurrencies from Binance and returns a formatted dashboard.

**This workflow is triggered by the main Secretary Agent workflow.**

---

## Quick Overview

| User types | What happens |
|------------|--------------|
| `Crypto rates` | Returns top 10 crypto prices with 24h changes |

### Bonus: Single Coin Lookup

| User types | Returns |
|------------|---------|
| `Crypto rates bitcoin` or `Crypto rates btc` | Bitcoin price only |
| `Crypto rates ethereum` or `Crypto rates eth` | Ethereum price only |
| `Crypto rates solana` or `Crypto rates sol` | Solana price only |
| `Crypto rates dogecoin` or `Crypto rates doge` | Dogecoin price only |
| `Crypto rates ton` or `Crypto rates toncoin` | Toncoin price only |

> **Note:** Single coin mode is currently limited to these 5 coins. The dashboard mode shows all 10.

---

## 📊 Top 10 Cryptocurrencies Included

| # | Coin | Symbol |
|---|------|--------|
| 1 | Bitcoin | BTC |
| 2 | Ethereum | ETH |
| 3 | Binance Coin | BNB |
| 4 | Ripple | XRP |
| 5 | Solana | SOL |
| 6 | Cardano | ADA |
| 7 | Dogecoin | DOGE |
| 8 | TRON | TRX |
| 9 | Chainlink | LINK |
| 10 | Toncoin | TON |

---

## 📤 Example Output

### Dashboard Mode (default):
```
💰 *Rates (24h)*

1. *BTC*: 📉 73499.52 USDT (-0.61%)
2. *ETH*: 📉 2000.85 USDT (-1.26%)
3. *BNB*: 📈 708.66 USDT (+0.19%)
4. *XRP*: 📉 1.33 USDT (-1.45%)
5. *SOL*: 📉 81.62 USDT (-1.46%)
6. *ADA*: 📉 0.23 USDT (-1.65%)
7. *DOGE*: 📉 0.10 USDT (-2.11%)
8. *TRX*: 📉 0.35 USDT (-0.03%)
9. *LINK*: 📉 9.04 USDT (-1.95%)
10. *TON*: 📈 1.87 USDT (+2.69%)
```

**Emoji indicators:**
- 📈 = Price increased (positive change)
- 📉 = Price decreased (negative change)

---

## 🏗️ Workflow Architecture

```
User: "Crypto rates"
    ↓
Main workflow routes to this sub-workflow
    ↓
Parse Crypto Request (detects mode: dashboard or single coin)
    ↓
Get Binance Ticker (fetches prices for 20+ coins)
    ↓
Format Crypto Response (extracts top 10, builds message)
    ↓
Send Telegram Message (sends to user)
```

---

## 🔧 Node Reference

| Node | Type | What it does |
|------|------|--------------|
| **When Executed by Another Workflow** | Trigger | Receives input from main workflow |
| **Parse Crypto Request** | Code | Detects single coin or dashboard mode |
| **Get Binance Ticker** | HTTP Request | Fetches prices from Binance API |
| **Format Crypto Response** | Code | Extracts top 10 coins, builds dashboard |
| **Send Telegram Message** | HTTP Request | Sends response to user |

---

## 📡 API Call Details

### Binance API

**Endpoint:** `https://api.binance.com/api/v3/ticker/24hr`

**Method:** GET

**Parameters:**
```
symbols=["BTCUSDT","ETHUSDT","BNBUSDT","XRPUSDT","SOLUSDT","ADAUSDT","DOGEUSDT","TRXUSDT","LINKUSDT","DOTUSDT","AVAXUSDT","MATICUSDT","SHIBUSDT","TONUSDT","ATOMUSDT","LTCUSDT","NEARUSDT","ETCUSDT","FILUSDT","APTUSDT"]
```

**Full URL:**
```
https://api.binance.com/api/v3/ticker/24hr?symbols=["BTCUSDT","ETHUSDT","BNBUSDT","XRPUSDT","SOLUSDT","ADAUSDT","DOGEUSDT","TRXUSDT","LINKUSDT","DOTUSDT","AVAXUSDT","MATICUSDT","SHIBUSDT","TONUSDT","ATOMUSDT","LTCUSDT","NEARUSDT","ETCUSDT","FILUSDT","APTUSDT"]
```

**Response for each coin includes:**
- `symbol` - Trading pair (e.g., "BTCUSDT")
- `lastPrice` - Current price
- `priceChangePercent` - 24h change percentage
- `highPrice` - 24h high
- `lowPrice` - 24h low

---

## 🎯 Mode Detection

### Parse Crypto Request Node

This node detects what the user wants:

| User input | Mode | Symbol | Coin Name |
|------------|------|--------|-----------|
| `Crypto rates` | `top` (dashboard) | N/A | N/A |
| `Crypto rates bitcoin` | `single` | BTCUSDT | Bitcoin (BTC) |
| `Crypto rates btc` | `single` | BTCUSDT | Bitcoin (BTC) |
| `Crypto rates ethereum` | `single` | ETHUSDT | Ethereum (ETH) |
| `Crypto rates eth` | `single` | ETHUSDT | Ethereum (ETH) |
| `Crypto rates solana` | `single` | SOLUSDT | Solana (SOL) |
| `Crypto rates sol` | `single` | SOLUSDT | Solana (SOL) |
| `Crypto rates doge` | `single` | DOGEUSDT | Dogecoin (DOGE) |
| `Crypto rates ton` | `single` | TONUSDT | Toncoin (TON) |

**Note:** Single coin mode currently only works for these 5 coins. All other inputs default to dashboard mode.

---

## 📝 Code Explanation

### Parse Crypto Request
Detects if user wants a specific coin or the full dashboard:

```javascript
let mode = 'top'; // default: dashboard

if (text.includes('bitcoin') || text.includes('btc')) {
  mode = 'single';
  symbol = 'BTCUSDT';
  coinName = 'Bitcoin (BTC)';
}
// ... similar for ETH, SOL, DOGE, TON
```

### Format Crypto Response
Extracts top 10 coins and builds the dashboard:

```javascript
const topCoinsOrder = ['BTC', 'ETH', 'BNB', 'XRP', 'SOL', 'ADA', 'DOGE', 'TRX', 'LINK', 'TON'];

// Build formatted message
for (let i = 0; i < results.length; i++) {
  const coin = results[i];
  const trend = coin.change >= 0 ? '📈' : '📉';
  const changeSign = coin.change >= 0 ? '+' : '';
  text += `${i+1}. *${coin.name}*: ${trend} ${coin.price} USDT (${changeSign}${coin.change}%)\n`;
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
In the **Secretary Agent** main workflow, update the **Crypto rates** Execute Workflow node with this workflow's ID: `vJk1238BRl23ug2ThRv`

### Step 4: Activate
- This workflow should be **inactive** (triggered by main workflow)
- Keep `active: false`

---

## 🧪 Testing

### Test Dashboard:
Send to your bot: `Crypto rates`

**Expected:** 10-line dashboard with prices and changes

### Test Single Coin:
Send to your bot: `Crypto rates bitcoin`

**Expected:** Bitcoin price (Note: returns dashboard currently - single mode needs implementation)

### Error Handling:
If API returns no data:
```
❌ Not found.
```

---

## 🔧 Full Coin List (20 coins fetched)

| # | Coin | Symbol |
|---|------|--------|
| 1 | Bitcoin | BTCUSDT |
| 2 | Ethereum | ETHUSDT |
| 3 | Binance Coin | BNBUSDT |
| 4 | Ripple | XRPUSDT |
| 5 | Solana | SOLUSDT |
| 6 | Cardano | ADAUSDT |
| 7 | Dogecoin | DOGEUSDT |
| 8 | TRON | TRXUSDT |
| 9 | Chainlink | LINKUSDT |
| 10 | Polkadot | DOTUSDT |
| 11 | Avalanche | AVAXUSDT |
| 12 | Polygon | MATICUSDT |
| 13 | Shiba Inu | SHIBUSDT |
| 14 | Toncoin | TONUSDT |
| 15 | Cosmos | ATOMUSDT |
| 16 | Litecoin | LTCUSDT |
| 17 | NEAR Protocol | NEARUSDT |
| 18 | Ethereum Classic | ETCUSDT |
| 19 | Filecoin | FILUSDT |
| 20 | Aptos | APTUSDT |

**Only top 10 are shown in dashboard.** The rest are fetched but not displayed.

---

## 📊 Adding New Coins to Dashboard

### Step 1: Add to `topCoinsOrder` array (Format Crypto Response node):
```javascript
const topCoinsOrder = [
  'BTC', 'ETH', 'BNB', 'XRP', 'SOL', 'ADA', 'DOGE', 'TRX', 'LINK', 'TON',
  'AVAX'  // Add new coin here
];
```

### Step 2: Ensure coin is in Binance request (Get Binance Ticker node):
```javascript
symbols=["BTCUSDT","ETHUSDT",...,"AVAXUSDT"]
```

---

## 🐛 Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| No response | Bot token incorrect | Update token in HTTP Request node |
| "Not found" | API returned empty data | Check Binance API status |
| Missing coins | Symbol not in request | Add symbol to Binance URL |
| Wrong price format | API response changed | Check Binance API documentation |
| Workflow not triggered | Not connected to main workflow | Verify workflow ID in main workflow |
| Single coin not working | Mode detection incomplete | Check text.includes() conditions |

---

## 📁 File Information

| Property | Value |
|----------|-------|
| **File Name** | `Crypto rates.json` |
| **Workflow Name** | Crypto rates |
| **Workflow ID** | `vJkb8BRltug2ThRv` |
| **Active Status** | `false` (triggered by main workflow) |
| **Tags** | `reddit` |

---

## 🔗 Dependencies

| Workflow | Relationship |
|----------|--------------|
| Secretary Agent (Main) | Calls this workflow when user types "Crypto rates" |

---

## ⚡ Performance

- **API Calls:** 1 call per request
- **Data Points:** 20+ cryptocurrencies
- **Response Time:** ~1 second
- **Rate Limit:** 1200 requests per minute per IP (Binance)

---

## 📝 Notes

1. This workflow does NOT run standalone - triggered by main workflow
2. Prices are in USDT (Tether)
3. All prices shown with 2 decimal places
4. Change percentages show with +/- sign
5. No API key required (public Binance endpoint)
6. Single coin mode is currently limited - only 5 coins recognized
7. Dashboard always shows top 10 by market cap order

---

## 🔐 Security

- No user data stored
- Read-only API calls
- Binance API is public - no authentication needed

---

## 🚀 Future Improvements

- [ ] Complete single coin mode for all 20 coins
- [ ] Add 24h high/low to single coin responses
- [ ] Add market cap data
- [ ] Add volume data
- [ ] Support for more fiat currencies (EUR, RUB, etc.)

---

**Version:** 1.0 | **Last Updated:** 2026-05-31