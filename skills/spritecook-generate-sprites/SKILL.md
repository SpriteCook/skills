---
name: spritecook-generate-sprites
description: Generate sprites and game assets using the SpriteCook MCP tools. Use when building games and the user needs sprites, pixel art, characters, items, tilesets, textures, icons, or UI elements. Also use when asked to create, generate, or make game art assets.
---

# SpriteCook - AI Game Asset Generator

Use SpriteCook MCP tools when the user needs pixel art, detailed sprites, game assets, icons, tilesets, textures, or UI elements for a game project. SpriteCook generates production-ready game art from text prompts.

**Requires:** SpriteCook MCP server connected to your editor. Set up with `npx spritecook-mcp setup` or see [spritecook.ai](https://spritecook.ai).

## When to Use

- User asks to generate, create, or make sprites, pixel art, game assets, or icons
- User is building a game and needs visual assets (characters, items, environments, UI)
- User references sprites, pixel art, or game art in their request
- A game project needs consistent visual assets across multiple types

## Available Tools

### generate_pixel_art

Generate game art assets from a text prompt. Waits up to 90s for result, returns download URLs.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `prompt` | string (required) | - | What to generate. Be specific: subject, pose, view angle |
| `width` | int | 64 | Width in pixels (16-512) |
| `height` | int | 64 | Height in pixels (16-512) |
| `variations` | int | 1 | Number of variations (1-4) |
| `pixel` | bool | true | Pixel art (true) or detailed/HD art (false) |
| `pixel_perfect` | bool | true | Grid-aligned pixel post-processing |
| `bg_mode` | string | "transparent" | "transparent", "white", or "include" |
| `theme` | string | null | Art theme context, e.g. "dark fantasy medieval" |
| `style` | string | null | Style direction, e.g. "16-bit SNES style" |
| `aspect_ratio` | string | "1:1" | "1:1", "16:9", or "9:16" |
| `smart_crop` | bool | true | Auto-crop to content bounds (see note below) |
| `smart_crop_mode` | string | "tightest" | "tightest" (smallest fit) or "power_of_2" (32, 64, 128...) |
| `mode` | string | "assets" | "assets", "texture", or "ui" |
| `resolution` | string | "1K" | "1K", "2K", or "4K" |
| `colors` | string[] | null | Hex color palette, max 8 (e.g. ["#2D1B2E", "#FF6B35"]) |
| `reference_asset_id` | string | null | Asset ID from a previous generation to use as style reference |
| `edit_asset_id` | string | null | Asset ID to edit/modify with the new prompt |

`reference_asset_id` and `edit_asset_id` are mutually exclusive. The referenced asset must belong to your account. Use `reference_asset_id` to maintain visual consistency across assets.

**Note on output dimensions:** `pixel_perfect` processing ensures clean pixel art, but the final sprite content may not fill the exact requested canvas size. When `smart_crop` is enabled, the output is trimmed to fit the content. In `"tightest"` mode (default), a 64x64 request might return a 54x54 image. In `"power_of_2"` mode, it snaps to the nearest power of 2 (32, 64, 128...), so 64x64 stays 64x64.

### check_job_status

Check progress and get results of a generation job by `job_id`. Returns asset download URLs when complete.

### get_credit_balance

Check remaining credits, subscription tier, and concurrent job limit on the connected account. Use this before batch generating to know how many credits you have and how many jobs you can run at once.

### download_asset

Download a generated asset as base64-encoded PNG. **Always use this tool to save assets** instead of manually constructing HTTP requests with `curl` or `Invoke-WebRequest`.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `asset_id` | string (required) | - | The asset ID from a generation result |
| `format` | string | "pixel" | "pixel" (pixel-processed) or "raw" (original HD render) |

Returns `base64_png` (a base64-encoded string) and a suggested `filename`. The response also includes `save_instructions` with ready-to-use commands.

**How to save the file:** Take the `base64_png` string, decode it from base64 to raw bytes, and write those bytes to a `.png` file. Do NOT try to analyze or interpret the bytes - just decode and save.

**PowerShell:**
```powershell
[IO.File]::WriteAllBytes("assets/$filename", [Convert]::FromBase64String($base64_png))
```

**Bash / macOS:**
```bash
echo "$base64_png" | base64 -d > "assets/$filename"
```

**Python:**
```python
import base64
open(f"assets/{filename}", "wb").write(base64.b64decode(base64_png))
```

## Autonomous Workflow

When a user is building a game, proactively identify and generate needed assets. Follow this pattern:

1. **Check credits** with `get_credit_balance` before generating. Note the `concurrent_jobs` field - this is how many jobs you can run at the same time. Generate assets sequentially (one at a time) unless your limit allows parallel generation.
2. **Identify required assets** from the game description (characters, items, tiles, UI, icons)
3. **Generate a hero asset first** (e.g. the main character) to establish the art style
4. **Use `reference_asset_id`** with the hero asset's ID for subsequent generations to maintain visual consistency
5. **Download the assets** using `download_asset(asset_id=...)` and save the decoded PNG files into the project's asset directory
6. **Reference the saved files** in the game code

Example: User says "make a fishing game" -> check credits -> generate the player character first -> use that asset ID as `reference_asset_id` for fish, rod, boat, etc. -> all assets share the same art style -> download and place in project.

Use `edit_asset_id` when the user wants to iterate on a specific asset (e.g. "make the sword larger" or "change the color of the helmet").

If you receive a 429 error (concurrent job limit), wait for the active job to complete before submitting the next one.

## Asset Type Tips

| Asset Type | Recommended Settings |
|------------|---------------------|
| Characters | width: 64-128, bg_mode: "transparent", pixel: true |
| Items/Icons | width: 32-64, bg_mode: "transparent", pixel: true |
| Tilesets | width: 128-256, mode: "texture", bg_mode: "include" |
| UI Elements | width: 64-256, mode: "ui", bg_mode: "transparent" |
| Backgrounds | width: 256-512, aspect_ratio: "16:9", bg_mode: "include" |

- Be specific in prompts: "a red dragon breathing fire, side view, single sprite" beats "dragon"
- Use `reference_asset_id` to keep consistent art style across all assets in a project
- Use `theme` and `colors` for additional style and palette consistency
- Set `variations: 2-3` when you want options to pick from
- Use `edit_asset_id` to refine or modify existing assets
