# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

graphify (PyPI: `graphifyy`) is a CLI + AI-assistant skill that turns a folder of
code, docs, papers, images, or video into a queryable knowledge graph
(`graphify-out/`): god nodes, Leiden communities, cross-file `calls` /
`imports` / `inherits` edges resolved via tree-sitter, and `query` / `path` /
`explain` / MCP tools against the resulting `graph.json`. Code parsing is
local-only (tree-sitter, no LLM); only the semantic pass over docs/media calls
a model backend. This repo is the library + CLI itself, not a consumer of it —
there is no `graphify-out/` here.

## Development setup

```bash
uv sync --all-extras          # editable install + dev deps (pytest, ruff, bandit, pyright, pre-commit...)
uv run graphify --version     # verify
uv run pre-commit install     # one-time, runs skillgen-check + ruff on commit
```

Active development happens on the `v8` branch. Commit style: `fix: <description>`
/ `feat: <description>` / `docs: <description>`.

## Common commands

```bash
uv run pytest tests/ -q                    # full test suite
uv run pytest tests/test_extract.py -q     # one module
uv run pytest tests/ -q -k "python"        # filter by name
uv run ruff check --config pyproject.toml  # lint (E9, F63, F7, F82 only — see [tool.ruff.lint])
uv run bandit -r graphify -ll               # security static analysis
uv run pip-audit --strict                   # dependency vuln scan
uv run python -m tools.skillgen --check     # verify generated skill artifacts aren't stale (see below)
```

CI (`.github/workflows/ci.yml`) runs three jobs: `skillgen-check` (generated
artifacts + per-host coverage audits), `test` (pytest on Python 3.10 and 3.12,
`--all-extras`), and `security-scan` (bandit/pip-audit, non-blocking). Run
`uv run pytest tests/ -q` before opening a PR.

> macOS note: `tests/fixtures/sample.f90` and `sample.F90` collide on
> case-insensitive filesystems (HFS+/APFS) — test Fortran variants on Linux/Docker.

## Architecture

See **ARCHITECTURE.md** for the core pipeline
(`detect → extract → build_graph → cluster → analyze → report → export`),
the extraction output schema (`{nodes, edges}` with `EXTRACTED` /
`INFERRED` / `AMBIGUOUS` confidence labels), and the security threat model
(everything external funnels through `graphify/security.py`). The points
below are things that table doesn't cover.

### `extract.py` is a legacy monolith; new languages live in `extractors/`

`graphify/extract.py` (~5000 lines) still contains the inline extraction logic
for several original languages (Python, JS/TS, Java, C/C++, Kotlin, Ruby,
Scala, Swift, PHP, part of C#), pulling shared helpers from
`extractors/engine.py`, `extractors/resolution.py`, `extractors/models.py`,
and `extractors/base.py`. Every language added since has its own
self-contained `extractors/<lang>.py` module (e.g. `rust.py`, `go.py`,
`elixir.py`, `terraform.py`, `sql.py`) following the pattern in
`extractors/base.py`, and `extract.py` just imports and dispatches to it.
**Add new languages as a new `extractors/<lang>.py` module**, not inline in
`extract.py` — follow the 5-step process in ARCHITECTURE.md ("Adding a new
language extractor").

Cross-file, language-specific resolution passes (receiver typing, qualified
member calls, framework conventions) that used to be hand-wired
`if <lang>_paths: ...` blocks at the tail of `extract()` are now registered as
`LanguageResolver` entries in `graphify/resolver_registry.py` — plug a new
language's cross-file pass in there instead of editing `extract()`.

### Beyond the core pipeline

Notable modules not in the ARCHITECTURE.md table: `dedup.py` /
`semantic_cleanup.py` (node dedup/cleanup passes), `global_graph.py` (merges
per-repo graphs into `~/.graphify/global-graph.json`), `wiki.py` (Obsidian-style
markdown wiki export), `affected.py` (reverse traversal / blast-radius),
`reflect.py` (aggregates `graphify-out/memory/` query outcomes into a lessons
doc), `prs.py` (PR dashboard / triage / conflict detection), `hooks.py` (git
post-commit/post-checkout hook install + merge driver for `graph.json`), and
`install.py` (the large multi-platform skill installer — Claude, Cursor,
Codex, Gemini, Aider, Kilo, and a dozen more agent hosts, each with its own
`_<platform>_install`/`_uninstall` pair).

### Generated skill artifacts — do not hand-edit

`graphify/skill*.md`, `graphify/skill-<host>.md`, and
`graphify/skills/<host>/references/*.md` are **generated** from fragments
under `tools/skillgen/fragments/` (driven by `tools/skillgen/platforms.toml`).
Hand-editing a generated file fails `skillgen-check` in CI and pre-commit the
same way. To change the shipped skill content:

```bash
# edit the relevant fragment(s) under tools/skillgen/fragments/, then:
python -m tools.skillgen            # regenerate every platform's artifacts
python -m tools.skillgen --check    # verify no drift
python -m tools.skillgen --bless    # refresh tools/skillgen/expected/ baselines
```

The six `always_on/*.md` blocks (injected into each host's `CLAUDE.md`/
`AGENTS.md`/etc. on `graphify <host> install`) are likewise generated and
guarded by `--always-on-roundtrip`.

## Contributing

- New language extractor → add a fixture to `tests/fixtures/` and tests to
  `tests/test_languages.py` (per ARCHITECTURE.md's 5-step checklist).
- Worked examples (`worked/{slug}/`, with an honest `review.md` of what the
  graph got right/wrong on a real corpus) are a welcome contribution type.
- Extraction bugs: report with the input file, cache entry
  (`graphify-out/cache/`), and what was missed or wrong.
