# AGENTS.md -- template_fill_pptx/ (Template Fill Engine)

**Direct OOXML editing -- no SVG round-trip.** Analyzes a PPTX as a reusable slide library (text slots / tables / charts), then fills new content from a JSON plan into cloned slides. Output is a native .pptx preserving original design, transitions, notes.

## Entry

Wrapper: scripts/template_fill_pptx.py (26 lines, thin CLI relay)
Package: 	emplate_fill_pptx/ (16 files, real package with __init__.py)
Invocation:
  python3 scripts/template_fill_pptx.py analyze <deck.pptx> -o slide_library.json
  python3 scripts/template_fill_pptx.py scaffold slide_library.json -o fill_plan.json
  python3 scripts/template_fill_pptx.py check-plan slide_library.json fill_plan.json
  python3 scripts/template_fill_pptx.py apply <deck.pptx> fill_plan.json -o output.pptx

## Pipeline (4 stages)

ANALYZE -> [SCAFFOLD] -> [CHECK-PLAN] -> APPLY

| Stage | Module | What it does |
|-------|--------|-------------|
| analyze | analyzer.py | Reads PPTX: enumerates slides, extracts text slots, tables (rows/cells), chart display caches, speaker notes, transitions. Output: slide_library.json |
| scaffold | scaffolder.py | Generates editable fill_plan.json skeleton from slide_library. Copies old_text->text fields. --slides for subset, --include-empty for empty slots |
| check-plan | checker.py | Validates fill plan against slide library capacity: text length vs slot char estimates, cell bounds vs table dimensions, series/category counts vs chart. Flags errors/warnings |
| apply | applier.py | Deep-clones source slides into new PPTX, runs text/table/chart fillers, injects transitions, rebuilds relationships/content-types. Writes timestamped .pptx |

## Key Files (16)

| File | Responsibility |
|------|---------------|
| cli.py | argparse: 4 subcommands (analyze/scaffold/check-plan/apply). Timestamps output paths |
| ooxml.py | Read-side primitives: namespaces, SlideRef, EMU->px, part/relationship resolution, container discovery (_text_containers, _table_containers, _chart_containers), _shape_identity, JSON I/O |
| analyzer.py | Slide library extraction. Classifies slides (THANKS_KEYWORDS, TOC_KEYWORDS, CHAPTER_KEYWORDS). Extracts text, tables, charts per slide |
| chart_read.py | Reads chart display cache (categories, series, values) from <c:ser> XML. No workbook parsing |
| scaffolder.py | Builds fill_plan.json skeleton. Defaults to first 6 slides |
| checker.py | 450-line capacity validation: text len vs char budget, row/col bounds, series/category counts |
| applier.py | Orchestrator: opens PPTX as ZipFile, clones slides, dispatches to text/table/chart/notes/transitions helpers |
| text_fill.py | Replaces <a:t> text nodes preserving paragraph/run formatting. Creates missing <a:p>/<a:r>/<a:t> if absent |
| table_fill.py | Edits table cell text by (row, col). Preserves table structure |
| chart_fill.py | Rewrites <c:ser> caches + rebuilds embedded .xlsx workbook. Styling/axes/legend untouched |
| clone.py | Deep-clones structured private parts (custom-data, SmartArt, per-slide themes). Shared by ref: layouts/masters/theme/media |
| notes.py | Builds notesSlide XML + rels. Reuses svg_to_pptx.pptx_notes (cross-package dep) |
| transitions.py | Injects <p:transition> from shared pptx_animations TRANSITIONS. Default: fade/0.5s |
| package.py | Write-side OOXML: content-type overrides, relationship elements, part-number allocation, pruning |
| selectors.py | Selector key builders: slot_id, shape_id, shape_name prefixes. Used by checker, text_fill, table_fill, chart_fill |

## Selector Pattern (CRITICAL)

All edits target shapes by 3 key types (tried in order):
  slot_id:s{slide}_sh{id}  -- analyzer-generated stable ID
  shape_id:{id}             -- PowerPoint cNvPr@id
  shape_name:{name}         -- PowerPoint cNvPr@name
Chart edits additionally match on chart_id:s{slide}_ch{id}.
Edits with optional: true silently skip unmatched shapes.

## Directory-Specific Conventions
- No SVG pipeline -- pure OOXML. No SVGs, no finalize_svg, no svg_to_pptx.
- Read-only on source PPTX during analyze. Write via ZipFile only during apply.
- Many-to-one slide mapping allowed: one source slide -> multiple output slides.
- Content-type management via _add_content_type_override in package.py.
- Timestamped output: _YYYYMMDD_HHMMSS appended unless stem already has one.
- Default transition: fade / 0.5s. Per-slide override via plan transition field.
- sys.path injection: parent scripts/ dir added before import (standard for repo).

## Anti-Patterns (hard failure)
| Anti-pattern | Why |
|---|---|
| Entering SVG pipeline | This is the non-SVG branch. Do NOT call finalize_svg, svg_to_pptx |
| Running without explicit user request | Only on "fill this deck" / "reuse template design" |
| Placeholder matching by position | Must use text-content matching |
| Creating fill plan manually | Always use scaffold subcommand |
| Editing table row/column structure | Table structure preserved. Cell text only |
| Skipping check-plan before apply | Catches overflows and mismatches |
| Image content insertion | TEXT/TABLE/CHART/NOTES only |
| Direct PPTX modification of source | Source is read-only |

