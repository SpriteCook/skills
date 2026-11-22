# SpriteCook Guided Characters

Read this when the user wants a base pixel-art character plus a matching set of animations through SpriteCook's guided character tools. Pair it with `spritecook-workflow-essentials` for credits, manifests, and safe downloads.

## `list_character_workflows`

List guided pixel-art character perspectives, default animation ids, source-view/prep requirements, frame counts, and credit estimates.

Perspectives and preset animations:

| Perspective | Defaults | Presets |
|-------------|----------|---------|
| `platformer` | `idle`, `walk`, `jump` | `idle`, `walk`, `jump`, `run`, `attack`, `hurt`, `death` |
| `isometric` | `idle`, `walk_down`, `walk_right` | `idle`, `idle_back`, `walk_down`, `walk_right`, `jump_back`, `jump_front`, `run_down`, `run_right`, `attack`, `hurt`, `death` |
| `topdown` | `idle`, `walk_up`, `walk_right`, `walk_down` | `idle`, `idle_back`, `idle_right`, `walk_up`, `walk_down`, `walk_right`, `attack`, `hurt`, `death` |

## `generate_character`

Generate a base pixel-art character with SpriteCook's recommended character settings: a 64x64 size hint, transparent background, 1K square, tight smart crop, and the perspective-specific character prompt rewrite. The final canvas is often 56-170 px, so check the returned size. Returns a job immediately by default; when complete, the first generated asset is the `character_id`. This tool has no `project_id`; move the result with `assign_assets_to_project` when it belongs in a project.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `prompt` | string (required) | - | Character description |
| `perspective` | string (required) | - | `platformer`, `isometric`, or `topdown` |
| `model` | string | null | Optional generation model. Call `list_generation_models` for current options and costs. |
| `quality` | string | "medium" | GPT-Image-2 quality tier: "low", "medium", or "high" |
| `wait_seconds` | int | 0 | Optional bounded wait from 0-90 seconds before returning the polling contract |

## `generate_character_animations`

Generate preset and/or custom animations for a base character asset. Returns a character-animation run immediately by default; follow the returned polling contract until canonical `assets` entries are ready.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `character_id` | string (required) | - | Base character asset id: a `generate_character` result or any owned character still, including an edited, uploaded, or sliced sprite |
| `perspective` | string (required) | - | Same perspective used for the base character |
| `animation_ids` | string[] | perspective defaults | Preset ids from `list_character_workflows` |
| `custom_animations` | object[] | null | Custom animations with `{ id, label, prompt, source_view, output_frames }` |
| `bg_removal_provider` | string | "basic" | `basic` or `photoroom` |
| `wait_seconds` | int | 0 | Optional bounded wait from 0-90 seconds before returning the polling contract |

Character animation runs use `pixel-engine-v1.1`; this tool has no `model` argument. For `pixel-engine-v1.5`, animate each motion separately with `animate_game_art` on the same still (see `spritecook-animate-assets`). Run outputs use their own canvas, which is usually larger than the base still and not aligned to it; match frame 0 against the still locally when the hand-off must not jump. Fast jumps and overhead swings can touch the canvas edge, so check those frames. Two identical calls sent together can return the same run; send one call per take you want.

Passing any owned still as `character_id` is often the best way to animate a character that matches an existing art style. Give uploaded or sliced assets a descriptive label first with `update_asset_label`, because the label can feed the automatic animation prompt.

Custom animations use the custom prompt as the final animation prompt and skip preset prompt enhancement. `source_view` defaults to `front_idle`; if another source view needs prep, SpriteCook uses the matching workflow prep dependency for that perspective. A `source_view` selects a supported workflow input, not an arbitrary new state such as curled-up sleep. For a custom state that needs a different starting pose, prepare the still with `generate_game_art` first, then use `animate_game_art` on that prepared asset.

Example custom animation:

```json
{
  "label": "Spin Attack",
  "prompt": "Spin in place with a quick sword slash, looping cleanly.",
  "source_view": "right_walk",
  "output_frames": 8
}
```

## `check_character_animation_run`

Check a guided character animation run by id. Returns run status, item statuses, canonical generated `assets`, prep state, failures, and credits.
