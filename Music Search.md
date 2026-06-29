# PocketAI Music Search - Deezer Track Finder

## 📋 What This Workflow Does

This workflow handles **music search requests** from the main Secretary Agent. When a user types `Music [artist]` or `Музыка [artist]`, it searches Deezer, displays the first 10 tracks as interactive buttons, and allows the user to load more results.

**This workflow is triggered by the main Secretary Agent workflow.**

---

## Quick Overview

| User types | What happens |
|------------|--------------|
| `Music Eminem` | Searches Deezer → returns top 10 tracks as buttons |
| `Музыка Eminem` | Searches Deezer → returns top 10 tracks as buttons |

### Example Commands

| User types | Returns |
|------------|---------|
| `Music Travis Scott` | 10 track buttons for Travis Scott |
| `Music Eminem` | 10 track buttons for Eminem |
| `Music Sia` | 10 track buttons for Sia |

> **Note:** The "Music" or "Музыка" prefix is automatically removed before searching.

---

## 📊 Workflow Architecture

```
User: "Music Eminem"
    ↓
Parse search command (extracts query, removes "Music" prefix)
    ↓
Deezer Norm text (cleans and sanitizes query)
    ↓
Deezer (searches Deezer API)
    ↓
Buttons (builds track buttons, adds "More…" if needed)
    ↓
Send API Buttons (sends interactive message to user)
```

---

## 🔧 Node Reference

| Node | Type | What it does |
|------|------|--------------|
| **When Executed by Another Workflow** | Trigger | Receives input from main workflow |
| **Parse search command** | Code | Extracts user data, removes "Music"/"Музыка" prefix |
| **Deezer Norm text** | Code | Sanitizes and URL-encodes the search query |
| **Deezer** | HTTP Request | Searches Deezer API (limit: 25 results) |
| **Buttons** | Code | Builds track buttons, handles no results, adds "More…" |
| **Send API Buttons** | HTTP Request | Sends interactive message to user |
| **No Operation, do nothing11** | NoOp | No-op (placeholder) |

---

## 🧠 Deezer API Details

**Endpoint:** `https://api.deezer.com/search`
**Method:** GET
**Parameters:**
- `q` — Search query (URL-encoded)
- `limit` — Results per page (25)
- `index` — Offset (0 for first page)

**Example URL:**
```
https://api.deezer.com/search?q=Eminem&index=0&limit=25
```

**Response includes:**
- `data` — Array of tracks
- `total` — Total results count
- `next` — URL for next page (if available)

---

## 📤 Example Output

### Input:
```
Music Eminem
```

### Output (interactive message):

```
🔍 Search results for: Eminem
```

**Buttons (10 tracks):**
```
🎵 Lose Yourself — Eminem
🎵 Without Me — Eminem
🎵 Mockingbird — Eminem
🎵 Not Afraid — Eminem
🎵 Rap God — Eminem
🎵 The Real Slim Shady — Eminem
🎵 Stan — Eminem
🎵 Sing For The Moment — Eminem
🎵 My Name Is — Eminem
🎵 The Way I Am — Eminem
```

**"More…" button (if >10 results):**
```
More… (10/25)
```

Each button triggers a callback with the format:
```
dz|{track_id}
```

---

## 📤 Error Message (if no results)

```
❌ Nothing found: <b>Eminem</b>

💡 Try:
• Artist name only
• Song title only
• Different spelling
```

---

## 🎯 Button Structure

| Property | Value |
|----------|-------|
| **Text** | `🎵 {title} — {artist}` |
| **Callback Data** | `dz|{track_id}` |

**Example:**
```json
{
  "text": "🎵 Lose Yourself — Eminem",
  "callback_data": "dz|1109731"
}
```

### "More…" Button

| Property | Value |
|----------|-------|
| **Text** | `More… ({offset}/{total})` |
| **Callback Data** | `dzmore|{query}|{offset}` |

**Example:**
```json
{
  "text": "More… (10/25)",
  "callback_data": "dzmore|Eminem|10"
}
```

---

## 🛠️ Setup Instructions

### Step 1: Update Bot Token
Replace `YOUR_BOT_TOKEN` in **Send API Buttons** node:

```
https://api.telegram.org/botYOUR_BOT_TOKEN/sendMessage
```

### Step 2: Connect to Main Workflow
In the **Secretary Agent** main workflow, update the **Music Search** Execute Workflow node with this workflow's ID.

### Step 3: Activate
- This workflow should be **inactive** (triggered by main workflow)
- Keep `active: false`

---

## 🔧 Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| No results | Query too specific | Try artist name only |
| "Nothing found" | Deezer API down | Check Deezer API status |
| Buttons not showing | Bot token invalid | Update token in HTTP node |
| "More…" not working | Callback handler missing | Ensure dzmore handler exists |
| No response | Workflow not triggered | Verify workflow ID in main workflow |

---

## 📁 File Information

| Property | Value |
|----------|-------|
| **Workflow Name** | PocketAI Music Search |
| **Trigger** | Main Secretary Agent |
| **Active Status** | `false` (triggered by main workflow) |

---

## 🔗 Dependencies

| Dependency | Purpose |
|------------|---------|
| Deezer API | Music search |
| Telegram API | Message delivery |

---

## 📝 Notes

1. This workflow only **searches** for music and displays buttons
2. Clicking a button triggers the **Music Downloader** workflow
3. The "More…" button loads the next 10 tracks (offset + 10)
4. Each track button contains the Deezer track ID in `callback_data`
5. The "Music" or "Музыка" prefix is **required** to trigger this workflow
6. Results are limited to 25 per page (Deezer default)

---

## 🚀 Future Improvements

- [ ] Add support for album search
- [ ] Add support for playlist search
- [ ] Add preview audio button
- [ ] Add sorting options
- [ ] Add genre filter

---

**Version:** 1.0 | **Last Updated:** 2026-06-29