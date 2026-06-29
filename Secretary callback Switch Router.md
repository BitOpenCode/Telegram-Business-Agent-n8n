# Secretary callback Switch Router

## 📋 What This Workflow Does

Routes Telegram callback queries (button clicks) to the appropriate handler.

**Triggered by the main Secretary Agent workflow.**

**Version:** 1.1

---

## Quick Overview

| User action | Callback | Status |
|-------------|----------|--------|
| Clicks track | `dz\|123` | ✅ Active → downloads track |
| Clicks playlist | `add_to_playlist\|...` | 🚧 Planned |
| Other routes | — | 🚧 Planned |

---

## 📊 Workflow Architecture

```
Callback query
    ↓
Parse callback data
    ↓
Switch Router (12 routes)
    ↓
    └── dz → Code → Download track workflow
    └── other → (placeholders)
```

---

## 🔧 Nodes

| Node | What it does |
|------|--------------|
| **Parse callback data** | Extracts action, params, user data, business_connection_id |
| **Route to specific handler** | Switch with 12 routes (only `dz` active) |
| **Code** | Prepares data for download |
| **Download track PocketAI** | Triggers download workflow with `track_id` |

---

## 🛠️ Setup

1. Create **Download track PocketAI** workflow
2. Update `workflowId` in Execute Workflow node

---

## 📁 File Information

| Property | Value |
|----------|-------|
| **Version** | 1.1 |
| **Active Status** | `false` (triggered by main workflow) |
| **Routes** | 12 (1 active, 11 placeholders) |

---

**Version:** 1.1 | **Last Updated:** 2026-06-29