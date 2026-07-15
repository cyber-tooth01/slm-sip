# AGENTS.md

Notes for AI agents (opencode / Claude Code / Gemini CLI) working in `slm-sip`.

## Project

`slm-sip` is the **SLM-SIP MCQ Challenge** — a Colab-GPU fine-tuning pipeline for
`meta-llama/Llama-3.2-1B-Instruct` using Transformers + PEFT (LoRA). The real work
lives in `notebook.ipynb`, designed to run on Google Colab (T4 / A100). `main.py`
is an unused stub; `uv` manages the local venv (Python 3.14) only for offline edits.

## Colab MCP — driving the runtime from the agent

A `colab-mcp` MCP server is registered globally in
`~/.config/opencode/opencode.jsonc`:

```jsonc
"mcp": {
  "colab-mcp": {
    "type": "local",
    "command": ["uvx", "git+https://github.com/googlecolab/colab-mcp"],
    "timeout": 30000,
    "enabled": true
  }
}
```

It bridges this local agent to a Colab session running in the user's browser.
Source: <https://github.com/googlecolab/colab-mcp>. Requires a client that
supports `notifications/tools/list_changed` and runs locally (opencode does).

### How to use it

1. **The user must have a Colab tab open** with a runtime started
   (Runtime > Change runtime type > T4 GPU for this project). The MCP server
   cannot start a runtime on its own; it only attaches to one already running.
2. On first call, `colab-mcp` exposes a connect/bridge tool. Run it; it returns
   a snippet to paste into a Colab cell (or auto-executes depending on version).
   The runtime proxy is enabled by default (`-p`).
3. Once connected, the Colab tools (e.g. `execute_cell`, `list_cells`,
   `read_output`) appear via `notifications/tools/list_changed`. Use them to
   run cells from `notebook.ipynb` against the live runtime and read back
   outputs/errors.
4. Log files live in `/tmp/colab-mcp-logs/` if a cell silently fails.

### Constraints for this notebook

- **Gated model**: `meta-llama/Llama-3.2-1B-Instruct` needs an HF token. The
  notebook expects `HF_TOKEN` in Colab Secrets. Do not commit a token; if the
  user asks, point them to Colab > Secrets > add `HF_TOKEN`.
- **GPU required**: the notebook `assert torch.cuda.is_available()`. Make sure
  the runtime is GPU before running training cells.
- **Data**: `train.csv`, `test.csv`, `sample_submission.csv` must be uploaded
  to the Colab working dir (or mounted from Drive with `DATA_DIR` adjusted).
- Outputs / LoRA adapter go to `./llama1b-mcq-lora` on the Colab runtime; pull
  artifacts down to this repo only if the user asks.

## Local commands

No lint/typecheck configured. The only local runnable check is the stub:

```bash
uv sync
uv run python main.py
```

Notebook execution is **not** done locally — go through the Colab MCP.
