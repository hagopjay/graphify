# FAQ & Troubleshooting

Quick answers to common questions. If you don't see your issue, check [GitHub Issues](https://github.com/Graphify-Labs/graphify/issues) or ask on [Discord](https://discord.gg/598Ad9zQZ).

---

## Installation & Setup

### Q: "graphify: command not found"

**A:** The command is installed but not on your PATH. Fix it:

```bash
# If using uv tool:
uv tool update-shell

# If using pipx:
pipx ensurepath

# Then open a new terminal
```

If still stuck:
```bash
# Find where it was installed
which uv
~/.local/bin/graphify --version

# Add the directory to your PATH manually
export PATH="$HOME/.local/bin:$PATH"
```

### Q: "ModuleNotFoundError: No module named 'graphify'"

**A:** The Python environment that runs graphify differs from where you installed it.

**Root cause:** Plain `pip install` installs to your current Python, but graphify runs from a different one (common on macOS with multiple Pythons).

**Fix:**
```bash
# Uninstall
pip uninstall graphifyy

# Reinstall using uv or pipx (they isolate the environment)
uv tool install graphifyy
# or
pipx install graphifyy
```

### Q: "graphify install" says "unsupported platform"

**A:** Your AI assistant isn't supported yet. Supported platforms:

- Claude Code (Claude.ai, desktop, VS Code)
- Cursor (Cursor IDE)
- Codex (Codex)
- GitHub Copilot (with prompt support)
- Gemini CLI
- Ollama (local)
- And 15+ more

See full list: `graphify install --help`

### Q: Can I use graphify offline?

**A:** Yes, mostly.

- **Code extraction:** 100% offline (tree-sitter, no network)
- **Documentation extraction:** Requires an LLM
  - Use local LLM: `graphify . --api-key none --api-endpoint http://localhost:8000`
  - Or skip it: `graphify . --code-only`

---

## Running graphify

### Q: "graphify takes too long on my project"

**A:** Depends on project size:

- **< 10k files:** Should complete in 2-5 minutes
- **10-50k files:** 10-20 minutes
- **> 50k files:** May timeout or hit memory limits

**If it's taking too long:**

1. Check progress in the output (it prints status updates)
2. If it's stuck on "semantic extraction" (docs/PDFs), you can cancel and re-run with `--code-only`
3. For huge projects, run on a subset: `graphify ./src ./tests --out mysubset`

### Q: "Error: No module called 'tree_sitter_python'"

**A:** Tree-sitter language library missing.

**Fix:**
```bash
# graphify auto-installs these, but if missing:
pip install tree-sitter-python tree-sitter-javascript
# (repeat for other languages you use)
```

### Q: "ModuleNotFoundError during skill execution"

**A:** The skill's Python differs from your system Python.

**Fix:**
1. Run `graphify hook install` to update embedded Python path in hook
2. If that doesn't work, run `graphify .` manually in the CLI, then commit `graphify-out/` to use the cached Python path

### Q: "graph.html is blank / unresponsive"

**A:** Browser issue or file too large.

**Debug:**
```bash
# Check file size
du -sh graphify-out/graph.html

# If > 100MB, your graph is huge
# Visualize a subgraph instead:
graphify export --community 0 graphify-out/graph.json graphify-out/community-0.html
graphify export --community 1 graphify-out/graph.json graphify-out/community-1.html
```

**Try a different browser:**
```bash
# Open in Firefox instead of Chrome
firefox graphify-out/graph.html
```

---

## Using the Graph

### Q: "Query returns 0 results"

**A:** The question is too specific or phrased differently than the code.

**Try:**
```bash
# Instead of:
graphify query "JWT validation logic" graphify-out/graph.json

# Try simpler terms:
graphify query "authenticate" graphify-out/graph.json
graphify query "token" graphify-out/graph.json
graphify query "security" graphify-out/graph.json
```

Or use `explain` + `path` (more reliable):
```bash
graphify explain "validate_token" graphify-out/graph.json
graphify path "token" "user" graphify-out/graph.json
```

### Q: "Path returns 0 hops but I know they're connected"

**A:** They're connected but not in the direction you traced.

**Check:**
```bash
# Trace the other direction
graphify path "B" "A" graphify-out/graph.json

# See all edges for A
graphify explain "A" graphify-out/graph.json --incoming --outgoing
```

### Q: "Node name has special characters. How do I query it?"

**A:** Escape or quote it.

```bash
# For names with spaces or symbols:
graphify explain "my-function()" graphify-out/graph.json
graphify path "User::create()" "Database::write()" graphify-out/graph.json

# If it still fails, try the file + location:
graphify explain "auth.py:validate" graphify-out/graph.json
```

### Q: "How do I search the graph in graph.html?"

**A:** Use the search box (top-left corner):

- Type a node name → highlights that node + neighbors
- Click a node → shows its edges
- Drag → move around
- Scroll → zoom
- Right-click → context menu

---

## Performance & Limits

### Q: "My graph is 1GB. Is that normal?"

**A:** No. That's huge.

```bash
# Check what's in it
du -sh graphify-out/graph.json
wc -l graphify-out/graph.json  # Line count

# If > 1M nodes, something went wrong
# (typical projects: 100-10k nodes)
```

**Causes:**
- Parsing very large generated files (e.g., a 100MB `.js` minified bundle)
- Including vendor directories (`node_modules/`, `venv/`)

**Fix:**
```bash
# Re-run excluding vendor:
graphify . --exclude "node_modules,venv,.git,build"
```

### Q: "Comparison report is incomplete"

**A:** If comparing very large graphs, the diff can be truncated.

**Workaround:**
```bash
# Export a diff to file
graphify compare old.json new.json > diff.txt
```

### Q: "Can I run graphify on multiple branches?"

**A:** Yes, in separate output dirs:

```bash
# On main branch
graphify . --out graphify-out-main

# Switch to feature branch
git checkout feature-x
graphify . --out graphify-out-feature

# Compare
graphify compare graphify-out-main/graph.json graphify-out-feature/graph.json
```

---

## Extraction Issues

### Q: "Some of my functions don't show up in the graph"

**A:** graphify uses tree-sitter, which has limits:

- **Dynamic code:** `eval()`, `exec()`, Ruby's `method_missing` → not extracted
- **Comments:** Regular comments not extracted (only `# NOTE:` and `# WHY:`)
- **String-based references:** If a function name is in a string → likely not extracted

**Workaround:**
- Make code more explicit (less dynamic)
- Use `# NOTE:` comments to document dynamic relationships

### Q: "The graph includes nodes I don't want (like test files)"

**A:** Configure exclusions:

```bash
graphify . --exclude "test,spec,mock,fixture" --out graphify-out
```

Or customize `detect.py` in your copy of graphify. See [Developer Guide](DEVELOPER.md).

### Q: "An edge is missing / wrong"

**A:** Check confidence:

```bash
# Look at the raw graph
jq '.edges[] | select(.source == "A" and .target == "B")' graphify-out/graph.json

# Check confidence
# EXTRACTED = explicit in source
# INFERRED = graph deduced it
# AMBIGUOUS = uncertain
```

If confidence is INFERRED, it's a best guess. If wrong, open an issue or use [Developer Guide](DEVELOPER.md) to tweak extraction.

### Q: "graph.json has syntax errors"

**A:** Should never happen, but if it does:

```bash
# Validate it
jq . graphify-out/graph.json > /dev/null

# Output should be either "valid JSON" or an error line number
```

If it fails, the extraction had a bug. Please open an issue with:
```bash
graphify . --verbose > graphify.log 2>&1
# Include graphify.log and the project files
```

---

## Semantic Extraction

### Q: "Semantic extraction is slow / expensive"

**A:** Semantic extraction calls your LLM for docs/PDFs/images. Each call costs tokens.

**To skip it:**
```bash
graphify . --code-only
```

**To use a cheaper model:**
```bash
graphify . --model gpt-3.5-turbo
```

**To use a local LLM:**
```bash
# Start Ollama first
ollama run mistral

# Then:
graphify . --api-endpoint http://localhost:11434 --model mistral
```

### Q: "PDFs are included but I don't want them"

**A:** Skip them:

```bash
graphify . --exclude "*.pdf"
```

### Q: "How is semantic extraction different from embedding search?"

**A:** graphify extracts and links concepts. Embedding search ranks by similarity.

```
Semantic extraction (graphify):
  "User model is defined in users.py"
  "TokenValidator uses User model"
  → Creates an edge: TokenValidator --uses--> User

Embedding search (RAG):
  Question: "Where is user validation?"
  Ranks: [users.py (0.95), auth.py (0.82), ...]
  → Returns a ranked list, no guarantee of correctness
```

graphify is more explicit; embeddings are more fuzzy.

---

## Community & Features

### Q: "Can I customize community labels?"

**A:** The labels are auto-generated. To change them:

```bash
# Edit graph.json (dangerous, but possible)
jq '.nodes[] |= if .community == 0 then .community_label = "My Name" else . end' \
  graphify-out/graph.json > tmp.json && mv tmp.json graphify-out/graph.json
```

Better: Open a feature request to let you name communities during generation.

### Q: "Can I merge/split communities?"

**A:** Not directly in graphify, but you can:

1. Extract the community (see [Cookbook](COOKBOOK.md))
2. Analyze it
3. Plan a refactor
4. Re-run graphify after refactoring

### Q: "How often should I re-run graphify?"

**A:** Depends on your needs:

- **Architecture decisions:** After major refactors (quarterly)
- **Onboarding:** Every release or major feature
- **Monitoring:** Monthly or per sprint
- **Debugging:** Ad-hoc as needed

**Recommendation:** Add to CI:
```bash
# .github/workflows/graph.yml
- name: Update knowledge graph
  run: graphify . --out graphify-out
- name: Commit if changed
  run: |
    git add graphify-out/
    git commit -m "Update knowledge graph" || true
```

---

## Troubleshooting Checklists

### Graph Looks Empty

- [ ] `graphify-out/graph.json` exists and has content
- [ ] `graphify stats graphify-out/graph.json` shows > 0 nodes
- [ ] Run `graphify . --verbose` to check for errors
- [ ] Check file extensions: are your code files supported?

### Queries Return Nothing

- [ ] `graphify explain --god-nodes` works (sanity check)
- [ ] Node exists: `jq '.nodes[].label' graphify-out/graph.json | grep -i "your_term"`
- [ ] Try simpler query terms
- [ ] Check confidence: `--incoming --outgoing` to see connected nodes

### Performance Issues

- [ ] `graphify stats` to see node/edge count
- [ ] Time each stage: `graphify . --verbose 2>&1 | grep "Time:"`
- [ ] Try `--code-only` (skip semantic extraction)
- [ ] Run on a subset: `graphify ./src --out /tmp/test`

### Installation Issues

- [ ] `graphify --version` works
- [ ] `python --version` is 3.10+
- [ ] PATH includes `~/.local/bin` or `~/.venv/bin`
- [ ] Try `python -m graphify --version` (direct module invocation)

---

## Getting More Help

1. **Check the docs:**
   - [Getting Started](GETTING_STARTED.md)
   - [Concepts](CONCEPTS.md)
   - [Cookbook](COOKBOOK.md)

2. **Run with verbose mode:**
   ```bash
   graphify . --verbose
   ```

3. **Search GitHub Issues:**
   https://github.com/Graphify-Labs/graphify/issues

4. **Ask on Discord:**
   https://discord.gg/598Ad9zQZ

5. **File an issue with:**
   - OS and Python version
   - `graphify --version`
   - Output of `graphify . --verbose`
   - A minimal reproduction case

---

Good luck! We're here to help. 🚀
