---
name: spritecook-generate-sprites
description: Generate sprites and game assets using the SpriteCook MCP tools. Use when building games and the user needs sprites, pixel art, detailed art, characters, items, tilesets, textures, icons, or UI elements. Also use when asked to create, generate, or make game art assets.
---

# SpriteCook - AI Game Asset Generator

Use SpriteCook MCP tools when the user needs pixel art, detailed/HD sprites, game assets, icons, tilesets, textures, UI elements, or short image-to-animation game clips for a project. SpriteCook generates production-ready game art from text prompts in two styles: pixel art and detailed HD art, and can animate existing SpriteCook assets.

**Requires:** SpriteCook MCP server connected to your editor. Set up with `npx spritecook-mcp setup` or see [spritecook.ai](https://spritecook.ai).

## Credential Safety

- Never ask the user to paste a SpriteCook API key into chat, prompts, code blocks, shell commands, or generated files.
- Never print, persist, echo, or inline API keys or `Authorization` headers in agent output.
- Prefer SpriteCook MCP tools, presigned URLs, or a preconfigured local connector/helper that already handles authentication outside the prompt.
- If a raw API call is required and no authenticated connector/helper is available, stop and ask the user to configure one rather than improvising secret handling in the conversation.

## When to Use

- User asks to generate, create, or make sprites, pixel art, detailed art, game assets, or icons
- User asks to animate an existing sprite, prop, or game asset
- User is building a game and needs visual assets (characters, items, environments, UI)
- User references sprites, pixel art, HD art, or game art in their request
- A game project needs consistent visual assets across multiple types

## Available Tools

### generate_game_art

Generate game art assets from a text prompt. Supports both pixel art and detailed/HD styles. Waits up to 90s for result, returns download URLs.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `prompt` | string (required) | - | What to generate. Be specific: subject, pose, view angle |
| `width` | int | 64 | Width in pixels (16-512) |
| `height` | int | 64 | Height in pixels (16-512) |
| `variations` | int | 1 | Number of variations (1-4) |
| `pixel` | bool | true | True for pixel art, false for detailed/HD art |
| `bg_mode` | string | "transparent" | "transparent", "white", or "include" |
| `theme` | string | null | Art theme context, e.g. "dark fantasy medieval" |
| `style` | string | null | Style direction, e.g. "16-bit SNES style" |
| `aspect_ratio` | string | "1:1" | "1:1", "16:9", or "9:16" |
| `smart_crop` | bool | true | Auto-crop to content bounds (see note below) |
| `smart_crop_mode` | string | "tightest" | "tightest" (smallest fit) or "power_of_2" (32, 64, 128...) |
| `model` | string | null | "gemini-2.5-flash-image" (default) or "gemini-3-pro-image-preview" (higher quality, 2x credit cost) |
| `mode` | string | "assets" | "assets", "texture", or "ui" |
| `resolution` | string | "1K" | "1K", "2K", or "4K" |
| `colors` | string[] | null | Hex color palette, max 8 (e.g. ["#2D1B2E", "#FF6B35"]) |
| `reference_asset_id` | string | null | Asset ID from a previous generation to use as style reference |
| `edit_asset_id` | string | null | Asset ID to edit/modify with the new prompt |

`reference_asset_id` and `edit_asset_id` are mutually exclusive. The referenced asset must belong to your account. Use `reference_asset_id` to maintain visual consistency across assets.

**Note on output dimensions:** When `pixel=true`, pixel-perfect grid alignment is applied automatically for clean pixel art. The final sprite content may not fill the exact requested canvas size. When `smart_crop` is enabled, the output is trimmed to fit the content. In `"tightest"` mode (default), a 64x64 request might return a 54x54 image. In `"power_of_2"` mode, it snaps to the nearest power of 2 (32, 64, 128...), so 64x64 stays 64x64.

## Pixel Art vs Detailed Art

SpriteCook supports two art styles controlled by the `pixel` parameter:

**Pixel art** (`pixel: true`, default):
- Crisp hard edges, no anti-aliasing, visible pixel grid
- Automatic pixel-perfect post-processing for clean grid alignment
- Best for: retro games, indie games, 8-bit/16-bit style projects
- Typical sizes: 16-128px for sprites, up to 256px for tilesets

**Detailed/HD art** (`pixel: false`):
- Smooth gradients, fine detail, anti-aliased edges
- Higher fidelity output without pixel grid constraints
- Best for: HD 2D games, concept art, marketing assets, higher-resolution projects
- Typical sizes: 128-512px for sprites, larger for backgrounds

Choose based on the game's art direction. When the user doesn't specify, default to pixel art. If they mention "HD", "detailed", "high-res", "realistic", or "smooth", use `pixel: false`.

## Model Selection

Both pixel art and detailed mode default to `gemini-2.5-flash-image` (fast, good quality). You can optionally set the `model` parameter:

- **`gemini-2.5-flash-image`** (default): Fast generation, good quality, standard credit cost.
- **`gemini-3-pro-image-preview`**: Higher quality results with more detail and consistency, but costs 2x credits. Use when the user wants the best possible output or when standard results aren't meeting expectations.

### animate_game_art

Animate an existing SpriteCook asset into a short pixel-art or detailed animation. This tool expects `asset_id` and should not be treated as a raw base64 transport path. Waits up to 90s for the result, and falls back to returning a job id when the animation is still running.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `asset_id` | string (required) | - | Existing SpriteCook asset ID to animate. The asset must belong to the authenticated SpriteCook account |
| `prompt` | string (required) | - | Describe the exact motion over time, not just the asset |
| `pixel` | bool | null | Optional animation mode override. If omitted, SpriteCook infers the mode from the asset |
| `output_frames` | int | 8 | Even frame count. Pixel supports 2-16, detailed supports 2-24 |
| `output_format` | string | "webp" | "webp", "gif", or "spritesheet" |
| `negative_prompt` | string | null | Optional motion/content exclusions |
| `matte_color` | string | "#808080" | Hex matte color used before processing transparency |
| `removebg` | string | "Basic" | "None", "Basic", or "Pro" background removal before returning. Pro costs extra per frame |
| `colors` | int | 24 | Pixel palette size. Only valid when `pixel=true` |

Workflow for agents:

1. If the user already has a SpriteCook asset, pass its `asset_id` directly to `animate_game_art`.
2. If the user only has a local image file or data URL, import it through a preconfigured SpriteCook connector, local helper, or existing authenticated workflow that handles secrets outside the prompt and returns an `asset_id`.
3. Use the returned `asset_id` from that upload step in `animate_game_art`.
4. Keep `pixel` omitted unless you need to force a mode.

The upload step is the place where the source image is imported into SpriteCook. It is not the animation call itself.

Agent persistence guidance:

- When working inside a real game project or codebase, persist important SpriteCook `asset_id` values in a local project manifest so they can be reused later without asking the user to look them up in the SpriteCook UI.
- This is especially useful for:
  - assets you may animate later
  - assets you may use as `reference_asset_id`
  - assets you may use as `edit_asset_id`
  - hero assets that define the project style
- Prefer a simple machine-readable file in the workspace, for example `spritecook-assets.json`, unless the project already has an asset manifest.
- For each saved asset, include at least:
  - `asset_id`
  - local file path or saved filename
  - prompt
  - whether it is `pixel` or detailed
  - dimensions
  - a short purpose label such as `player_idle_source`, `enemy_slime_ref`, or `coin_ui_icon`
- Before generating a new reference asset or asking the user for an asset id, check the local manifest first.
- When the user request is one-off and not tied to a real project workspace, you do not need to create a manifest.

Source size rules:

- A SpriteCook import step creates the source asset from base64 image data or a data URL.
- The imported source should still respect the animation constraints the agent intends to use next.
- For animation, small source assets should stay in pixel mode.
- Pixel animation is the correct choice for assets up to `256x256`.
- Detailed animation is the correct choice for assets between `256x256` and `2048x2048`.
- Do not force a sub-256 source into detailed mode.

Import example via a preconfigured authenticated helper:

```http
POST /v1/api/assets/import
Content-Type: application/json

{
  "image": "data:image/png;base64,...",
  "pixel": true,
  "display_name": "duck"
}
```

Typical response fields to retain in the manifest:

```json
{
  "id": "7a9d3f31-...",
  "display_name": "duck",
  "file_name": "duck",
  "width": 128,
  "height": 128,
  "pixel_url": "https://..."
}
```

Animation prompts should be explicit about the motion arc. Good prompts describe:
- what moves
- how it moves
- direction and timing
- loop behavior
- what should stay stable

Good example: `subtle idle bob, cloak sways gently left and right, two-frame breathing in the chest, feet stay planted, clean seamless loop`

Weak example: `idle animation`

### check_job_status

Check progress and get results of a generation or animation job by `job_id`. Still-image jobs return `assets`. Animation jobs return `output` with the animation asset id, preview/download URL, spritesheet URL, content type, and metadata.

### get_credit_balance

Check remaining credits, subscription tier, and concurrent job limit on the connected account. Use this before batch generating to know how many credits you have and how many jobs you can run at once.

## Expected Timing

Use these as practical expectations when planning work:

- Average sprite generation time: about 20 seconds
- Average animation time: about 120 seconds

Animation is a long-running operation compared to still-image generation. If the client can continue working in the background, start the animation, keep the returned `job_id`, and come back with `check_job_status(job_id=...)` instead of blocking on it.

## Downloading Assets

Each generated asset in the response may include both authenticated API URLs and, when available, presigned download URLs.

Download preference for agents:

- Prefer `_presigned_pixel_url` when saving the pixel PNG
- Prefer `_presigned_url` when saving the raw/original image
- Avoid direct authenticated download endpoints in skill-driven workflows unless a preconfigured connector/helper handles auth out of band

If you try an authenticated download endpoint without the required out-of-band auth handling, you may save a JSON error body instead of an image file.

**PowerShell:**
```powershell
Invoke-WebRequest -Uri $presigned_pixel_url -OutFile "assets/sprite.png"
```

**Bash / macOS:**
```bash
curl -sL -o "assets/sprite.png" "$presigned_pixel_url"
```

Use the asset's `display_name` or `id` to create a meaningful filename. Presigned URLs are temporary (expire after about 1 hour), so download promptly after generation.

If a downloaded "image" is suspiciously tiny or fails to open, inspect the file contents before proceeding. It may be an auth error or JSON response that was saved with an image filename.

Assets remain saved in SpriteCook on the user's account, but when working on a real game project you should usually download them into the project immediately and keep a reference to the corresponding `asset_id`. That gives the project an immediate local copy while still preserving the SpriteCook asset for later animation, editing, and style reuse.

## Autonomous Workflow

When a user is building a game, proactively identify and generate needed assets. Follow this pattern:

1. **Check credits** with `get_credit_balance` before generating. Note the `concurrent_jobs` field - this is how many jobs you can run at the same time. Generate assets sequentially (one at a time) unless your limit allows parallel generation.
2. **Identify required assets** from the game description (characters, items, tiles, UI, icons)
3. **Decide pixel vs detailed** based on the game's art style or the user's preference
4. **Generate a hero asset first** (e.g. the main character) to establish the art style
5. **Use `reference_asset_id`** with the hero asset's ID for subsequent generations to maintain visual consistency
6. **Download the assets** from `_presigned_pixel_url` when available and save the PNG files into the project's asset directory
7. **Persist the returned asset ids** in a local manifest such as `spritecook-assets.json`
8. **Reference the saved files** in the game code
9. **Treat animation as a background step** when possible. Kick it off, keep the `job_id`, and continue with other project work until polling is needed.

For animation requests:

1. Use `animate_game_art` only with an existing SpriteCook `asset_id`
2. If you only have local image bytes, import them first through a preconfigured authenticated SpriteCook helper or connector
3. Check the source dimensions before sending, and choose pixel vs detailed mode accordingly
4. Prefer reusing an asset id from the local project manifest before asking the user for one
5. Write a motion-specific prompt that describes the action frame-to-frame
6. Include `removebg: "Basic"` by default when the user wants a clean transparent result
7. Use `removebg: "Pro"` only when the user explicitly wants the higher-quality paid cleanup
8. If the tool returns a queued/running job, follow up with `check_job_status(job_id=...)`

Example: User says "make a fishing game" -> check credits -> generate the player character first -> use that asset ID as `reference_asset_id` for fish, rod, boat, etc. -> all assets share the same art style -> download and place in project.

Use `edit_asset_id` when the user wants to iterate on a specific asset (e.g. "make the sword larger" or "change the color of the helmet").

If you receive a 429 error (concurrent job limit), wait for the active job to complete before submitting the next one.

## Asset Type Tips

| Asset Type | Recommended Settings |
|------------|---------------------|
| Pixel Characters | width: 64-128, bg_mode: "transparent", pixel: true |
| Pixel Items/Icons | width: 32-64, bg_mode: "transparent", pixel: true |
| Pixel Tilesets | width: 128-256, mode: "texture", bg_mode: "include", pixel: true |
| HD Characters | width: 256-512, bg_mode: "transparent", pixel: false |
| HD Items/Icons | width: 128-256, bg_mode: "transparent", pixel: false |
| HD Backgrounds | width: 512, aspect_ratio: "16:9", bg_mode: "include", pixel: false |
| UI Elements | width: 64-256, mode: "ui", bg_mode: "transparent" |
| Backgrounds | width: 256-512, aspect_ratio: "16:9", bg_mode: "include" |

- Be specific in prompts: "a red dragon breathing fire, side view, single sprite" beats "dragon"
- For animation prompts, describe the motion precisely: "torch flame flickers upward, slight left-right sway, loop cleanly" beats "torch animation"
- Use `reference_asset_id` to keep consistent art style across all assets in a project
- Use `theme` and `colors` for additional style and palette consistency
- Set `variations: 2-3` when you want options to pick from
- Use `edit_asset_id` to refine or modify existing assets
