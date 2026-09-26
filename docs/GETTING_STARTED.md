# Getting Started with graphify

Welcome! This guide will get you from zero to "wow, that's a knowledge graph" in about 10 minutes.

## What You'll Do Today

1. Install graphify
2. Map a codebase (the fun part)
3. Open the interactive graph in your browser
4. Ask it questions instead of grepping files

## 5-Minute Setup

### Step 1: Install

```bash
# Using uv (recommended - isolated environment)
uv tool install graphifyy

# Or with pip/pipx
pipx install graphifyy
# or: pip install graphifyy
```

**Stuck?** Run `graphify --version` to verify the install.

### Step 2: Register with Your AI Assistant

```bash
graphify install
```

This registers graphify as a skill in Claude Code, Cursor, Copilot, or whichever AI assistant you use.

### Step 3: Run It

In your AI assistant, type:

```
/graphify .
```

That's it. Go grab coffee ☕ — it's mapping your project.

## What Just Happened?

Three files appeared in `graphify-out/`:

```
graphify-out/
├── graph.html          ← Open this in your browser! 🎨
├── GRAPH_REPORT.md     ← Read this for insights 📊
└── graph.json          ← The raw data (for queries) 📦
```

## Your First Exploration

### Open the Graph

1. **Find** `graphify-out/graph.html` in your project
2. **Double-click** it (or right-click → Open in Browser)
3. **Play around!**

What you'll see:
- **Nodes** = concepts, functions, files, classes
- **Edges** = relationships (calls, imports, uses, references)
- **Colors** = communities (detected subsystems)
- **Size** = importance (how connected it is)

**Try these:**
- **Drag nodes** to see what they connect to
- **Click a node** to highlight it and its neighbors
- **Search** for a function name you know
- **Zoom** with your mouse wheel
- **Right-click** a node to "expand" and see all connections

### Read the Report

Open `GRAPH_REPORT.md`:

- **God nodes** = the busiest intersection points ("everything goes through here")
- **Communities** = subsystems graphify detected automatically
- **Suggested questions** = starting points for curiosity

## Your First Query

Now that you have the graph, you can ask it questions *without* re-running the analysis.

In your terminal:

```bash
# List god nodes
graphify explain --god-nodes graphify-out/graph.json

# Search for a concept
graphify explain "FastAPI" graphify-out/graph.json

# Find the shortest path between two things
graphify path "imports" "database" graphify-out/graph.json

# Ask a question
graphify query "How does the API handle authentication?" graphify-out/graph.json
```

## What Makes graphify Different?

| Approach | How It Works | Trade-off |
|----------|-------------|-----------|
| **grep/search** | Find text patterns | "What calls this?" requires manual tracing |
| **Language server (LSP)** | Rich semantics, but one file at a time | Can't see the forest |
| **Vector search (RAG)** | Ask a question in prose | Embeddings aren't explainable; no guarantee the answer is real |
| **graphify** | Built real graph from AST + semantic docs | Trade: ~2 min per large codebase, but then instant queries forever |

## Why Explore This Way?

Instead of:
> "Hmm, how does authentication flow through this system?"
> → `grep -r "auth"` (300 matches)
> → Click through 20 files
> → Still unsure

You can:
> "Hmm, how does authentication flow through this system?"
> → `graphify path "User" "Token"` (instant answer with hops drawn)
> → Visual proof of the chain

## Next Steps

- **[Concepts & Mental Models](CONCEPTS.md)** — Understand *how* graphify builds graphs
- **[Interactive Learning](LEARNING.md)** — Hands-on experiments you can try
- **[Cookbook](COOKBOOK.md)** — Real-world use cases ("Find API endpoints," "Trace a bug," etc.)
- **[Developer Guide](DEVELOPER.md)** — Tinker with the code, add a new language, customize extraction

---

## Troubleshooting

### "graphify: command not found"
`uv tool install` puts `graphify` in `~/.local/bin`. That dir needs to be on your PATH:
```bash
# Refresh your shell
uv tool update-shell
# Or open a new terminal
```

### "ModuleNotFoundError: No module named 'graphify'"
This happens when the skill runs Python from a different environment. If using plain `pip install`, that's the cause. Use `uv tool install` or `pipx install` instead.

### Graph took too long / hit a limit
- **Large codebase (50k+ files)?** Graphify filters to code extensions by default. Check `GRAPH_REPORT.md` for what it included.
- **Many PDFs/images?** Semantic extraction calls your LLM. Check your API credits / rate limits.

### Graph looks empty
- **Node count = 0?** The project might have no supported file types, or an error in extraction. Check the console output.
- **Few edges?** The language might need more extractors. See [Developer Guide](DEVELOPER.md).

---

## Questions?

- **[FAQ & Troubleshooting](FAQ.md)**
- **[Architecture Overview](../ARCHITECTURE.md)** (technical deep-dive)
- **Discord:** Join [Graphify Labs community](https://discord.gg/598Ad9zQZ)
- **GitHub Issues:** [Report a bug](https://github.com/Graphify-Labs/graphify/issues)

Happy exploring! 🚀
