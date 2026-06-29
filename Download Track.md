# Download Track

## 📋 What This Workflow Does

This workflow handles **music download requests** triggered by button clicks from the main Secretary Agent. When a user selects a track from search results, this workflow fetches the audio from Deezer, resolves the best streaming source via Song.link, and sends the MP3 file back to the user with cover art and metadata.

**This workflow is triggered by the main Secretary Agent workflow via callback queries.**

---

## Quick Overview

| User action | What happens |
|-------------|--------------|
| Clicks a track button (`dz\|123456789`) | Track ID extracted → API calls → audio downloaded → MP3 sent back |

---

## 📊 Workflow Architecture

```
User clicks track button
    ↓
Parse dz callback (extracts track_id, user data)
    ↓
Code (builds Deezer + Song.link URLs)
    ↓
HTTP Request Deezer (gets track metadata)
    ↓
HTTP Request Song.link (gets streaming links)
    ↓
Code fallback (resolves best stream URL)
    ↓
Execute Command (yt-dlp downloads audio)
    ↓
Read/Write Files from Disk (reads MP3 file)
    ↓
Code Give Correct Name to File (renames to "Artist - Title.mp3")
    ↓
HTTP Request Get Image for File (downloads cover art)
    ↓
Prepare Thumbnail (converts to thumbnail for Telegram)
    ↓
Send Music to user (sends MP3 with cover art and metadata)
    ↓
Execute Command Delete file from Server (cleans up)
```

---

## 🔧 Node Reference

| Node | Type | What it does |
|------|------|--------------|
| **When Executed by Another Workflow** | Trigger | Receives callback from main workflow |
| **Parse dz callback** | Code | Extracts track_id, user data, business_connection_id |
| **Code** | Code | Builds Deezer and Song.link API URLs |
| **HTTP Request Deezer** | HTTP | Fetches track metadata (title, artist, duration, cover) |
| **HTTP Request** | HTTP | Fetches streaming links from Song.link |
| **Code fallback** | Code | Resolves best stream URL (YouTube > SoundCloud > yt-dlp search) |
| **Execute Command** | Command | Runs `yt-dlp` to download MP3 |
| **Read/Write Files from Disk** | File | Reads downloaded MP3 file |
| **Code Give Correct Name to File** | Code | Renames file to `Artist - Title.mp3` |
| **HTTP Request Get Image for File** | HTTP | Downloads cover art image |
| **Prepare Thumbnail** | Code | Converts cover art to thumbnail format |
| **Send Music to user** | HTTP | Sends MP3 with metadata and thumbnail |
| **Execute Command Delete file from Server** | Command | Removes temporary MP3 |
| **Send Unavailable Message1** | HTTP | Sends error if track not found |

---

## 🧠 Stream URL Resolution Logic

| Priority | Source | How it works |
|----------|--------|--------------|
| 1 | YouTube (direct) | Uses Song.link YouTube URL |
| 2 | SoundCloud (direct) | Uses Song.link SoundCloud URL |
| 3 | YouTube (yt-dlp search) | Searches `ytsearch:Artist - Title official audio` |

---

## 📤 Example Output (to user)

**Input:** User clicks track button for "Lose Yourself" by Eminem

**Output:** MP3 file with:
- **Title:** Lose Yourself
- **Performer:** Eminem
- **Duration:** 326 seconds
- **Thumbnail:** Album cover art
- **Caption:** `PocketAgent`

---

## 📡 API Details

### Deezer API
**Endpoint:** `https://api.deezer.com/track/{track_id}`
**Method:** GET
**Response:** Track metadata (title, artist, duration, album cover URL)

### Song.link API
**Endpoint:** `https://api.song.link/v1-alpha.1/links?url=https://www.deezer.com/track/{track_id}&userCountry=US&songIfSingle=true`
**Method:** GET
**Response:** Streaming links across platforms (YouTube, Spotify, Apple Music, etc.)

### yt-dlp Command
```bash
yt-dlp -x --audio-format mp3 --audio-quality 128K -o "/tmp/ps_{track_id}.mp3" "{stream_url}"
```

---

## 🔧 Setup Instructions

### Prerequisites
- **yt-dlp** installed on the server
- **ffmpeg** installed (for audio conversion)

### Install yt-dlp:
```bash
pip install yt-dlp
```

### Install ffmpeg:
```bash
# Ubuntu/Debian
sudo apt install ffmpeg

# macOS
brew install ffmpeg
```

---

## 🛠️ Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Track not found | No stream URL available | "Send Unavailable Message1" handles this |
| yt-dlp fails | Missing ffmpeg or outdated yt-dlp | Update: `yt-dlp -U` |
| File not deleted | Permission issue | Check `/tmp/` write permissions |
| No audio sent | `yt-dlp` error | Check error output branch |

---

## 📁 File Information

| Property | Value |
|----------|-------|
| **Workflow Name** | PocketAI Music Downloader |
| **Trigger** | Callback query from main workflow |
| **Active Status** | `false` (triggered by main workflow) |

---

## 🔗 Dependencies

| Dependency | Purpose |
|------------|---------|
| Deezer API | Track metadata |
| Song.link API | Platform streaming links |
| yt-dlp | Audio download |
| ffmpeg | Audio conversion |

---

**Version:** 1.1 | **Last Updated:** 2026-06-29
