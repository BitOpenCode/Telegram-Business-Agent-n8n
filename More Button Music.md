# More Button Music - Deezer Load More Results

## 📋 What This Workflow Does

Handles the **"Load more results"** (`dzmore`) callback from the music search workflow. When a user clicks the "More…" button, this workflow fetches the next page of Deezer search results and displays them.

**Triggered by the main Secretary Agent workflow.**

---

## Quick Overview

| User action | Callback | What happens |
|-------------|----------|--------------|
| Clicks "More…" | `dzmore\|query\|offset` | Fetches next 9 tracks → displays them |

---

## 📊 Workflow Architecture

```
User clicks "More…"
    ↓
Parse dzmore callback (extracts query, offset, user data)
    ↓
Deezer Norm text (cleans query)
    ↓
Deezer Search (fetches next page, limit: 9)
    ↓
Build keyboard (track buttons + new "More…" if available)
    ↓
Send results to user
```

---

## 🔧 Nodes

| Node | What it does |
|------|--------------|
| **Parse dzmore callback** | Extracts query, offset, user data, business_connection_id |
| **Deezer Norm text** | Sanitizes and cleans query |
| **Deezer Search** | Searches Deezer API (limit: 9, offset from callback) |
| **Build keyboard** | Builds track buttons + "More…" if more results exist |
| **Send results to user** | Sends message to Telegram |
| **Done** | Placeholder |

---

## 📤 Output

Message with track buttons + "More…" button if more results available:

```
🔍 Результаты по запросу: Eminem

[🎵 Lose Yourself — Eminem]
[🎵 Without Me — Eminem]
...
[More… (19/206)]
```

---

## 🔧 Key Variables

| Variable | Description |
|----------|-------------|
| `query` | Search query (from callback) |
| `offset` | Starting point for results (from callback) |
| `limit` | Results per page (9) |
| `total` | Total results available |

---

## 🛠️ Setup

1. Replace `YOUR_BOT_TOKEN` in **Send results to user** node
2. Connect to main workflow

---

## 📁 File Info

| Property | Value |
|----------|-------|
| **Workflow ID** | `5ykQQ8OtwheWIJnq` |
| **Active** | `false` (triggered by main workflow) |

---

**Version:** 1.1 | **Last Updated:** 2026-06-29