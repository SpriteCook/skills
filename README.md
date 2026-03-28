# SpriteCook Agent Skills

Agent skills for [SpriteCook](https://spritecook.ai) - AI-powered game asset generation. These skills teach AI coding agents how to generate pixel art and detailed/HD game art, plus animate existing assets with SpriteCook's MCP tools.

Skills follow the open [Agent Skills](https://agentskills.io/) standard and work across all compatible editors.

## Available Skills

### spritecook-workflow-essentials

Shared workflow rules for SpriteCook:

- Check credits before larger runs
- Prefer presigned download URLs
- Save reusable `asset_id` values in a local manifest
- Use `smart_crop_mode="tightest"` by default
- Use `gemini-3.1-flash-image-preview` as the recommended default model

### spritecook-generate-sprites

Still-image generation guidance:

- Generate production-ready pixel art or detailed/HD assets
- Choose the right mode, model, and crop settings
- Keep style consistency with `reference_asset_id`
- Use `edit_asset_id` only when modifying an existing SpriteCook asset

### spritecook-animate-assets

Animation workflow guidance:

- Import local images first and animate by `asset_id`
- Use `edge_margin=6` by default for safer framing
- Write grounded, character-specific motion prompts
- Use `auto_enhance_prompt=true` when simple prompts like `Idle` or `Attack` are enough

## Installation

### Quick install (all compatible editors)

```bash
npx skills add spritecook/skills
```

### With SpriteCook CLI (also sets up MCP connection)

```bash
npx spritecook-mcp setup
```

## Prerequisites

You need the SpriteCook MCP server connected to your editor. The skill tells your AI agent how to use SpriteCook, but the MCP connection provides the actual tools.

**Set up MCP + skill in one step:**

```bash
npx spritecook-mcp setup
```

Or see [spritecook.ai](https://spritecook.ai) for manual setup instructions.

## Supported Editors

Works with any editor that supports the Agent Skills standard:

- Cursor
- VS Code (GitHub Copilot)
- Claude Code
- Claude Desktop
- Antigravity
- Codex
- Windsurf
- And more

## License

MIT
