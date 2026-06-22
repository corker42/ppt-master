# AGENTS.md — svg_to_pptx (DrawingML Conversion Engine)

## Purpose

Converts hand-written SVGs (Executor Step 6 output) into **native editable PPTX** with real PowerPoint shapes (DrawingML). This is the final export step of the 7-step pipeline.

**Input:** svg_output/ (Executor's hand-written SVGs — NOT svg_final/)
**Output:** exports/<project>_<timestamp>.pptx (native), ackup/<timestamp>/svg_output/ (always)

## File Map

| File | Role |
|------|------|
| pptx_cli.py | CLI entry point: arg parsing, backup, dispatch to builder |
| pptx_builder.py | create_pptx_with_native_svg() — core assembly: slide creation, shape injection, rels, content types, zip packaging |
| drawingml_converter.py | convert_svg_to_slide_shapes() — SVG tree walk, element dispatch, animation anchor extraction, chrome detection |
| drawingml_context.py | ConvertContext — shared state (scale, translate, defs, id counter, media, rels, anim targets, trace events) |
| drawingml_elements.py | Per-element converters: rect, circle, ellipse, line, path, polygon, polyline, text, image, nested SVG |
| drawingml_paths.py | SVG path <d> to DrawingML :path: parse, normalize, arc-to-bezier, curve flattening |
| drawingml_styles.py | Fill (solid/gradient), stroke, effect/shadow XML builders |
| drawingml_utils.py | Coord helpers: EMU conversion, color parsing, font utilities, East-Asian font list, dash presets, transform matrix |
| pptx_slide_xml.py | create_slide_xml_with_svg() — slide XML skeleton + fallback PNG/SVG dual-format blip + transition XML |
| pptx_dimensions.py | Slide dimensions in EMU, SVG format detection from viewBox, project info |
| pptx_media.py | SVG->PNG rasterization (cairosvg > svglib) for Office compatibility fallback |
| pptx_discovery.py | ind_svg_files(), ind_notes_files() — discover SVG/notes in project dir |
| pptx_notes.py | markdown_to_plain_text() + create_notes_slide_xml() for speaker notes XML |
| pptx_narration.py | Audio embedding: find m4a/mp3/wav files, inject narration XML, recorded-timing setup |
| use_expander.py | In-memory <use data-icon="..."> expansion (delegates to svg_finalize.embed_icons) |
| 	span_flattener.py | In-memory positional <tspan> flattening (delegates to svg_finalize.flatten_tspan) |
| nimation_config.py | Sidecar animation JSON: loading, validation, SVG group scanning, chrome filtering |

## Key Conventions

- **Default source:** svg_output/ (original Executor output). Use -s final for svg_final/.
- **Backup always:** svg_output/ copied to ackup/<timestamp>/ on every run.
- **Native = DrawingML shapes.** --svg-snapshot also emits an SVG-image preview pptx.
- **--no-merge:** Keep dy-stacked lines as separate text frames (default merges into one).
- **Animation flags:** -t (transition), -a (entrance), --animation-trigger, --animation-config, --auto-advance.
- **Compatibility mode (default):** dual PNG+SVG blip. --no-compat for Office 2019+ only.
- **Converter invariants:** flat transform decomposition (no shear), browser-style inherit, text-anchor via manual offset.

## Anti-Patterns (hard fails)

| Pattern | Why |
|---------|-----|
| --only native or --only legacy | Suppresses one output file. NEVER use. |
| cp svg_output/ X instead of backup | Backup is automatic on every run. Don't manually copy. |
| cp instead of inalize_svg.py on raw SVGs | finalize does icon embed + crop + text flatten + rect->path. |
| SVG source from svg_final/ by default | Default reads svg_output/ — only -s final switches it. |
| Calling svg_to_pptx.py directly | Use pptx_cli.py entry point (part of svg_to_pptx package). |
