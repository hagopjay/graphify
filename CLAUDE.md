# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

graphify (PyPI package `graphifyy`, CLI `graphify`) turns a folder of code/docs/media into a queryable knowledge graph in `graphify-out/` (`graph.json`, `graph.html`, `GRAPH_REPORT.md`). It ships as a Python library **plus** an AI-assistant skill (`graphify/skill*.md`) that orchestrates the library from inside Claude Code, Codex, Cursor, etc. Read `ARCHITECTURE.md` for the module table and `AGENTS.md` for the graph-first rule (read `graphify-out/GRAPH_REPORT.md` before architecture questions; run `graphify update .` after code edits).

## Commands

Active development branch is `v8` (not `main`). Commit style: `fix: …` / `feat: …` / `docs: …`.

```bash
uv sync --all-extras                      # venv + all extras + dev group (pytest, ruff, pyright…)
uv run pytest tests/ -q                   # full suite (CI runs this on 3.10 and 3.12)
uv run pytest tests/test_extract.py -q    # one module
uv run pytest tests/ -q -k "python"       # filter by name
uv run ruff check --config pyproject.toml graphify tests
uv run pyright                            # basic mode over graphify/ and tests/
uv run pre-commit run --all-files         # ruff + skillgen --check
uv run graphify --help                    # smoke test the CLI
```

CI uses `uv run --frozen` — never let a command rewrite `uv.lock`. Run `uv lock` deliberately if you change dependencies.

Tests are pure unit tests: no network, no writes outside `tmp_path`. `tests/test_skillgen.py` reads a pinned baseline commit via `git show`, so it needs full git history (shallow clones fail it). `worked/` holds real-corpus example outputs and is excluded from pytest collection.

## Generated skill files — do not hand-edit

Everything under `graphify/skill*.md`, `graphify/command-kilo.md`, `graphify/skills/<host>/references/*.md`, and `graphify/always_on/*.md` is **rendered** from `tools/skillgen/fragments/` per `tools/skillgen/platforms.toml`. A hand edit to a rendered file fails the pre-commit hook and the CI `skillgen-check` job.

```bash
python -m tools.skillgen            # regenerate rendered artifacts from fragments
python -m tools.skillgen --check    # byte-diff render vs committed + expected/ (CI gate)
python -m tools.skillgen --bless    # refresh tools/skillgen/expected/ after an intended change
```

Workflow: edit a fragment (or `platforms.toml`) → `python -m tools.skillgen` → `--bless` → commit fragments, rendered files, and `expected/` together. CI additionally runs `--audit-coverage`, `--schema-singleton`, `--monolith-roundtrip`, `--always-on-roundtrip` against the v8 baseline.

## Architecture: the pieces that span files

**Pipeline** (`ARCHITECTURE.md`): `detect.collect_files → extract.extract → build.build_graph → cluster.cluster → analyze.analyze → report.render_report → export.export`. Stages pass plain dicts / `nx.Graph`; the only side effects are writes under `graphify-out/`. `validate.validate_extraction` enforces the `{nodes, edges}` schema between extract and build. Every edge carries `confidence` ∈ `EXTRACTED | INFERRED | AMBIGUOUS`.

**Two extraction passes, one merge.** The skill (`graphify/skill.md`, Step 3) runs Part A — deterministic tree-sitter AST extraction via `graphify extract` (local, no LLM) — then Part B — semantic extraction of docs/PDFs/images by assistant subagents (or `graphify/llm.py` backends when run outside an assistant) — and Part C merges them (AST nodes win, semantic nodes deduped by id) before `graphify build`. Keep code-file coverage in Part A; do not route source files to the semantic pass.

**`extract.py` is a facade mid-migration.** Per-language extractors are being moved verbatim into `graphify/extractors/<lang>.py` (see `graphify/extractors/MIGRATION.md` — read it before touching any extractor). Rules that matter: import direction is strictly `extract.py → extractors/` (never import `graphify.extract` inside the package); `extract.py` must keep re-exporting every moved name; shared helpers go in `extractors/base.py`; the config-driven languages (python, js, java, c, cpp, csharp, kotlin, scala, php, lua, swift, groovy…) share `_extract_generic` and must move as one batch, not individually. `extractors/__init__.py:LANGUAGE_EXTRACTORS` is the registry seed; dispatch still flows through `extract.extract()`.

**Cross-file resolution is a registry.** Language-specific second passes (receiver typing, qualified member calls) register a `LanguageResolver(name, suffixes, resolve)` in `graphify/resolver_registry.py`; passes run only when the corpus contains a matching suffix. New languages plug in by registering, not by editing `extract()`'s tail. Companion modules: `symbol_resolution.py`, `ruby_resolution.py`, `pascal_resolution.py`, `ids.py` (`make_id` — node-id contract; ids are repo-relative, see `_file_stem` in `extractors/base.py`).

**Exporters** are being split the same way: `export.py` → `graphify/exporters/{html,graphdb}.py`, with shared constants in `exporters/base.py` (import only from `base`, never between exporters).

**Adding a language** (ARCHITECTURE.md): add `extract_<lang>()` (prefer a new module under `extractors/` + registry entry), register the suffix in `extract()` dispatch, `detect.CODE_EXTENSIONS`, and `watch._WATCHED_EXTENSIONS`, add the tree-sitter dep to `pyproject.toml`, and add a fixture under `tests/fixtures/` plus tests in `tests/test_languages.py`. Grammar packages must ship prebuilt wheels or go in an optional extra (see the `dm`/`pascal` comments in `pyproject.toml`).

**Security boundary:** all external input passes through `graphify/security.py` (`validate_url`, `safe_fetch*`, `validate_graph_path` — must resolve inside `graphify-out/`, `sanitize_label`). `SECURITY.md` has the threat model.

**Incremental / long-running paths:** `graphify update` (AST-only re-extract of changed files, `affected.py`, `cache.py`), `watch.py` (file watcher → flag file), `hooks.py` (git post-commit hooks embed the interpreter path at install time), `serve.py` (MCP stdio/HTTP server over `graph.json`). Guards to preserve: an empty extraction must never clobber an existing `graph.json`; `to_json` refuses to shrink the graph (#479).

## Conventions specific to this repo

- Issue numbers in comments (`#1392`) are the project's way of recording *why* a guard exists — keep them when moving code.
- `ruff` is intentionally narrow (`E9, F63, F7, F82`, line length 100); don't reformat unrelated code.
- Optional features are `pyproject` extras (`mcp`, `pdf`, `video`, `neo4j`, …); import their deps lazily inside the function that needs them so the default install stays lean.
- Test files are one-per-concern (`test_<feature>.py`), with language fixtures in `tests/fixtures/`.
