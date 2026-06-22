# AGENTS.md --- ppt-master Skill Directory

**This directory IS the skill.** SKILL.md (566 lines) is the absolute workflow authority. Do NOT substitute generic coding skills for it -- on conflict, SKILL.md wins unless user explicitly overrides.

## Entry Points

| What | File |
|---|---|
| Master workflow (7-step pipeline) | `SKILL.md` |
| Role definitions (load before each phase) | `references/strategist.md`, `references/executor-base.md`, `references/executor-general.md`, `references/executor-consultant.md`, `references/executor-consultant-top.md` |
| Tech constraints | `references/shared-standards.md`, `references/canvas-formats.md`, `references/animations.md`, `references/svg-image-embedding.md` |
| Image framework | `references/image-base.md`, `references/image-generator.md`, `references/image-searcher.md` |
| Template spec formats | `templates/design_spec_reference.md`, `templates/spec_lock_reference.md` |
| Standalone workflows | `workflows/` (10 files) |
| Script documentation | `scripts/README.md`, `scripts/docs/` |

## Pipeline (7 Steps, from SKILL.md)

1. **Source -> MD** -- `source_to_md/*.py` or read Markdown directly
2. **Project init** -- `project_manager.py init/import-sources`
3. **Template** -- ONLY explicit directory paths with `design_spec.md` (`kind: brand|layout|deck`) trigger. No fuzzy name resolution.
4. **Strategist** -- Eight Confirmations (BLOCKING). Output: `design_spec.md` + `spec_lock.md`
5. **Images** -- Conditional. AI gen via `image_gen.py --manifest`; web search via `image_search.py`
6. **Executor** -- SVGs one at a time (FORBIDDEN: batch, sub-agent, script-generated). Live preview auto-start.
7. **Post-processing** -- `total_md_split.py` -> `finalize_svg.py` -> `svg_to_pptx.py` (sequential, NEVER combined)

## Standalone Workflows (10)

| Workflow | Trigger |
|---|---|
| `topic-research.md` | Topic-only (no source files) |
| `template-fill-pptx.md` | User provides existing .pptx + content to fill back |
| `create-template.md` | User wants to create a new layout/deck template |
| `create-brand.md` | User wants to create/extract a brand identity |
| `resume-execute.md` | Continue generating projects/<x> in fresh chat (split mode) |
| `verify-charts.md` | Deck contains data charts (run between Step 6-7) |
| `visual-review.md` | User explicitly requests per-page visual check (OPT-IN ONLY) |
| `live-preview.md` | User mentions preview or wants to apply annotations |
| `customize-animations.md` | User asks to tune animation order/effect/timing |
| `generate-audio.md` | User asks for narrated/video export |

## Directory-Specific Anti-Patterns (hard failure)

| Anti-pattern | Rule |
|---|---|
| SVG written during Strategist phase | S3 No cross-phase bundling |
| SVG delegated to sub-agent | S6 Main-agent only |
| Pages generated in batches (5 at a time) | S7 Sequential, one per turn |
| Script-generated SVGs (loop+templating) | S9 Hand-write only |
| `cp` instead of `finalize_svg.py` | S7.2 finalize does icon embed + crop + flatten |
| `--only` flag on `svg_to_pptx.py` | Suppresses output |
| Template name fuzzy-matching | Step 3: path-only trigger |
| Running `notes_to_audio.py` directly without generate-audio workflow | Missing voice recommendation |
| Automated visual review | opt-in only (user must ask) |
| Creating `animations.json` without request | Default global animations exist |
| Reading image files directly | Use `analyze_images.py` or spec section VIII |
| Combining post-processing steps into one code block | Split into 3 sequential calls |

## Bilingual Convention

- `workflows/`, `references/`, `scripts/docs/` --- English
- Content values in `design_spec.md` may be in any language (match source)
- Template structure/field names always English
