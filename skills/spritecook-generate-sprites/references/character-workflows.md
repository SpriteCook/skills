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

## New skin: `reskin_character`

Gives an animated pixel character a new look without animating it again. SpriteCook repaints the first frame of each facing's idle and rebuilds every frame of every animation from it, so the moves, frame counts and timing stay the same. Only offered when the server has New skin on; if the tool is missing, it is not available yet.

Use it for enemy variants (a frost boar, a golden boar), team or faction colours, outfits and armour on the same body, materials (stone, gold, ice), and day or season versions. It works best on recolours, materials and clothes that keep the outline. It cannot change the shape: wings, a tail, a bigger weapon or a different body need a new character from `generate_character` instead.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `character_id` | string (required) | - | Base character asset id that already has animations |
| `description` | string | null | The new look, up to 300 characters, for example "a frost boar with icy blue fur and white tusks" |
| `animate_rest` | bool | true | Also animate the ones that stay out again from the new character, at the normal animation price |
| `confirm` | bool | false | `false` runs only the free check; `true` with a description starts the paid paints |
| `wait_seconds` | int | 0 | Optional bounded wait from 0-90 seconds |

Workflow:

1. Call `reskin_character(character_id)` without `confirm`. The free check returns `status="planning"`; poll `check_character_skin(skin_id)` until it is `planned` (seconds to a minute).
2. Show the user what the check found: `new_look` (follow the repaint directly), `new_look_with_key_frames` (frames that move too far, such as a jump's airborne frames, follow extra key frames painted on one sheet per facing), and `stay_out` with the reason. With `animate_rest=true` the ones that stay out are animated again from the new character; `price.animate_rest_credits` is that extra cost. Show `price.credits` (one paint per facing plus one per key frame sheet). Paints that fail the outline check are refunded and painted once more.
3. Only after the user agrees, call `reskin_character(character_id, description=..., animate_rest=..., confirm=true)`.
4. Poll `check_character_skin(skin_id)` until `ready` or `failed`. Ready returns `new_character_id`, the new character in `character`, and the new animations in `assets` with `sprite_url` and `spritesheet_url`. When `animate_rest` was asked, `animate_rest.poll` points at `check_character_animation_run` for those animations. A failure carries a short `error`, and its credits are refunded.

Write the description as the whole new look, not a change list ("a knight in red and gold armour", not "make the armour red"). Do not ask for new parts sticking out.

## `check_character_skin`

Check a skin from `reskin_character` or `reskin_animation`: `status` (`planning`, `planned`, `painting`, `applying`, `ready`, `failed`), progress per paint in `paints`, results when ready, `credits_used` and `refunded_credits`.
