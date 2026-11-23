# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A portfolio project (started from the DataTalks.Club LLM Zoomcamp) that builds a retrieval agent system step by step. Work is organized into numbered module folders (`01_rag_foundations/`, `02_vector_search/` — the latter is currently empty). The numbered folder names are intentional learning-stage labels, not Python packages.

## Environments

There are two independent `uv` environments (Python 3.11, see `.python-version`):

- **Development** — repo root `pyproject.toml`. Used by `01_rag_foundations/development/`.
  ```bash
  uv sync                      # from repo root
  ```
- **Homework** — `01_rag_foundations/homework/pyproject.toml`, its own `uv.lock`. Homework work goes here and must not modify the development implementation.
  ```bash
  cd 01_rag_foundations/homework && uv sync
  ```

`OPENAI_API_KEY` is loaded from `.env` at the repo root via `python-dotenv` (notebooks call `load_dotenv()`).

There is no test suite, linter, or build step configured. Work is done in Jupyter notebooks; run them **from `01_rag_foundations/development/`** so that plain imports like `from ingestion import ...` and `from rag_pipeline import RAGBase` resolve (the modules are not installed as a package). From elsewhere, load them by file path with `importlib` as shown in `README.md`.

## Architecture (01_rag_foundations/development)

The notebooks are a progression; each builds on helper modules extracted from the previous ones:

1. `01` keyword search basics → `02` extracts the reusable pipeline → `03` persists the index in SQLite → `04` one function-calling round trip → `05` repeated agent loop → `06` ToyAIKit vs. the hand-written loop → `07` Pydantic AI vs. ToyAIKit. Notebooks 05–07 have companion `*-notes.md` explainers that should stay in sync with their notebook.

Reusable modules:

- `ingestion.py` — `load_faq_data()` fetches `https://datatalks.club/faq/json/courses.json` and then each course's JSON; `build_index()` builds an in-memory `minsearch.Index` (text fields `question`/`section`/`answer`, keyword field `course`).
- `rag_pipeline.py` — `RAGBase`: `search` (boosts `question` 3.0, `section` 0.5; filters by `course`, default `llm-zoomcamp`) → `build_context` → `build_prompt` → `llm` (OpenAI **Responses API**, `developer` role for instructions, default model `gpt-5.4-mini`) → `response_generation` (end-to-end).
- `agent_loop.py` — hand-written bounded tool-calling loop over the Responses API: `agent_loop` appends `response.output` to the message history, `process_response`/`make_call` execute `function_call` items via a `tool_functions` name→callable dict and append `function_call_output` items, and the loop stops when a response contains no function calls (raises `RuntimeError` after `max_iterations`).

Persistent index: `faq_search.db` is a `sqlitesearch.TextSearchIndex` written once by an ingestion step and reopened by later notebooks/processes instead of re-fetching and re-indexing. It is committed to the repo; `*.db-shm`/`*.db-wal` are gitignored.

Framework comparisons use `toyaikit` (`Tools`, `OpenAIResponsesRunner`, `IPythonChatInterface`) and `pydantic_ai` (`Agent`, `RunContext`), both declared in the root dependencies.
