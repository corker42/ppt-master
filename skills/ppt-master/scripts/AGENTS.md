# AGENTS.md — scripts/ Directory

**This directory is the tool layer.** 28 entry-point .py files + 9 subpackages with __init__.py (real packages). scripts/ itself has **no __init__.py** — scripts are invoked directly via `python3 scripts/xxx.py`.

## Rule of thumb

- **Top-level .py = CLI entry point.** Invoke via `python3 scripts/<name>.py args...`
- **Subpackage = importable library.** `from svg_to_pptx import convert_svg_to_slide_shapes`
- **Import helpers**, never fork them. Shared modules: `project_utils.py`, `error_helper.py`, `config.py`, `image_backends/backend_common.py`, `image_sources/provider_common.py`, `tts_backends/backend_common.py`

## Script Quick Reference

| Script | What it does | Invocation |
|--------|-------------|------------|
| project_manager.py | Init, import-sources, validate projects | `init/import-sources/validate/info` |
| image_gen.py | AI image gen (17 backends) | `--manifest` (pipeline) / positional (debug) / `--list-backends` |
| image_search.py | Web image search (pexels/pixabay/openverse/wikimedia) | query + `--filename` + `-o` |
| svg_to_pptx.py | Thin wrapper -> svg_to_pptx/ package | `python3 svg_to_pptx.py <path>` |
| template_fill_pptx.py | Thin wrapper -> template_fill_pptx/ package | `analyze / scaffold / check-plan / apply` |
| finalize_svg.py | SVG post-processing: embed-icons, align-images, flatten-text, fix-rounded | `<path> [--only step ...]` |
| total_md_split.py | Split total.md speaker notes into per-page .md | `<path>` |
| pptx_to_svg.py | PPTX -> SVG reverse converter (standalone CLI) | `<file.pptx> -o <dir>` |
| pptx_template_import.py | PPTX -> manifest.json + svg/ for template workflows | `<file.pptx> [--manifest-only]` |
| svg_quality_checker.py | Validate SVGs vs spec_lock + canvas + animation config | `<path>` |
| svg_position_calculator.py | Chart coordinate pre-calc + post-validation | `analyze/from-json/calc bar/pie/line/grid` |
| notes_to_audio.py | TTS narration (edge/elevenlabs/minimax/qwen/cosyvoice) | `<path> --voice <voice>` |
| animation_config.py | Scaffold/validate per-object animation sidecar | `scaffold/list-groups/validate <path>` |
| pptx_animations.py | Pure XML generation for transitions + entrance animations | `--demo / --list` |
| update_spec.py | Propagate spec_lock.md color/font changes to SVG files | `<path> colors.primary=#FF0000` |
| analyze_images.py | Image dimension/format detection | `<images_path> [--canvas ppt169]` |
| latex_render.py | LaTeX formula -> PNG (fallback chain) | `<path> [--manifest ...]` |
| visual_review.py | Render SVGs to PNGs via Playwright (Chromium) | `<path>` |
| rotate_images.py | Image orientation management | `gen/fix/auto <dir>` |
| check_annotations.py | Scan SVGs for data-edit-target / data-edit-annotation | `<path or .svg>` |
| batch_validate.py | Batch project structural validation | `examples / projects / --all` |
| gemini_watermark_remover.py | Remove Gemini watermark from images | `<image_path>` |
| register_template.py | Register brand/layout/deck into template index | `<id> --kind brand|layout|deck` |
| generate_examples_index.py | Auto-generate examples/README.md | `[examples_dir]` |
| update_repo.py | Git pull + pip sync if requirements.txt changed | `[--skip-pip]` |
| config.py | Central config: canvas formats, colors, paths (import only) | `from config import Config` |
| error_helper.py | User-facing error messages + fix suggestions (import only) | `ErrorHelper.get_solution(...)` |
| project_utils.py | Project info parsing, validation helpers (import only) | `validate_project_structure()` |

## Subpackages (9)

| Package | Modules | Role |
|---------|---------|------|
| svg_to_pptx/ | 14 | DrawingML converter, PPTX builder, CLI, media/notes/narration/animation |
| template_fill_pptx/ | 14 | OOXML analyzer, scaffolder, checker, text/table/chart fill, cli, applier |
| image_backends/ | 14 | backend_common.py + 13 provider impls (openai, gemini, qwen, stability, bfl, ideogram, minimax, zhipu, volcengine, siliconflow, fal, replicate, openrouter, modelscope) |
| image_sources/ | 5 | provider_common.py + 4 providers (pexels, pixabay, openverse, wikimedia) |
| svg_finalize/ | 7 | embed_icons, embed_images, crop_images, fix_aspect, flatten_tspan, rect_to_path, align_embed_images |
| pptx_to_svg/ | 2 | OOXML -> SVG reverse conversion (converter.py, emu_units.py) |
| svg_editor/ | 1 | Live preview HTTP server (port 5050) |
| tts_backends/ | 6 | backend_common.py + 5 backends (edge, elevenlabs, minimax, qwen, cosyvoice) |
| template_import/ | 1 | PPTX metadata extraction for /create-template |
| source_to_md/ | 5 | pdf_to_md, doc_to_md, excel_to_md, ppt_to_md, web_to_md |

## Convention Checklist

- **Shebang**: `#!/usr/bin/env python3` on every .py
- **main(argv) -> int**: `def main(argv: Optional[list[str]] = None) -> int:` + `raise SystemExit(main())`
- **sys.path injection**: `sys.path.insert(0, str(Path(__file__).resolve().parent))` before sibling imports (noqa: E402)
- **stderr/stdout**: progress/logs to stderr, primary output to stdout
- **Lazy SDKs**: heavy SDKs imported inside function, soft-fail with install instructions
- **Module docstring**: name + purpose + Usage + Examples + Dependencies
- **`@dataclass` over pydantic**: no pydantic, no attrs
- **Bare `except:`**: HARD FORBIDDEN -- always name the exception

## Anti-Patterns (hard failure)

| Anti-pattern | Why |
|---|---|
| Tests/ dirs, pytest imports | Forbidden by code-style.md S11 |
| Duplicating shared helpers | Extend backend_common.py, provider_common.py, config.py, error_helper.py, project_utils.py -- never fork |
| Reading .jpg/.png with open() | Use analyze_images.py output or Design Spec Image Resource List |
| sys.path injection without noqa: E402 | Linter fails on post-injection imports |

## Pipeline Order (Step 7)

NEVER combine into one code block. Run sequentially, ONE AT A TIME:

1. total_md_split.py <path>
2. finalize_svg.py <path> (NOT cp)
3. svg_to_pptx.py <path> (NOT --only)
