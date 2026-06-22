# AGENTS.md - pptx_to_svg/

**PPTX to SVG reverse conversion engine.** Reads OOXML (DrawingML) directly from a .pptx zip archive and emits shape-level SVG. Standalone tool, NOT part of the main generation pipeline.

## Used By

| Caller | Purpose |
|--------|---------|
| template_fill_pptx | Analysis phase - extract layouts and shape structure from existing decks |
| svg_quality_checker | Cross-reference SVG output against source PPTX |
| svg_editor/server.py | Load original deck for side-by-side comparison |
| svg_to_pptx/drawingml_styles.py, drawingml_elements.py | Round-trip test support |

## CLI Invocation

```
python3 scripts/pptx_to_svg.py <source.pptx> -o <output_dir> [--embed-images] [--keep-hidden] [--inheritance-mode both|layered|flat]
```

Default output structure (mode `both`):
```
<output_dir>/svg/          - layered (masters/layouts/slides separate)
<output_dir>/svg-flat/     - self-contained preview slides
<output_dir>/assets/       - extracted media files
```

## File Map

| File | Role | Public API |
|------|------|------------|
| __init__.py | Package entry | convert_pptx_to_svg() re-exported |
| converter.py | Top-level orchestrator | ConvertOptions, ConvertResult, convert_pptx_to_svg() |
| ooxml_loader.py | .pptx zip reader | OoxmlPackage (context manager), PartRef, SlideRef |
| slide_to_svg.py | Per-slide dispatch | assemble_slide(), AssemblyContext |
| shape_walker.py | Shape tree walker | ShapeNode (sp/pic/cxnSp/grpSp/graphicFrame), walk_sp_tree() |
| emu_units.py | Unit conversions | Xfrm, emu_to_px(), percent_to_ratio() |
| color_resolver.py | Color resolution | ColorPalette, resolve_color(), 6 OOXML types + modifiers |
| fill_to_svg.py | Fill: solid/gradient/none | resolve_fill(), FillResult |
| ln_to_svg.py | Line/connector stroke | resolve_stroke(), StrokeResult + markers |
| txbody_to_svg.py | Text body: paragraphs+runs | convert_txbody(), convert_vertical_txbody() |
| prstgeom_to_svg.py | Preset geometry | convert_prst_geom(), GeomResult (rect/ellipse/polygon/path) |
| custgeom_to_svg.py | Custom geometry to SVG path | convert_custom_geom() to d="..." string |
| pic_to_svg.py | Image extraction | convert_blip_fill(), convert_picture() + crop/stretch |
| tbl_to_svg.py | Table: grid+cells+borders | convert_tbl() |
| effect_to_svg.py | Shadow/glow/blur to SVG filters | convert_effects() to (filter_id, defs_xml) |

## Conventions

- **Real package** (__init__.py). All imports are sibling: `from .emu_units import Xfrm`
- **EMU units** throughout -- use Xfrm for coordinate transforms, never raw ints
- **Three inheritance modes**: `both` (default, writes svg/ + svg-flat/), `layered` (template author view), `flat` (self-contained slides)
- **Preset geometry mapping** covers 4 core (rect/roundRect/ellipse/line) + 20+ PowerPoint shapes (triangle, diamond, hexagon, arrow, star, etc.)
- **Text**: preserves runs + formatting + paragraph structure; bullets rendered as literal prefixes; no automatic word wrap
- **Color fallback**: ColorPalette resolves via master's `a:clrMap` to theme `a:clrScheme` to hardcoded defaults
- **Media**: external files by default; `embed_images=True` for base64 inline

## Anti-Patterns

| Anti-pattern | Reason |
|---|---|
| Round-trip PPTX->SVG->PPTX | Lossy - complex effects (3D, artistic) are approximations |
| Using this in-place of direct OOXML analysis | This is specifically SVG output; use ooxml_loader.OoxmlPackage if you only need XML access |
| Writing to output/svg directly | Always use convert_pptx_to_svg() which handles full inheritance + media extraction |
| Treating EffectResult as pixel-perfect | Shadow/glow/soft-edge are SVG approximations; exact match requires PowerPoint rendering |