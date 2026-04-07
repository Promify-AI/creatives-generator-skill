# Promify Creatives Generator Skill

A creatives generation skill that automatically generates high-quality ad creatives from product links via the [Promify](https://promify.ai) API.

---

## Features

Provide a product page URL and the skill will automatically:

1. Scrape the product name, description, price, and main image
2. Submit a creative generation task to the Promify API
3. Poll for the result and return:
   - Ad image (image URL)
   - Ad copy (Creative Copy)
   - Remaining quota

Supports generating creatives for **Meta Ads** (Facebook / Instagram), **Google Ads**, and more.

---

## Quick Start

### 1. Get a Promify API Key

Visit [https://promify.ai](https://promify.ai) to create an account and obtain your API Key.

### 2. Install the Skill in OpenClaw

1. Copy `SKILL.md` to your OpenClaw skills directory:

   ```bash
   cp SKILL.md ~/.openclaw/skills/promify-creatives-generator.md
   ```

2. Start a new conversation and use any trigger phrase to invoke the skill.

### 3. Configure Your API Key

On first use, the skill will check for a saved API Key. If none is found, just provide it when prompted — the skill saves it automatically to:

```
~/.promify/promify-creatives-generator.json
```

Or create it manually:

```bash
mkdir -p ~/.promify
echo '{"apiKey":"your_api_key_here"}' > ~/.promify/promify-creatives-generator.json
```

### 4. Generate Creatives

Paste a product link and describe what you need:

```
Generate ad creatives for this product: https://yourstore.com/products/summer-shirt
```

---

## Configuration

| Path | Description |
|------|-------------|
| `~/.promify/promify-creatives-generator.json` | Stores the Promify API Key |

File format:

```json
{
  "apiKey": "your_promify_api_key"
}
```

---

## Trigger Phrases

Any of the following phrases will activate this skill:

| Trigger |
|---------|
| generate creatives |
| ad creative |
| generate ad image |
| create ad from product link |
| Meta ad |
| Facebook ad |
| Instagram ad |
| Google ad |
| product url |

---

## Notes

- **No auto-retry on failure**: If an API call fails, the skill reports the error directly. Type `retry` to try again.
- **Timeout handling**: Tasks wait up to 3 minutes (polling every 10 seconds, 18 attempts). If timed out, type `retry` to resume polling.
- **Quota limits**: Daily quota resets at UTC 00:00. You'll be notified when exhausted; quota is refunded if a task fails.

---

## About Promify

[Promify](https://promify.ai) is an AI marketing platform that automatically generates professional ad creatives and launches fully optimized campaigns on Meta Ads and Google Ads, powered by leading AI models from OpenAI, Google, and Anthropic.
