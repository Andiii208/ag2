# Screen Context: ask "what was I looking at?" from a local screen history

One self-contained script records what is on your screen — the frontmost window only, OCR'd with Apple Vision — into a local JSONL file, and gives the agent a single **retrospective search** tool over that history. It mirrors the boundaries of the [ScreenContextAgent](https://github.com/ikeikeikeda66/screen-context-agent) project, kept observable in a single file.

Three boundaries shape it:

1. **History stays local.** The JSONL file never leaves the machine, and the agent is given no capture tool at all — only search.
2. **Retrieval only on explicit request.** Both the agent's prompt and the search tool's description say `screen_history` may be called only when the user explicitly asks about earlier screen content. The demo asks a second, screen-unrelated question that must not touch the history.
3. **OCR output is untrusted observation.** Every result is labelled `source=observed_screen, trust=untrusted` — in the text the model sees and in the `ToolResult` metadata — and carries its capture timestamp and source app, so the model quotes observations instead of treating them as fact.

## Requirements

- macOS, with Screen Recording permission for your terminal and Apple Vision available for OCR
- Python >= 3.10
- An Anthropic API key

## Run

From the root of this repository:

```bash
uv pip install "ag2" pyobjc-framework-Cocoa pyobjc-framework-Quartz pyobjc-framework-Vision
export ANTHROPIC_API_KEY=...
python -m examples.tools.screen_context
```

Grant Screen Recording when macOS asks, then switch to the window you want the agent to be asked about when the script prompts you. The history file is `screen_history.jsonl` by default; set `SCREEN_CONTEXT_HISTORY` to place it elsewhere. Delete the file to wipe it.

The capture path is deliberately macOS-only: on other platforms the module still imports, but recording raises a clear error instead of falling back to a weaker capture.

## Tests

```bash
pytest test/tools/test_screen_context.py
```

The tests drive the store and the tool through the public agent seam (`TestConfig` / `TrackingConfig`), so they run on any OS — including Linux CI — even though capture itself only works on macOS.

## Build with AG2

This project is built with [AG2 (Formerly AutoGen)](https://ag2.ai/) and utilizes the following features from the library:

1. `Agent` with a boundary-setting system prompt and a single custom tool.
2. The `@tool` decorator, returning a `ToolResult` whose `metadata` carries the provenance labels.
3. `AnthropicConfig` as the default `ModelConfig`, swappable for other providers or a test double.

Check out more projects built with AG2 at [Build with AG2](https://github.com/ag2ai/build-with-ag2)!
