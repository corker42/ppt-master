# AGENTS.md — image_backends/ Directory

**14 backends, 1 interface.** Provider abstraction for AI image generation. Each backend is a self-contained module invoked by `image_gen.py` via `__import__()`.

## Contract (every backend MUST export)

```python
def generate(prompt: str,
             aspect_ratio: str = "1:1",
             image_size: str = "1K",
             output_dir: str = None,
             filename: str = None,
             model: str = None,
             max_retries: int = MAX_RETRIES) -> str:
    """Returns path to saved image file."""
```

`_generate_image(api_key, ...)` is the private impl. Public `generate()` wraps it in a retry loop with `is_rate_limit_error()` + `retry_delay()`.

## Registration

Backends are registered in `image_gen.py` via the `BACKEND_REGISTRY` dict (NOT a class-based registry). `__init__.py` is minimal (1-line docstring). To add a backend:

1. Create `backend_<name>.py` with `generate()`
2. Add entry to `BACKEND_REGISTRY` in `image_gen.py`:
   - `module`: `"backend_<name>"`
   - `tier`: `"core"` / `"extended"` / `"experimental"`
   - `label`, `default_model`, `key_hint`, `aliases`

## Env Var Convention

| Purpose | Pattern | Example |
|---------|---------|---------|
| Backend select | `IMAGE_BACKEND=<name>` | `IMAGE_BACKEND=openai` |
| API key | `<PROVIDER>_API_KEY` | `OPENAI_API_KEY`, `GEMINI_API_KEY` |
| Base URL | `<PROVIDER>_BASE_URL` | `OPENAI_BASE_URL`, `GEMINI_BASE_URL` |
| Model | `<PROVIDER>_MODEL` | `OPENAI_MODEL`, `GEMINI_MODEL` |

Use `require_api_key(*candidates, message=...)` from `backend_common.py`. Use `os.environ.get()` for optional vars.

## Shared Helpers (`backend_common.py`)

| Function | Purpose |
|----------|---------|
| `save_image_bytes(image_bytes, path, content_type)` | Save with format auto-detection and Pillow transcode |
| `download_image(url, path, headers, timeout)` | HTTP download + save |
| `detect_image_extension(bytes, content_type)` | Magic bytes + Content-Type |
| `resolve_output_path(prompt, output_dir, filename, ext)` | Safe filename from prompt |
| `normalize_image_size(size)` | "512px", "1K", "2K", "4K" canonicalization |
| `require_api_key(*candidates, message)` | Fail-fast env var check |
| `is_rate_limit_error(exc)` | Detect 429 / rate / quota / resource_exhausted |
| `retry_delay(attempt, rate_limited)` | Exponential backoff for rate limits, 5s for others |
| `http_error(response, label)` | Readable RuntimeError from HTTP response |
| `poll_json(url, headers, interval, timeout, ...)` | Poll async endpoint until ready/failed |
| `report_resolution(path)` | Print image dimensions (Pillow) |

Only `MAX_RETRIES=3` is imported as constant.

## Must-Know Patterns

- **Header guard**: `if __name__ == "__main__" and any(arg in {"-h", "--help", "help"} ...)` — print docstring and exit
- **Lazy SDKs**: Import provider SDK (`google-genai`, `openai`, etc.) at module top level. If it fails, `_load_backend()` in image_gen.py prints `pip install` hint.
- **Heartbeat thread**: Long-polling backends (openai, gemini) print progress every 5s via daemon thread
- **Aspect ratio validation**: Each backend defines its own `VALID_ASPECT_RATIOS` list; validate early in `_generate_image()`
- **Size mapping**: Many backends define `ASPECT_RATIO_SIZE_MAP[image_size][aspect_ratio] -> "WxH"`
- **Format alignment**: Use `save_image_bytes()` to transcode mismatched formats through Pillow

## Anti-Patterns (hard failure)

| Anti-pattern | Why |
|---|---|
| `DEPRECATED_IMAGE_KEYS` (IMAGE_API_KEY, IMAGE_MODEL, IMAGE_BASE_URL) | Removed. Use provider-specific names only |
| Positional invocation (`image_gen.py "prompt"`) for pipeline use | Debug only. Pipeline MUST use `--manifest` mode |
| Skipping `--manifest` for single image | Even single-image pipeline must write manifest first |
| Adding new helpers without checking backend_common.py first | Extend first, fork never |
| Raw `open()` on image bytes | Use `save_image_bytes()` for format-aware write |

## Backend Inventory (14)

| Backend | Env Key | Default Model | Tier |
|---------|---------|--------------|------|
| openai | `OPENAI_API_KEY` | `gpt-image-2` (current best overall) | core |
| gemini | `GEMINI_API_KEY` | `gemini-3.1-flash-image-preview` | core |
| qwen | `QWEN_API_KEY` / `DASHSCOPE_API_KEY` | `qwen-image-2.0-pro` | core |
| zhipu | `ZHIPU_API_KEY` / `BIGMODEL_API_KEY` | `glm-image` | core |
| volcengine | `VOLCENGINE_API_KEY` / `ARK_API_KEY` | `doubao-seedream-4-5-251128` | core |
| stability | `STABILITY_API_KEY` | `stable-image-core` | extended |
| bfl | `BFL_API_KEY` | `flux-pro-1.1-ultra` | extended |
| ideogram | `IDEOGRAM_API_KEY` | `ideogram-v3` | extended |
| siliconflow | `SILICONFLOW_API_KEY` | `Qwen/Qwen-Image` | experimental |
| fal | `FAL_KEY` / `FAL_API_KEY` | `fal-ai/imagen3/fast` | experimental |
| replicate | `REPLICATE_API_TOKEN` / `REPLICATE_API_KEY` | `black-forest-labs/flux-1.1-pro` | experimental |
| openrouter | `OPENROUTER_API_KEY` | `google/gemini-3.1-flash-image-preview` | experimental |
| minimax | `MINIMAX_API_KEY` | `image-01` | experimental |
| modelscope | `MODELSCOPE_API_KEY` | `Tongyi-MAI/Z-Image-Turbo` | experimental |
