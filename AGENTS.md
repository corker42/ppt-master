# AGENTS.md

This file is the project entry point for general AI agents.

**You MUST read [skills/ppt-master/SKILL.md](skills/ppt-master/SKILL.md) before any PPT generation task or repo modification.** This repository exists to generate presentations; SKILL.md is the authoritative workflow that owns project creation, role switching, serial execution, quality gates, post-processing, export, and every per-step command. The rest of this file only points to where related material lives — it never substitutes for SKILL.md.

## Project Overview

PPT Master is an AI-driven presentation generation system. Multi-role collaboration (Strategist → Image_Generator → Executor) converts source documents (PDF/DOCX/URL/Markdown) into natively editable PPTX with real PowerPoint shapes (DrawingML).

**Core Pipeline**: Source Document → Create Project → [Template] → Strategist Eight Confirmations → [Image_Generator] → Executor Live Preview → Quality Check → Post-processing → Export PPTX

> Topic-only requests with no source material: run the standalone [	opic-research](skills/ppt-master/workflows/topic-research.md) workflow before SKILL.md Step 1 to gather web materials.
>
> Template fill: when the user provides an existing .pptx template plus text materials or a topic and asks to reuse the original PPT design or fill content back into it (for example, "fill this deck with the new content", "fill this back into the template", or "reuse this deck's design"), run the standalone [	emplate-fill-pptx](skills/ppt-master/workflows/template-fill-pptx.md) workflow. This route edits PPTX directly and must not enter the SVG generation pipeline.
>
> Phase B resumption (split-mode execution): when the user opens a fresh chat and says "继续生成 projects/<x>" or similar, run the standalone [
esume-execute](skills/ppt-master/workflows/resume-execute.md) workflow to enter Phase B (SVG generation + export) without re-running Phase A.
>
> Decks containing data charts: run the standalone [erify-charts](skills/ppt-master/workflows/verify-charts.md) workflow between the executor and post-processing steps to calibrate chart coordinates.
>
> Recorded narration / video export: run the standalone [generate-audio](skills/ppt-master/workflows/generate-audio.md) workflow after post-processing.
>
> Object-level animation tuning: when the user asks to change animation order, effect, timing, or a specific object's reveal behavior, run the standalone [customize-animations](skills/ppt-master/workflows/customize-animations.md) workflow. Default export already has global animations; do not create nimations.json unless customization was requested.
>
> Live preview: any time the user mentions "live preview", "preview", "看效果", or wants to click/select a slide element, run [live-preview](skills/ppt-master/workflows/live-preview.md). Step 6 auto-starts it during generation; the workflow covers post-export re-entry and applying submitted annotations.
>
> Brand identity setup: when the user asks to "set up brand" / "建立品牌" / "做品牌规范", provides a brand asset (logo / brand site URL / branded PPTX / brand PDF), or wants to extract a brand from existing materials, run the standalone [create-brand](skills/ppt-master/workflows/create-brand.md) workflow. Output goes to skills/ppt-master/templates/brands/<id>/. Brands apply at SKILL.md Step 3 via the same explicit-path rule as layout templates — the user supplies the brand directory path to apply it; bare brand names never trigger.
>
> Visual self-check: only when the user explicitly requests a per-page visual review on the generated SVGs (e.g., "跑一下视觉自检 / 视觉回看 / 视觉 rubric", "visual review", "check each page visually"), run the standalone [isual-review](skills/ppt-master/workflows/visual-review.md) workflow between the executor and post-processing steps. The main pipeline does NOT invoke it automatically; do not infer or recommend it from deck size, model identity, or any other signal — user request is the only trigger.

## Structure Tree

```
G:\ppt-master-main\
├── .claude-plugin/marketplace.json    # Skill marketplace manifest (v2.7.0)
├── .github/workflows/deploy-pages.yml  # Only CI: GitHub Pages deploy
├── AGENTS.md                            # ← this file, entry for general AI agents
├── CLAUDE.md                            # Entry for Claude Code (same content)
├── README.md                            # User-facing project readme
├── README_CN.md                         # Chinese readme
├── docs/
│   ├── faq.md, getting-started.md, why-ppt-master.md, etc.
│   ├── rules/
│   │   ├── code-style.md             # Python conventions (zero tests, sys.path inject)
│   │   └── prompt-style.md           # Reference doc style guide
│   └── zh/                            # Chinese translations
├── examples/                           # 23 example projects
├── projects/                           # User project workspace (empty by default)
├── skills/ppt-master/
│   ├── SKILL.md                        # *** THE WORKFLOW AUTHORITY ***
│   ├── references/                     # Role defs + tech specs (20 files)
│   ├── scripts/                        # 124 Python files (28 entry-points)
│   │   ├── source_to_md/               # PDF/DOCX/XLSX/PPTX/Web → Markdown
│   │   ├── image_backends/             # 17 AI image generation providers
│   │   ├── tts_backends/              # 5 TTS backends (Edge/ElevenLabs/MiniMax/Qwen/CosyVoice)
│   │   ├── svg_to_pptx/               # PPTX export engine (DrawingML)
│   │   ├── pptx_to_svg/               # PPTX → SVG reverse conversion
│   │   ├── template_fill_pptx/        # Template fill engine (direct PPTX)
│   │   ├── image_sources/             # Web image search providers
│   │   ├── svg_editor/                # Live preview HTTP server
│   │   ├── svg_finalize/              # Post-processing (icon embed, crop, flatten)
│   │   ├── template_import/           # Template manifest extraction
│   │   └── docs/                      # Tool-specific documentation
│   ├── workflows/                      # 10 standalone workflow files
│   └── templates/
│       ├── brands/                     # 2 brand presets (anthropic, google)
│       ├── layouts/                    # 8 layout templates (academic_defense, ai_ops, etc.)
│       ├── charts/                     # 74 SVG chart templates
│       └── icons/                      # 6 icon sets (tabler-outline, phosphor-duotone, etc.)
```

## WHERE TO LOOK

| Task | Look in |
|------|---------|
| PPT generation workflow | skills/ppt-master/SKILL.md — start here for anything PPT-related |
| Role definitions (Strategist, Executor, Image_X) | skills/ppt-master/references/ |
| SVG/PPT technical constraints | skills/ppt-master/references/shared-standards.md |
| Canvas sizes (16:9, 4:3, Xiaohongshu, etc.) | skills/ppt-master/references/canvas-formats.md |
| Available chart templates (74 SVGs) | skills/ppt-master/templates/charts/charts_index.json |
| Available layout templates (8 layouts) | skills/ppt-master/templates/layouts/layouts_index.json |
| Available brand presets (2 brands) | skills/ppt-master/templates/brands/brands_index.json |
| Icon library (6 sets) | skills/ppt-master/templates/icons/README.md |
| Python code conventions | docs/rules/code-style.md |
| Reference doc writing style | docs/rules/prompt-style.md |
| Script documentation | skills/ppt-master/scripts/docs/ |
| User documentation | docs/ (getting-started.md, faq.md, etc.) |
| Example projects | examples/ (23 decks) |
| Project workspace | projects/ (user creates subdirs here) |
| Design spec reference | skills/ppt-master/templates/design_spec_reference.md |
| Spec lock reference | skills/ppt-master/templates/spec_lock_reference.md |
| Chart style guide | skills/ppt-master/templates/charts/CHART_STYLE_GUIDE.md |
| Image generation backends | skills/ppt-master/scripts/image_gen.py --list-backends |

## CODE MAP

**Key entry-point scripts** (28 top-level .py in scripts/):

| Script | Key symbol | Purpose |
|--------|-----------|---------|
| project_manager.py | class ProjectManager | init, import-sources --move, alidate |
| image_gen.py | main() | AI image generation (17 backends: openai, gemini, qwen, zhipu, volcengine, stability, bfl, ideogram, minimax, siliconflow, fal, replicate, openrouter, modelscope) |
| image_search.py | main(argv) | Web image search (pexels, pixabay, openverse, wikimedia) |
| svg_to_pptx.py | pptx_cli.main() | SVG → PPTX DrawingML converter |
| inalize_svg.py | main() | SVG post-processing (icon embed, crop, text flatten, rect→path) |
| 	otal_md_split.py | main() | Split speaker notes into per-page files |
| svg_quality_checker.py | class SVGQualityChecker | Validates SVGs vs spec_lock |
| nimation_config.py | main(argv) | Object-level animation scaffold/validate |
| 
otes_to_audio.py | class AudioBackend | TTS narration (edge, elevenlabs, minimax, qwen, cosyvoice) |
| latex_render.py | main(argv) | LaTeX formula → PNG (fallback chain: codecogs→quicklatex→mathpad→wikimedia) |
| nalyze_images.py | main() | Image dimension/format detection |
| config.py | class Config | Central config: get_canvas_format(), get_color_scheme() |
| svg_editor/server.py | main() | Browser live preview (http://localhost:5050) |
| pptx_template_import.py | main() | Extract template manifest from PPTX |
| 	emplate_fill_pptx.py | cli.main() | Direct PPTX fill (no SVG pipeline) |
| update_spec.py | main() | Propagate spec_lock color/font changes across SVGs |
| svg_position_calculator.py | class BarChartCalculator, PieChartCalculator, LineChartCalculator, RadarChartCalculator, GridLayoutCalculator, SVGPositionValidator | Chart coordinate calibration |
| pptx_to_svg/converter.py | class ConvertOptions, ConvertResult | PPTX → SVG reverse converter |
| svg_to_pptx/drawingml_converter.py | drawingml_converter | Core SVG → DrawingML conversion |

**Shared helpers**:
- image_backends/backend_common.py — HTTP download, retry, format detection
- image_sources/provider_common.py — class AssetCandidate, class ImageSearchRequest, license classification
- error_helper.py — class ErrorHelper, user-facing error templates
- pptx_to_svg/emu_units.py — class Xfrm, EMU ↔ pt conversion

## CONVENTIONS

- **Zero tests by design** — NO 	ests/, 	est_*.py, pytest/unittest imports. See docs/rules/code-style.md §11. Use inline python3 -c "..." smoke runs against projects/ samples instead.
- **No linting/formatting CI** — Only deploy-pages.yml (GitHub Pages). No ruff, black, mypy in CI. Local hooks may exist but are not enforced.
- **sys.path injection** — scripts/ is NOT a pip package. Entry-points inject scripts/ dir via Path(__file__).resolve().parent before sibling imports. Annotate post-injection imports with # noqa: E402.
- **Shebang** — Every .py starts with #!/usr/bin/env python3.
- **main() → int** — def main(argv: Optional[list[str]] = None) -> int:, called via 
aise SystemExit(main()).
- **stderr/stdout** — Progress/log to stderr; primary output to stdout.
- **@dataclass over pydantic** — Use @dataclass for value types. No pydantic, no attrs.
- **Lazy imports for SDKs** — Provider SDKs (google-genai, openai, etc.) imported inside the function that uses them, soft-fail with ImportError → RuntimeError with install instructions.
- **Markdown language consistency** — workflows/, 
eferences/, docs/ dirs are single-language per directory. No mixing English + Chinese scaffolding in one file. Chat replies are unaffected.
- **Bilingual docs** — docs/ is English, docs/zh/ is Chinese.
- **File header** — Every script has module docstring with name, purpose, Usage, Examples, Dependencies.
- **CLAUDE.md = AGENTS.md** — Both files carry identical content; CLAUDE.md is the Claude Code entry, AGENTS.md is for general AI agents.

## ANTI-PATTERNS

| Anti-pattern | Context |
|---|---|
| **Script-generated SVG** | SVG MUST be hand-written by main agent. A script that loops pages and emits SVGs in batch was tried and abandoned. FORBIDDEN (SKILL.md rule 9) |
| **Sub-agent SVG generation** | Executor Step 6 SVG generation is context-dependent, MUST stay in main agent. FORBIDDEN (rule 6) |
| **Batch page generation** | SVGs MUST be generated one page at a time, sequentially. FORBIDDEN (rule 7) |
| **Cross-phase bundling** | E.g., writing SVG code during Strategist phase. FORBIDDEN (rules 3/5) |
| **Combining post-processing steps** | 	otal_md_split.py + inalize_svg.py + svg_to_pptx.py MUST run ONE AT A TIME. NEVER combine into a single code block (Step 7) |
| **cp instead of finalize_svg.py** | finalize does icon embedding, image crop, text flatten, rect→path. cp breaks output. NEVER substitute (Step 7.2) |
| **--only flag on svg_to_pptx.py** | Suppresses one output file. NEVER use (Step 7.3) |
| **Template name fuzzy-matching** | Bare template names ("academic_defense") do NOT trigger Step 3. User must supply explicit path. NEVER resolve a name to a path (Step 3) |
| **Automated visual review** | isual-review workflow is opt-in only. Do NOT run by default or recommend based on model/deck size (Step 6 note) |
| **Creating animations.json without request** | Default export already has global entrance animations. Do NOT create unless user asks for object-level customization |
| **Reading image files directly** | NEVER read .jpg/.png with read_file or open(). Use analyze_images.py output or Design Spec Image Resource List (Step 4) |
| **Tests/ directories** | FORBIDDEN by code-style.md §11. No test files, no test dirs |
| **Bare except:** | Hard rule: always name the exception class |
| **Duplicating shared helper logic** | If helper exists in backend_common.py or provider_common.py, extend it; do NOT fork |
| **In-pipeline image gen without manifest** | Pipeline AI image gen MUST use --manifest mode. Positional form is for out-of-pipeline debug only |

## COMMANDS

`ash
# Source → Markdown
python3 skills/ppt-master/scripts/source_to_md/pdf_to_md.py <PDF>
python3 skills/ppt-master/scripts/source_to_md/doc_to_md.py <DOCX>
python3 skills/ppt-master/scripts/source_to_md/excel_to_md.py <XLSX>
python3 skills/ppt-master/scripts/source_to_md/ppt_to_md.py <PPTX>
python3 skills/ppt-master/scripts/source_to_md/web_to_md.py <URL>

# Project management
python3 skills/ppt-master/scripts/project_manager.py init <name> --format ppt169
python3 skills/ppt-master/scripts/project_manager.py import-sources <path> <files...> --move
python3 skills/ppt-master/scripts/project_manager.py validate <path>

# Image & formula
python3 skills/ppt-master/scripts/analyze_images.py <path>/images
python3 skills/ppt-master/scripts/latex_render.py <path>
python3 skills/ppt-master/scripts/image_gen.py --manifest <path>/images/image_prompts.json
python3 skills/ppt-master/scripts/image_search.py <query> -o <path>/images

# SVG quality check
python3 skills/ppt-master/scripts/svg_quality_checker.py <path>

# Post-processing (sequential, ONE AT A TIME)
python3 skills/ppt-master/scripts/total_md_split.py <path>
python3 skills/ppt-master/scripts/finalize_svg.py <path>
python3 skills/ppt-master/scripts/svg_to_pptx.py <path>

# Live preview
python3 skills/ppt-master/scripts/svg_editor/server.py <path> --live

# Template fill (direct PPTX, no SVG)
python3 skills/ppt-master/scripts/template_fill_pptx.py analyze <source.pptx> -o analysis/slide_library.json
python3 skills/ppt-master/scripts/template_fill_pptx.py apply <source.pptx> <fill_plan.json> -o exports/filled.pptx

# Audio narration
python3 skills/ppt-master/scripts/notes_to_audio.py <path> --voice <voice>

# Spec propagation
python3 skills/ppt-master/scripts/update_spec.py <path> --color primary #FF0000

# Repository update
python3 skills/ppt-master/scripts/update_repo.py
```

## Compatibility Boundary

- This repository is a workflow/skill package, not an app or service scaffold.
- Do NOT assume generic-project conventions like .worktrees/, 	ests/, or mandatory branch setup unless the user explicitly requests them.
- On conflict with a generic coding skill, prioritize [skills/ppt-master/SKILL.md](skills/ppt-master/SKILL.md) inside this repository.
