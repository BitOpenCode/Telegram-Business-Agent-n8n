# ART Create - T-Shirt Mockup Generator

## 📋 What This Workflow Does

This workflow generates a **3D T-Shirt mockup** from a user-submitted image. When a user sends a photo with a design, this workflow analyzes the image, creates a detailed prompt, and generates a realistic 3D T-shirt mockup using AI.

**This workflow is triggered by the main Secretary Agent workflow.**

---

## Quick Overview

| User sends | What happens |
|------------|--------------|
| Photo | Image is analyzed → prompt generated → 3D mockup created → returned to user |

---

## 📊 Workflow Architecture

```
User sends photo
    ↓
Parse ART Command (extracts image, user data)
    ↓
Get File TG (gets file info from Telegram)
    ↓
Download File (downloads the image)
    ↓
Get Image URL (uploads to ImgBB for URL)
    ↓
Analyze image AI (Gemma free) - describes the image
    ↓
Generate Image Prompt (GPT-4.1) - creates T-shirt mockup prompt
    ↓
Delete AI Bullshit md (cleans markdown from response)
    ↓
Image Input (prepares data for KIE)
    ↓
Generate Image - Nano Banana 2 (KIE AI) - generates mockup
    ↓
Wait 30 Sec (wait for generation)
    ↓
Get Generated Image URL (checks status)
    ↓
Switch (routes by state: success / waiting / fail)
    ↓
Send Image (delivers mockup to user)
```

---

## 🔧 Node Reference

| Node | Type | What it does |
|------|------|--------------|
| **When Executed by Another Workflow** | Trigger | Receives input from main workflow |
| **Parse ART Command** | Code | Extracts image file_id, user data, chat info |
| **Get File TG** | HTTP Request | Gets file metadata from Telegram |
| **Download File** | HTTP Request | Downloads the actual image |
| **Get Image URL** | HTTP Request | Uploads to ImgBB, returns public URL |
| **Analyze image AI** | HTTP Request | Gemma (free) - describes image in plain text |
| **Generate Image Prompt** | HTTP Request | GPT-4.1 - creates T-shirt mockup prompt |
| **Delete AI Bullshit md** | Code | Cleans markdown/formatting from prompt |
| **Image Input** | Set | Prepares data for KIE API |
| **Generate Image - Nano Banana 2** | HTTP Request | KIE AI - generates the mockup |
| **Wait 30 Sec** | Wait | Waits for generation to complete |
| **Get Generated Image URL** | HTTP Request | Checks generation status |
| **Switch** | Switch | Routes based on state (success/waiting/fail) |
| **Send Image** | HTTP Request | Sends mockup to user |
| **No Operation, do nothing** | NoOp | Handles fail state gracefully |

---

## 🧠 AI Model Pipeline

| Step | Model | Provider | Purpose |
|------|-------|----------|---------|
| 1 | `google/gemma-4-26b-a4b-it:free` | Google (OpenRouter) | **Free** - Image description (plain text, no markdown) |
| 2 | `openai/gpt-4.1` | OpenAI (OpenRouter) | T-shirt mockup prompt generation |
| 3 | `nano-banana-2` | KIE AI | 3D T-shirt mockup generation |

---

## 📤 Example Output

### Input:
User sends a photo of a design (e.g., a cat riding a shark)

### Output (mockup):
- 3D T-shirt mockup with the design printed on a realistic T-shirt
- Professional studio lighting
- Ghost-mannequin style (body shape visible, invisible wearer)

**The generated image is sent back to the user with caption:** `PocketAgent`

---

## 📡 API Details

### ImgBB (Image Upload)
**Endpoint:** `https://api.imgbb.com/1/upload`
**Auth:** API key in URL (`key=YOUR_KEY`)
**Method:** POST (multipart/form-data)
**Purpose:** Uploads image and returns public URL

### OpenRouter (Image Analysis)
**Endpoint:** `https://openrouter.ai/api/v1/chat/completions`
**Auth:** `Bearer sk-or-v1-YOUR_TOKEN`
**Model:** `google/gemma-4-26b-a4b-it:free` (free, rate-limited)

**Prompt:**
```
Describe this image. Focus on colors, typography, layout, and visual details. Return a plain text description without markdown, without bold, without any formatting.
```

### OpenRouter (Prompt Generation)
**Endpoint:** `https://openrouter.ai/api/v1/chat/completions`
**Auth:** `Bearer sk-or-v1-YOUR_TOKEN`
**Model:** `openai/gpt-4.1`

**System Prompt:**
```
You are a T-Shirt Mockup Prompt Builder. Output only JSON with key 'image_prompt'. Prompt must be under 150 words. Preserve the artwork exactly as described. Focus on garment realism, print quality, and professional presentation.
```

### KIE AI (Image Generation)
**Endpoint:** `https://api.kie.ai/api/v1/jobs/createTask`
**Auth:** `Bearer YOUR_TOKEN`
**Model:** `nano-banana-2`

**Parameters:**
- `aspect_ratio`: `1:1`
- `resolution`: `1K`
- `output_format`: `png`

**Status Check:** `https://api.kie.ai/api/v1/jobs/recordInfo?taskId=...`

---

## 🎯 Key Prompts

### Image Description (Analyze image AI)
```
Describe this image. Focus on colors, typography, layout, and visual details. Return a plain text description without markdown, without bold, without any formatting.
```

### Mockup Prompt Builder (Generate Image Prompt)
```
You are a T-Shirt Mockup Prompt Builder. Output only JSON with key 'image_prompt'. Prompt must be under 150 words. Preserve the artwork exactly as described. Focus on garment realism, print quality, and professional presentation.
```

### Output Example (from GPT-4.1):
```json
{
  "image_prompt": "3D T-Shirt mockup featuring a front print of a surreal, whimsical painted illustration: a black cat riding on the back of a shark in the ocean. The artwork uses primarily blue and white tones... Showcase the print as vivid and crisp on a high-quality, realistic cotton T-shirt..."
}
```

### Final KIE Input:
```json
{
  "model": "nano-banana-2",
  "input": {
    "prompt": "3D T-Shirt mockup featuring a front print of a surreal...",
    "image_input": ["https://i.ibb.co/..."],
    "aspect_ratio": "1:1",
    "resolution": "1K",
    "output_format": "png"
  }
}
```

---

## 🛠️ Setup Instructions

### Step 1: Get API Keys

| Service | What you need | Where to get |
|---------|---------------|--------------|
| **ImgBB** | API key | https://api.imgbb.com/ |
| **OpenRouter** | API key (`sk-or-v1-...`) | https://openrouter.ai/ |
| **KIE AI** | API key | https://kie.ai/api-key |

### Step 2: Update Tokens in Nodes

**Get File TG & Download File (2 nodes):**
```
https://api.telegram.org/botYOUR_BOT_TOKEN/...
```
Replace `YOUR_BOT_TOKEN` with your actual Telegram bot token.

**Analyze image AI & Generate Image Prompt (2 nodes):**
```
Authorization: Bearer sk-or-v1-YOUR_TOKEN
```
Replace `YOUR_TOKEN` with your OpenRouter API key.

**Get Image URL:**
```
https://api.imgbb.com/1/upload?key=YOUR_KEY
```
Replace `YOUR_KEY` with your ImgBB API key.

**Generate Image - Nano Banana 2 & Get Generated Image URL (2 nodes):**
```
Authorization: Bearer YOUR_TOKEN
```
Replace `YOUR_TOKEN` with your KIE AI API key.

**Send Image:**
```
https://api.telegram.org/botYOUR_BOT_TOKEN/sendPhoto
```
Replace `YOUR_BOT_TOKEN` with your actual Telegram bot token.

### Step 3: Connect to Main Workflow
In the **Secretary Agent** main workflow, update the **ART Create** Execute Workflow node with this workflow's ID: `wKSSghWslOiYFWc5`

### Step 4: Activate
- This workflow should be **inactive** (triggered by main workflow)
- Keep `active: false`

---

## 🧪 Testing

### Test Basic Image:
Send to your bot: **(any image)**

**Expected:** T-shirt mockup generated and returned

### Error Handling:
If API fails:
```
No Operation, do nothing (silent fallback)
```

If image not found:
```
❌ No output data returned (n8n stops)
```

---

## 🔧 Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Mockup not generated | KIE API token expired | Update token in both KIE nodes |
| Image analysis fails | OpenRouter token invalid | Check `sk-or-v1-` token |
| Upload fails | ImgBB key expired | Regenerate ImgBB API key |
| Rate limit error | Gemma free tier | Wait or switch to paid model |
| JSON parsing error | Markdown in response | "Delete AI Bullshit md" node should handle this |
| Generation timeout | KIE queue busy | Increase wait time (currently 30s) |
| No response | Workflow not triggered | Verify workflow ID in main workflow |

---

## 📁 File Information

| Property | Value |
|----------|-------|
| **File Name** | `ART create PocketAI _r.json` |
| **Workflow Name** | ART create PocketAI /r |
| **Workflow ID** | `wKSSghWslOiYFWc5` |
| **Active Status** | `false` (triggered by main workflow) |
| **Tags** | `art`, `t-shirt`, `mockup`, `generation` |

---

## 🔗 Dependencies

| Workflow | Relationship |
|----------|--------------|
| Secretary Agent (Main) | Calls this workflow when user sends a photo |
| ImgBB | External - image hosting |
| OpenRouter | External - AI models |
| KIE AI | External - image generation |

---

## ⚡ Performance

- **Image Upload:** ~1-2 seconds
- **Image Analysis (Gemma):** ~3-5 seconds (rate-limited)
- **Prompt Generation (GPT-4.1):** ~2-3 seconds
- **Mockup Generation (KIE):** ~20-30 seconds
- **Total Time:** ~30-40 seconds

---

## 💰 Cost Breakdown

| Service | Model | Approx. Cost per request |
|---------|-------|--------------------------|
| ImgBB | Free | $0 |
| OpenRouter | google/gemma-4-26b-a4b-it:free | $0 (rate-limited) |
| OpenRouter | openai/gpt-4.1 | ~$0.01-$0.02 |
| KIE AI | nano-banana-2 | ~$0.005-$0.01 |
| **Total** | | **~$0.01-$0.03 per mockup** |

---

## 📝 Notes

1. This workflow requires a **photo** — it does NOT work with text-only messages
2. The **Delete AI Bullshit md** node is only needed for Gemma (free) model
3. If you use a paid model (e.g., GPT-4o), you can skip the cleanup node
4. KIE AI may have queue times during peak hours
5. Mockups are generated in **1:1** aspect ratio (square)
6. Output format is **PNG** at **1K** resolution

---

## 🔐 Security

- **ImgBB:** Image uploaded publicly (URL accessible)
- **OpenRouter:** API keys stored in credentials
- **KIE AI:** API keys stored in credentials
- **Telegram:** Files downloaded directly from Telegram servers
- **No user data stored** in this workflow

---

## 🚀 Future Improvements

- [ ] Add support for video-to-mockup
- [ ] Add support for multiple mockup angles
- [ ] Add support for different garment types (hoodies, bags, etc.)
- [ ] Add color selector for T-shirt base
- [ ] Add batch generation

---

**Version:** 1.1 | **Last Updated:** 2026-06-29
