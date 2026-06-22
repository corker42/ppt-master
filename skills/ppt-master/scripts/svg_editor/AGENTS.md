# AGENTS.md — svg_editor/

**Browser-based live preview server for SVG annotation and editing.** Flask + socketio, serves at http://localhost:5050.

## Files

| File | Purpose |
|---|---|
| server.py (939L) | Flask app, API endpoints, file watcher, SVG image/icon inlining, .lock singleton |
| annotations.py (338L) | data-edit-target / data-edit-annotation read/write on SVG XML |
| __init__.py | Subpackage marker (from svg_editor import server) |
| static/ | index.html + app.js + style.css — the browser UI |

## Launch

```bash
# Step 6 auto-start (long timeout):
python3 svg_editor/server.py <project_path> --live

# Post-export re-entry (shorter timeout):
python3 svg_editor/server.py <project_path>
```

- Server binds 127.0.0.1:5050. Port conflict → --port <other> and report new URL.
- --live = 7200s idle timeout (generation-length). Plain = 900s. --timeout 0 disables.
- --no-browser = remote server (SSH port forward). webbrowser.open() used otherwise.
- **Single instance per project**: <project>/.live_preview.lock records pid+port. Second launch rejects and prints existing URL. Stale locks (dead pid) auto-overwritten.

## How It Works

- Serves svg_output/*.svg and images/* with <use data-icon> inlined for correct rendering.
- **Direct edit** (deterministic): select element → right panel → preview updates immediately → **Apply changes** writes to svg_output/. Drag-to-move on selected element. Arrow-key nudge (1px; Shift+10px).
- **Annotate** (needs AI): select element → write instruction → **Add annotation** → **Apply changes** writes data-edit-target/data-edit-annotation into SVG. Then user says "apply my annotations" → agent edits SVGs and re-exports.
- **Undo**: Ctrl+Z drops last staged edit (per-slide LIFO). Coalesces consecutive same-element edits. **Apply changes** appends history to <project>/.live_edits.jsonl.

## Critical Rules

| Rule | Detail |
|---|---|
| **Annotations from generation** | Handled **post-export** only (after Step 7). Do NOT read/apply during generation. |
| **Already running** | NEVER restart. Point user to the URL. |
| **Re-export after edits** | Chat-driven: finalize_svg.py <path> → svg_to_pptx.py <path>. Editor never runs export pipeline. |
| **Apply annotations** | Run check_annotations.py <path> first to list pending changes. Edit SVGs per annotation text. Remove data-edit-target/data-edit-annotation attributes. Re-export. |
| **Only "Apply changes" writes disk** | Staged edits + annotations in server memory until **Apply changes**. Closing tab while unsaved work exists triggers browser "leave site?" prompt. |
| **Transient ids** | _edit_N assigned to elements while editor runs. Stripped from unannotated elements before write-back. |
| **Stop conditions** | User clicks **Exit preview**, says "stop preview" in chat, idle timeout fires, or process killed. |

## UI Details

- Bilingual (EN/中): auto-detects from navigator.language, persists in localStorage, toggle via right panel button.
- Slide nav: first/prev/next/last buttons + ←/→/Home/End keys (suppressed while typing annotation).
- Overlap picker: right-click canvas → list of all elements under pointer top→bottom.
- Multi-select: batch editor over shared x/y + fill/stroke/opacity. Text fields appear only when all selected are text.
- SVG <g> group: select via Alt+click or **Select parent group** from child element context.