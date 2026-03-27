# SpriteCook Agent Skills

Agent skills for [SpriteCook](https://spritecook.ai) - AI-powered game asset generation. These skills teach AI coding agents how to generate pixel art and detailed/HD game art, including sprites, characters, items, tilesets, short animations, and more using SpriteCook's MCP tools.

Skills follow the open [Agent Skills](https://agentskills.io/) standard and work across all compatible editors.

## Available Skills

### spritecook-generate-sprites

Workflow guide for SpriteCook game art generation. Teaches the agent to:

- Generate production-ready pixel art or detailed/HD game art (characters, items, tilesets, UI, backgrounds)
- Use a short preflight workflow: check credits, prefer presigned download URLs, and save reusable asset IDs in a local manifest
- Import a source image through `POST /v1/api/assets/import`, receive an `asset_id`, and animate that asset instead of sending large base64 blobs through MCP
- Animate existing SpriteCook assets with motion-specific prompts while respecting source size and mode rules
- Persist important generated/imported `asset_id` values in a local project manifest so the agent can later animate, edit, or reuse them as references without asking the user to look them up
- Treat animation as a longer-running step than sprite generation, and keep `job_id` references so background-capable agents can continue other work while polling later
- Choose the right art style (pixel vs detailed) based on the game's needs
- Maintain visual consistency across assets using style references
- Download and save generated assets into your project while also keeping the matching SpriteCook `asset_id` for later reuse
- Autonomously identify and create all assets needed for a game

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

You need the SpriteCook MCP server connected to your editor. The skill tells your AI agent *how* to use SpriteCook, but the MCP connection provides the actual tools.

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
