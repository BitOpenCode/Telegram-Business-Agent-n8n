# AI Video & Image Generator - UGC Ad Creator

## 📋 What This Workflow Does

This workflow generates a **UGC-style video ad** from a user-submitted photo. When a user sends a photo with a description, the workflow:
1. Analyzes the image (AI)
2. Generates an enhanced version of the image (Nano Banana 2)
3. Creates a cinematic video ad (Veo 3) based on the generated image

**Triggered by the main Secretary Agent workflow.**

---

## Quick Overview

| User sends | What happens |
|------------|--------------|
| Photo + caption | Image analyzed → Enhanced image generated → Video ad created |

---

## 📊 Workflow Architecture

```
User sends photo with caption
    ↓
Parse ART Command (extracts image, user data, caption)
    ↓
Get File TG + Download File (downloads image from Telegram)
    ↓
Get Image URL (uploads to ImgBB for public URL)
    ↓
Analyze image AI (Gemma free) — describes the image
    ↓
Generate Image Prompt (Gemini Flash) — creates enhanced image prompt
    ↓
Delete AI Bullshit md (cleans markdown/formatting)
    ↓
Image Input (prepares data for KIE)
    ↓
Generate Image - Nano Banana (KIE AI) — generates enhanced image
    ↓
Wait 30 Sec (wait for generation)
    ↓
Get Generated Image URL (checks status)
    ↓
Switch (routes by state: success / waiting / fail)
    ↓
Send Image — delivers enhanced image to user
    ↓
Video Input (prepares data for Veo)
    ↓
Generate Video - Veo (KIE AI) — generates video ad
    ↓
Wait 30 Sec1 (wait for generation)
    ↓
Get Generated Video URL (checks status)
    ↓
Switch1 (routes by state: success / waiting / fail)
    ↓
Send Video — delivers video to user
```

---

## 🔧 Node Reference

| Node | Type | What it does |
|------|------|--------------|
| **When Executed by Another Workflow** | Trigger | Receives input from main workflow |
| **Parse ART Command** | Code | Extracts image, user data, business_connection_id, caption |
| **Get File TG** | HTTP Request | Gets file metadata from Telegram |
| **Download File** | HTTP Request | Downloads the actual image |
| **Get Image URL** | HTTP Request | Uploads to ImgBB, returns public URL |
| **Analyze image AI** | HTTP Request | Gemma (free) — describes image in plain text |
| **Generate Image Prompt** | HTTP Request | Gemini Flash — creates enhanced image prompt |
| **Delete AI Bullshit md** | Code | Cleans markdown/formatting from response |
| **Image Input** | Set | Prepares data for KIE API |
| **Generate Image - Nano Banana** | HTTP Request | KIE AI — generates enhanced image |
| **Wait 30 Sec2** | Wait | Waits for generation to complete |
| **Get Generated Image URL1** | HTTP Request | Checks generation status |
| **Switch2** | Switch | Routes by state (success/waiting/fail) |
| **Send Image** | HTTP Request | Sends enhanced image to user |
| **Video Input1** | Set | Prepares data for Veo API |
| **Generate Video - Veo** | HTTP Request | KIE AI — generates video ad |
| **Wait 30 Sec1** | Wait | Waits for generation to complete |
| **Get Generated Video URL** | HTTP Request | Checks generation status |
| **Switch1** | Switch | Routes by state (success/waiting/fail) |
| **Send Video** | HTTP Request | Sends video to user |
| **No Operation, do nothing** | NoOp | Handles fail states gracefully |

---

## 🧠 AI Pipeline

### Image Generation

| Step | Model | Provider | Purpose |
|------|-------|----------|---------|
| 1 | `google/gemma-4-26b-a4b-it:free` | OpenRouter | Image description (plain text) |
| 2 | `google/gemini-3.1-flash-image` | OpenRouter | Enhanced image prompt generation |
| 3 | `nano-banana-2` | KIE AI | Enhanced image generation |

### Video Generation

| Step | Model | Provider | Purpose |
|------|-------|----------|---------|
| 4 | `veo3_lite` | KIE AI | Cinematic video ad generation |

---

## 📤 Example Output

### Input:
User sends photo of a car + caption: `"Video. Create a cinematic hip-hop car advert."`

### Output (Image):
- Enhanced version of the user's image
- 1:1 aspect ratio, 1K resolution, PNG format

### Output (Video):
- Cinematic UGC-style video ad
- Dramatic lighting, slow motion
- Urban night setting, neon reflections
- 16:9 aspect ratio, 1080p resolution
- 8 seconds duration

---

## 📡 API Details

### ImgBB (Image Upload)
**Endpoint:** `https://api.imgbb.com/1/upload`
**Auth:** API key in URL (`key=YOUR_KEY`)
**Method:** POST (multipart/form-data)

### OpenRouter (AI Models)
**Endpoint:** `https://openrouter.ai/api/v1/chat/completions`
**Auth:** `Bearer sk-or-v1-YOUR_TOKEN`

**Models used:**
- `google/gemma-4-26b-a4b-it:free` — Image analysis (free)
- `google/gemini-3.1-flash-image` — Enhanced image prompt generation

### KIE AI (Image & Video Generation)
**Image Generation:**
- **Endpoint:** `https://api.kie.ai/api/v1/jobs/createTask`
- **Model:** `nano-banana-2`
- **Params:** aspect_ratio: 1:1, resolution: 1K, format: png

**Video Generation:**
- **Endpoint:** `https://api.kie.ai/api/v1/veo/generate`
- **Model:** `veo3_lite`
- **Params:** aspect_ratio: 16:9, resolution: 1080p, duration: 8s

**Status Check:** `https://api.kie.ai/api/v1/jobs/recordInfo` or `https://api.kie.ai/api/v1/veo/record-info`

---

## 🛠️ Setup Instructions

### Step 1: Get API Keys

| Service | What you need | Where to get |
|---------|---------------|--------------|
| **ImgBB** | API key | https://api.imgbb.com/ |
| **OpenRouter** | API key (`sk-or-v1-...`) | https://openrouter.ai/ |
| **KIE AI** | API key | https://kie.ai/api-key |

### Step 2: Update Tokens in Nodes

**Replace placeholders in all nodes:**

| Placeholder | Where |
|-------------|-------|
| `YOUR_BOT_TOKEN` | All Telegram HTTP Request nodes |
| `YOUR_KEY` | Get Image URL node (ImgBB) |
| `YOUR_TOKEN` | OpenRouter nodes (Bearer) |
| `YOUR_TOKEN` | KIE AI nodes (Bearer) |

### Step 3: Connect to Main Workflow
In the **Secretary Agent** main workflow, update the **ART create** Execute Workflow node with this workflow's ID.

### Step 4: Activate
- This workflow should be **inactive** (triggered by main workflow)
- Keep `active: false`

---

## 💰 Cost Breakdown (KIE AI)

| Item | Credits | USD ($) |
|------|---------|---------|
| **Nano Banana 2** (image) | ~8 | ~$0.04 |
| **Veo 3 Lite** (video) | ~35 | ~$0.175 |
| **Total per run** | ~43 | ~$0.215 |

> **Note:** OpenRouter models (Gemma, Gemini Flash) are free with rate limits.
> KIE AI gives **50 free credits per day** for testing.

**50 free credits per day = ~1 video + 1 image per day free**

---

### 💳 Topping Up Credits

For more than 1 video per day or faster processing queues, we recommend topping up your KIE AI balance:

| Payment Method | Status |
|----------------|--------|
| 💳 Credit/Debit Card | ✅ Available |
| ₿ Cryptocurrency | ✅ Available (USDC, USDT, BTC, ETH) |

**Top up here:** [KIE AI Dashboard](https://kie.ai/dashboard)

> **Pro Tip:** Higher credit balance unlocks faster generation queues and priority processing.

---

## 📝 Notes

1. This workflow requires a **photo** — it does NOT work with text-only messages
2. The workflow generates both an enhanced image AND a video ad
3. Video generation uses `veo3_lite` (most cost-effective model)
4. Image generation uses `nano-banana-2`
5. KIE AI gives **50 free credits per day** — enough for ~1 full run (image + video)
6. Video duration is 8 seconds (cost-effective)
7. Aspect ratio: 16:9 for video, 1:1 for image
8. **For higher quality video generation or larger volumes, top up your KIE AI balance via credit/debit card or cryptocurrency**

---

## 🔧 Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|----------|
| Credits insufficient | Free daily credits used up | Wait for next day or top up |
| Video not generated | KIE queue busy | Increase wait time |
| Image analysis fails | OpenRouter token invalid | Check `sk-or-v1-` token |
| Upload fails | ImgBB key expired | Regenerate ImgBB API key |
| No response | Workflow not triggered | Verify workflow ID in main workflow |


---

## 🔗 Dependencies

| Workflow | Relationship |
|----------|--------------|
| Secretary Agent (Main) | Calls this workflow when user sends a photo |
| ImgBB | External — image hosting |
| OpenRouter | External — AI models |
| KIE AI | External — image & video generation |

---

## 🚀 Future Improvements

- [ ] Support for different video styles (cinematic, product, lifestyle)
- [ ] Support for different video durations (4s, 8s, 16s)
- [ ] Add audio selection for video
- [ ] Batch generation
- [ ] Different aspect ratios
- [ ] Higher resolution options

---

**Version:** 1.1 | **Last Updated:** 2026-06-30