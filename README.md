# NeMo Agent Toolkit — Example Notebooks

Four self-contained Jupyter notebooks demonstrating [NVIDIA NeMo Agent Toolkit](https://docs.nvidia.com/nemo/agent-toolkit/latest/index.html)
(package name `nvidia-nat`, previously called *AIQToolkit*).

- **`01_nat_basics.ipynb`** — core concepts: installing the toolkit, writing a workflow config
  (`functions:` / `llms:` / `workflow:`), running it via the `nat` CLI, running it from Python, and building a
  reusable workflow object with `WorkflowBuilder`.
- **`02_prompt_optimization.ipynb`** — using the toolkit's built-in optimizer (`nat optimize`) to automatically
  rewrite an agent's prompt with a genetic algorithm, scored against a small evaluation dataset with `nat eval`,
  and comparing accuracy before/after.
- **`03_memory.ipynb`** — memory in agentic systems (working memory vs. long-term memory vs. RAG), and the
  toolkit's memory module: the `memory:` config section, the built-in `add_memory` / `get_memory` / `delete_memory`
  tools, a survey of the shipped backends, and a hands-on example using a self-hosted Redis backend that persists
  a fact across two separate `nat run` invocations.
- **`04_observability_file_tracing.ipynb`** — NAT's built-in file tracing exporter: configuring
  `general.telemetry.tracing`, running a simple `chat_completion` workflow, and inspecting the local trace file
  that captures workflow/LLM spans, timing, inputs, outputs, and errors.

Run `01_nat_basics.ipynb` first — `02_prompt_optimization.ipynb`, `03_memory.ipynb`, and
`04_observability_file_tracing.ipynb` all build on the same workflow config pattern and can be done in any order
after that.

## Setup

Requires Python 3.11, 3.12, or 3.13, and [uv](https://docs.astral.sh/uv/).

```bash
uv venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
uv pip install -r requirements.txt
```

### API key

All four notebooks call NVIDIA-hosted NIM microservices, which require an `NVIDIA_API_KEY`:

1. Sign in at [build.nvidia.com](https://build.nvidia.com/).
2. Open any model and generate a personal API key.
3. Either `export NVIDIA_API_KEY=nvapi-...` before launching Jupyter, or enter it when prompted inside the
   notebooks (they use `getpass` so the key isn't echoed or saved to disk).

### Using a different NIM-compatible endpoint (optional)

Every `_type: nim` LLM/embedder block in these notebooks sets `base_url: ${NVIDIA_BASE_URL}` — that's NAT's own
environment variable interpolation syntax, applied when the config YAML is loaded. If `NVIDIA_BASE_URL` isn't set,
this resolves to an empty/null value, which NAT omits from the request entirely — at that point,
`langchain-nvidia-ai-endpoints` (the library backing `_type: nim`) falls back to its own built-in default, the
public build.nvidia.com catalog, which is what an external user with a plain `nvapi-...` key should use. If you
need to point at a different NIM-compatible endpoint instead — a self-hosted NIM container, an internal gateway,
etc. — set the `NVIDIA_BASE_URL` environment variable (e.g. in a local, gitignored `.env` file) alongside
`NVIDIA_API_KEY`, and every LLM/embedder in every notebook picks it up automatically with no other changes.

### Docker (notebook 3 only)

`03_memory.ipynb` uses a self-hosted Redis Stack container (RediSearch + RedisJSON) as the long-term memory
backend — it's the only one of the toolkit's four memory backends that doesn't require a third-party cloud
account. You'll need Docker (or Podman) installed, or a Redis Stack server of your own reachable at
`localhost:6379`. Notebooks 1, 2, and 4 don't need this.

## Launch

```bash
jupyter notebook notebooks/
```

or open the `notebooks/` folder in VS Code / JupyterLab and run the notebooks with the `.venv` kernel.

## Notes

- All four notebooks write small files into the working directory as they run (`workflow.yml`, `eval_dataset.json`,
  `optimizer_config.yml`, `optimizer_results/`, `eval_baseline/`, `eval_optimized/`, `memory_workflow.yml`,
  `memory_workflow_with_delete.yml`, `observability_config.yml`, `traces/`). These are scratch outputs, safe to
  delete between runs.
- The notebooks use the toolkit's own runtime paths directly. Notebooks 1 and 3 invoke `nat` CLI commands with `!`
  shell cells; notebook 2 uses the Python `EvaluationRun` / `optimize_config` APIs so its notebook-local
  prompt-only workflow registration is visible to the evaluator and optimizer.
- `requirements.txt` pins `nvidia-nat==1.8.0`; adjust the version if you want a newer release.
- The upstream docs describe the toolkit's `main` branch and can drift from whatever version you have pinned —
  this bit us twice while building notebook 2 (the `ragas` evaluator and `optimized_config.yml` output don't exist
  in `1.8.0`). When something doesn't match, `nat info components` and the installed source under
  `.venv/lib/*/site-packages/nat/` are more reliable than the docs.
- **Before committing a run's output**, check for secrets in cell output: `nat` logs the full effective config —
  `api_key` included, in plain text — at INFO level any time you use `--override` (see the note in notebook 2,
  step 6). If you add your own `--override` cells, pass `nat --log-level warning ...` to suppress that dump, and
  clear notebook outputs before committing as a general habit.

## References

- [Documentation](https://docs.nvidia.com/nemo/agent-toolkit/latest/index.html)
- [GitHub repository](https://github.com/NVIDIA/NeMo-Agent-Toolkit)
- [Optimizer guide](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/main/docs/source/improve-workflows/optimizer.md)
- [Evaluation guide](https://github.com/NVIDIA/NeMo-Agent-Toolkit/blob/main/docs/source/improve-workflows/evaluate.md)
- [Memory example (Redis)](https://github.com/NVIDIA/NeMo-Agent-Toolkit/tree/main/examples/memory/redis)
