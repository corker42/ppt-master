# AGENTS.md — svg_finalize (SVG Post-Processing Pipeline)

## Purpose

Unified SVG post-processing: icon embedding, image crop/embed, text flattening, rounded-rect to path. Step 7.2 of the 7-step pipeline.

**Two consumers:** (1) on-disk `svg_output/ → svg_final/`, (2) in-memory reuse by `svg_to_pptx` (any module deletion breaks native PPTX).

## Files & Role

| File | Step | What it does |
|------|------|-------------|
| `__init__.py` | — | Docstring only (dual-consumer note). No `main()`. |
| `embed_icons.py` | 1/4 | `<use data-icon="library/name"/>` → `<g transform="...">` with paths from `templates/icons/<library>/`. Respects fill, stroke-width, scale. Supports chunk-filled, tabler-filled, tabler-outline, phosphor-duotone, simple-icons. |
| `align_embed_images.py` | 2/4 | **Merged step** replacing crop_images + fix_image_aspect + embed_images. Per `<image>`: `slice` → crop bitmap in memory, `meet` → fit-box, embed as base64. Skips EMF/WMF (native PPTX passthrough). Single SVG parse/write cycle. |
| `crop_images.py` | — | Legacy — standalone crop (requires Pillow). Superseded by `align_embed_images.py`. |
| `fix_image_aspect.py` | — | Legacy — standalone fix-aspect. Superseded by `align_embed_images.py`. |
| `embed_images.py` | — | Legacy — standalone base64 embed. Superseded by `align_embed_images.py`. |
| `flatten_tspan.py` | 3/4 | Collapses nested `<tspan>` to flat `<text>`/`<tspan>`. Handles attr inheritance, dy stacking, whitespace collapse. |
| `svg_rect_to_path.py` | 4/4 | `<rect rx="N" ry="N">` → `<path d="...">` with elliptical arc corners (PowerPoint loses rx/ry on "Convert to Shape"). |

## CLI

```bash
python3 scripts/finalize_svg.py <project_path> [--only step1 step2 ...] [--compress] [--max-dimension N]
```

Default: all 4 steps. `--only` accepts: `embed-icons`, `align-images`, `flatten-text`, `fix-rounded`. Aliases `crop-images`/`fix-aspect`/`embed-images` all map to `align-images`.

| Old name | Maps to |
|----------|---------|
| `crop-images` | `align-images` |
| `fix-aspect` | `align-images` |
| `embed-images` | `align-images` |

## Critical Rules

- **NEVER run individual modules as CLI** — must use `finalize_svg.py` unified entry point.
- **NEVER skip embed_icons** — PPTX will have broken `<use>` refs.
- **NEVER skip align_images (embed sub-step)** — PPTX will have external image refs that cannot resolve inside a pptx-internal SVG.
- **NEVER `cp` instead of finalize** — cp skips icon embed, image crop, text flatten, rect→path.
- **Icons MUST have `fill="#HEX"`**. Stroke-style libs (tabler-outline) need explicit `stroke-width` from spec_lock.
- **One stylistic icon library per deck** (chunk-filled/tabler-filled/tabler-outline/phosphor-duotone).
- **simple-icons** ONLY for real brand logos, never generic icons.
- **Dual-consumer awareness**: These modules are memory-reused by `svg_to_pptx`. Deleting/renaming a module here may also break native pptx conversion silently.

## Pipeline Position (Step 7.2)

`total_md_split.py <path>` → **`finalize_svg.py <path>`** → `svg_to_pptx.py <path>`

Run ONE AT A TIME, never combined.