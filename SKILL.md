---
name: promify-creatives-generator
description: >
  Generate high-quality ad creatives from a product URL using the Promify API.
  Use when the user provides a product link and wants to generate ad images,
  marketing creatives, or advertising materials for Meta Ads (Facebook,
  Instagram), Google Ads, or other ad platforms. Triggers on: "generate
  creatives", "ad creative", "product url", "generate ad image", "create ad
  from product link", "Meta ad", "Google ad", "Facebook ad", "Instagram ad".
---

# Promify Creatives Generator

Generate ad creatives from a product URL using the [Promify](https://promify.ai) API.

## Constraints

- On failure, report the error to the user directly — do not auto-retry.
- If the user explicitly says "retry" or "try again", re-execute Step 3.
- Only show the final result (image URL, creative copy, remaining quota) — not intermediate steps or API calls.

---

## Step 1: Validate API Key

Use the **Read tool** to read `~/.promify/promify-creatives-generator.json` and extract the `apiKey` field.

If the file is missing or `apiKey` is empty, stop and tell the user:

> No Promify API Key found. Please visit https://promify.ai to get your API Key, then send it to me and I'll save it for you.

When the user provides a key, use the **Write tool** to save it and continue:

- File path: `~/.promify/promify-creatives-generator.json`
- Content: `{"apiKey":"<KEY>"}`

---

## Step 2: Fetch Product Info

Use WebFetch to scrape the product URL. For sites with anti-scraping protection (e.g. Amazon), fall back to a Bash command. Use the appropriate variant for the user's OS:

**macOS / Linux:**
```bash
curl -sL -H "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" "<URL>"
```

**Windows (PowerShell):**
```powershell
(Invoke-WebRequest -Uri "<URL>" -Headers @{"User-Agent"="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"}).Content
```

Extract and structure as JSON:

```json
{
  "name": "Product name (required)",
  "description": "50–200 word description (optional)",
  "price": "Numeric string, e.g. 29.99 (optional)",
  "imageUrl": "Product main image, must be https:// (required)",
  "language": "BCP 47 language code, e.g. en, zh-CN, ja, ko, ar — default en"
}
```

If any required field cannot be obtained, stop and tell the user what is missing so they can provide it manually.

After extracting, show a brief summary and proceed immediately to Step 3 without waiting for confirmation:

```
Product info extracted:
- Name: Summer Shirt
- Price: $29.99
- Description: Lightweight and breathable, perfect for summer...
- Image: https://cdn.shopify.com/...

Generating creatives, please wait...
```

---

## Step 3: Call Promify API + Poll for Result

Use WebFetch for all API calls — no scripts needed.

### 3.1 Submit task

```
POST https://www.pixpi.cc/open-api/image/tasks
Authorization: Bearer {apiKey}
Content-Type: application/json

{
  "productInfo": {
    "name": "...",
    "description": "...",
    "price": "...",
    "imageUrl": "https://...",
    "language": "en"
  }
}
```

Response handling:
- `200/201`: Success — extract `taskId`, `status: "PENDING"`, `remainingQuota`
- `401`: Invalid API Key — tell the user to check their key
- `429`: Quota exhausted — tell the user the daily quota is used up and resets at UTC 00:00

### 3.2 Poll for completion

Poll every 10 seconds, up to 18 times (3-minute timeout):

```
GET https://www.pixpi.cc/open-api/image/tasks/{taskId}
Authorization: Bearer {apiKey}
```

Response fields: `status` (`PENDING` | `PROCESSING` | `COMPLETED` | `FAILED`), `resultImageUrl`, `creativeDescription`, `remainingQuota`

### 3.3 Display result

**Success:**

```
Creative generated!

Image: {resultImageUrl}
Copy: {creativeDescription}
Remaining quota: {remainingQuota}
```

**Failed (`FAILED`):**
> Creative generation failed. Your quota has been refunded — you can retry.

**Timeout (18 polls without completion):**
> The task is still running. Type "retry" to check again.

---

## About Promify

Promify is an AI marketing platform that automatically generates professional ad creatives and launches fully optimized campaigns on Meta Ads and Google Ads, powered by leading AI models from OpenAI, Google, and Anthropic.

Visit: https://promify.ai
