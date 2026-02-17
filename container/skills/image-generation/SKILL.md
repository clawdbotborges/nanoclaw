---
name: image-generation
description: Generate and edit images using Google Gemini. Use when the user asks you to create, draw, generate, design, or edit an image.
allowed-tools: Bash(node:*),Write,Read
---

# AI Image Generation with Gemini

Use the Gemini API to generate images from text prompts or edit existing images.

## Quick start

Write a Node.js script and run it:

```bash
node /workspace/group/generate-image.mjs
```

## Script template — Generate from text

```javascript
const GEMINI_API_KEY = process.env.GOOGLE_API_KEY;
const MODEL = 'gemini-2.5-flash-image';
const OUTPUT = '/workspace/group/generated-image.png';

const prompt = 'A serene mountain landscape at sunset with a lake reflection';

const resp = await fetch(
  `https://generativelanguage.googleapis.com/v1beta/models/${MODEL}:generateContent?key=${GEMINI_API_KEY}`,
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      contents: [{ parts: [{ text: prompt }] }],
      generationConfig: {
        responseModalities: ['TEXT', 'IMAGE'],
        imageConfig: { imageSize: '2K' }
      }
    })
  }
);

const data = await resp.json();
if (!resp.ok) { console.error('API error:', JSON.stringify(data)); process.exit(1); }

const parts = data.candidates?.[0]?.content?.parts || [];
for (const part of parts) {
  if (part.inlineData) {
    const fs = await import('fs');
    fs.writeFileSync(OUTPUT, Buffer.from(part.inlineData.data, 'base64'));
    console.log(`Saved: ${OUTPUT}`);
  }
  if (part.text) {
    console.log('Model note:', part.text);
  }
}
```

## Script template — Edit an existing image

```javascript
import { readFileSync, writeFileSync } from 'fs';

const GEMINI_API_KEY = process.env.GOOGLE_API_KEY;
const MODEL = 'gemini-2.5-flash-image';
const INPUT = '/workspace/group/input-image.png';
const OUTPUT = '/workspace/group/edited-image.png';

const imageData = readFileSync(INPUT).toString('base64');
const mimeType = INPUT.endsWith('.png') ? 'image/png' : 'image/jpeg';

const resp = await fetch(
  `https://generativelanguage.googleapis.com/v1beta/models/${MODEL}:generateContent?key=${GEMINI_API_KEY}`,
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      contents: [{
        parts: [
          { text: 'Remove the background and replace it with a solid white background' },
          { inlineData: { mimeType, data: imageData } }
        ]
      }],
      generationConfig: {
        responseModalities: ['TEXT', 'IMAGE'],
        imageConfig: { imageSize: '2K' }
      }
    })
  }
);

const data = await resp.json();
if (!resp.ok) { console.error('API error:', JSON.stringify(data)); process.exit(1); }

const parts = data.candidates?.[0]?.content?.parts || [];
for (const part of parts) {
  if (part.inlineData) {
    writeFileSync(OUTPUT, Buffer.from(part.inlineData.data, 'base64'));
    console.log(`Saved: ${OUTPUT}`);
  }
  if (part.text) {
    console.log('Model note:', part.text);
  }
}
```

## Models

| Model | Best for | Speed |
|-------|----------|-------|
| `gemini-2.5-flash-image` | Fast generation, good quality | Fast |
| `gemini-3-pro-image-preview` | Highest quality, complex prompts | Slower |

Use `gemini-2.5-flash-image` by default. Switch to `gemini-3-pro-image-preview` when the user asks for higher quality or the result from flash isn't good enough.

## Image configuration

```javascript
imageConfig: {
  imageSize: '2K',       // '1K', '2K', or '4K'
  aspectRatio: '16:9'    // see below
}
```

### Aspect ratios

| Ratio | Use case |
|-------|----------|
| `1:1` | Profile pictures, social posts |
| `16:9` | Presentations, desktop wallpapers |
| `9:16` | Phone wallpapers, stories |
| `4:3` | Standard photos |
| `3:2` | Photography prints |
| `3:4` | Portraits |
| `21:9` | Ultra-wide banners |

## Sending the generated image

After generating, send to the user:

```javascript
// Copy to IPC images directory
import { copyFileSync } from 'fs';
copyFileSync(OUTPUT, '/workspace/ipc/images/generated-image.png');
```

Then call `mcp__nanoclaw__send_image` with the path and a caption.

## Tips

- Be descriptive in prompts: include style, mood, lighting, composition
- For logos/icons: specify "clean vector style, simple, white background"
- For photos: specify "photorealistic, high detail, professional photography"
- For art: specify the style ("watercolor", "oil painting", "digital art", "anime")
- For editing: be specific about what to change and what to keep
- Always save to `/workspace/group/` first, then copy to `/workspace/ipc/images/` for sending
