# Concepts & Mental Models: How graphify Works

This guide builds mental models for how graphify transforms code into explorable knowledge.

## The Core Insight

Your codebase isn't just text files—it's a **network** of relationships. A function calls another, a class inherits from a parent, a module imports a dependency. These relationships form a **graph**.

Traditional tools let you navigate code *by proximity* (open a file, look at what's nearby). graphify lets you navigate code *by relationship* (find how A reaches B, no matter how far apart).

---

## Three Layers: Code, Relations, Meaning

```
Layer 1: Source Code (text)
  ↓
  Parsing (tree-sitter)
  ↓
Layer 2: Relations (syntax graph)
  import HTTPServer from "server.js"
  ↓
  "utils.js imports HTTPServer from server.js" [EXTRACTED]
  ↓
Layer 3: Meaning (knowledge graph)
  "utils needs server initialization"
  "HTTPServer enables API" 
  "API exposes endpoints" [INFERRED]
```

### Layer 1: Source Code
The files you wrote. Plain text (Python, JavaScript, Markdown, whatever).

### Layer 2: Relations (EXTRACTED)
What the code literally says:
- `import X from Y` → "module imports"
- `function foo() { bar(); }` → "calls"
- `class Child extends Parent` → "inherits"

**How?** Tree-sitter parses code into an AST (Abstract Syntax Tree). graphify walks that AST and collects explicit relationships.

**Why tree-sitter?** It's grammar-based (not regex), works across 40+ languages, and is *deterministic*—same code → same parse every time. No LLM needed.

### Layer 3: Meaning (INFERRED)
Relationships we deduce from the network structure:
- "This function is a bottleneck" (high degree in the graph)
- "These modules form a cohesive subsystem" (Leiden clustering)
- "Here's the shortest path from A to B" (graph traversal)
- "This concept appears in multiple communities" (centrality)

**How?** After extracting explicit relationships, we run a second pass:
- **Call-graph resolution:** If `foo()` calls `bar()`, and `bar()` is defined in another file, we now link them
- **Type inference:** If a variable's type is declared elsewhere, we connect it
- **Graph algorithms:** Community detection (Leiden), centrality, path-finding

**Confidence:** Every edge is tagged:
- `EXTRACTED` = from the source directly (100% trust)
- `INFERRED` = from graph analysis (we made a call; mark it)
- `AMBIGUOUS` = we're unsure; flagged for review

---

## The graphify Pipeline

```
┌─────────────┐
│   Your      │
│ Codebase    │
└──────┬──────┘
       │
       ▼
  ┌─────────────────────────────────┐
  │ 1. DETECT                       │ Scan filesystem, filter to code extensions
  │ collect_files(root) → [paths]   │
  └──────┬──────────────────────────┘
         │
         ▼
  ┌─────────────────────────────────────────┐
  │ 2. EXTRACT                              │ Parse each file with tree-sitter
  │ extract(path) → {nodes, edges}          │ Emit nodes (concepts) & edges (relations)
  └──────┬──────────────────────────────────┘
         │ (collect results)
         ▼
  ┌──────────────────────────────────────────┐
  │ 3. BUILD                                 │ Assemble into NetworkX graph
  │ build_graph(extractions) → nx.Graph      │ Nodes: {id, label, source_file, location}
  └──────┬───────────────────────────────────┘ Edges: {source, target, relation, confidence}
         │
         ▼
  ┌──────────────────────────────────────────┐
  │ 4. CLUSTER                               │ Apply Leiden algorithm
  │ cluster(G) → G with `community` attr     │ Detect subsystems, label them
  └──────┬───────────────────────────────────┘
         │
         ▼
  ┌──────────────────────────────────────────┐
  │ 5. ANALYZE                               │ Compute stats on the graph
  │ analyze(G) → {god_nodes, surprises, ...} │ Degree, clustering coeff, centrality
  └──────┬───────────────────────────────────┘
         │
         ▼
  ┌──────────────────────────────────────────┐
  │ 6. REPORT                                │ Render human-readable summary
  │ render_report(G, analysis) → markdown    │ Key concepts, questions, gotchas
  └──────┬───────────────────────────────────┘
         │
         ▼
  ┌──────────────────────────────────────────┐
  │ 7. EXPORT                                │ Write files to graphify-out/
  │ export(G, out_dir) → [files]             │ graph.json, graph.html, vault, svg
  └──────────────────────────────────────────┘
```

Each stage is a pure function: input → output, no side effects (except writing `graphify-out/`).

---

## Key Concepts

### Nodes
A node represents a **concept** that's worth naming:
- Functions, classes, modules
- Files, directories
- Type definitions, constants
- Comments with `# NOTE:` or `# WHY:`
- Sections of a Markdown file
- Frames in a video (if you configure semantic extraction)

Each node has:
- `id` — unique within the graph (e.g., `module.py:MyClass:method`)
- `label` — human-readable name (`MyClass.method`)
- `source_file` — where it came from (`module.py`)
- `source_location` — line number, timestamp, or frame count

### Edges
An edge is a **typed relationship**:

| Relation | Meaning |
|----------|---------|
| `calls` | function A invokes function B |
| `imports` | module A depends on module B |
| `inherits` | class A extends class B |
| `mixes_in` | (Ruby/Elixir) module inclusion |
| `references` | code mentions a concept (variable, constant, etc.) |
| `defines` | this location defines a name |
| `uses` | A depends on B (broad: imports, method calls, etc.) |
| `mentions` | comment or doc text talks about this |
| `appears_in_context` | (semantic) this concept appears near that one |

Every edge has a **confidence** tag:

```
┌─────────────┬─────────────────────────────────────┐
│ EXTRACTED   │ Direct from syntax (e.g., import)   │ Trust: 100%
├─────────────┼─────────────────────────────────────┤
│ INFERRED    │ Deduced from graph structure        │ Trust: High
│             │ (e.g., call-graph second pass)      │          
├─────────────┼─────────────────────────────────────┤
│ AMBIGUOUS   │ Uncertain; flagged for review       │ Trust: Low
└─────────────┴─────────────────────────────────────┘
```

### Communities
Leiden clustering partitions the graph into subsystems. Each community:
- Contains nodes that reference each other densely
- Has minimal edges crossing community boundaries
- Gets an auto-generated label (LLM-assisted, but local)

This surfaces the "shape" of your architecture without you describing it.

### God Nodes
A "god node" is a concept with very high degree (many incoming/outgoing edges). Examples:
- A utility function everyone calls
- A base class with many subclasses
- A shared constant or type

God nodes aren't necessarily bad, but they're *important*—everything flows through them. They're the first place to look for bottlenecks or refactoring.

---

## What Gets Extracted?

### Code (EXTRACTED via AST)

✅ **Extracted automatically** (no LLM):
- Function/method definitions and calls
- Class definitions and inheritance
- Module imports
- Variable declarations (type-annotated)
- Constants (usually)
- Interface/protocol definitions

❌ **Not extracted**:
- Comments (yet—see **Beyond Code** below)
- String literals (too much noise)
- Dynamic calls (`eval()`, `__getattr__`, Ruby's `method_missing`)

### Beyond Code (INFERRED via LLM)

With semantic extraction enabled:
- **Markdown sections** → nodes in the graph
- **Docstrings** → linked to the function they describe
- **TODO/FIXME comments** → surfaced as nodes
- **PDFs/images** → text extracted and linked
- **ADR/RFC files** → relationship suggestions ("This RFC justifies that decision")

---

## Graph Algorithms

### Shortest Path
```
graphify path "User" "Database"
```

Returns the shortest path in the graph. Example output:

```
Shortest path (3 hops):
  User --references--> Session <--uses-- Query --calls--> Database
```

This tells you how those two concepts are connected with minimal hops.

### Explain a Node
```
graphify explain "APIRouter"
```

Returns:
- The node's location (file, line)
- Its community (subsystem)
- All outgoing and incoming edges
- Degree (how many things it connects to)

### Query a Question
```
graphify query "How does authentication work?"
```

Converts your question into a subgraph of related nodes, ranked by relevance.

### Community Info
```
graphify communities graphify-out/graph.json
```

Lists all detected subsystems and their god nodes.

---

## Confidence & Explainability

Every relationship in the graph is traceable:

```json
{
  "source": "utils.js",
  "target": "HTTPServer",
  "relation": "imports",
  "confidence": "EXTRACTED",
  "source_location": "L12"
}
```

You can always ask: *Why is X connected to Y?* → The answer is in `source_location` and `confidence`.

This is different from vector search or language models, where the answer is "because the embeddings said so."

---

## Mental Model: Think of It as an x-ray

Your codebase is a building:
- **Files and folders** are rooms
- **Functions and classes** are furniture
- **Imports and calls** are hallways connecting rooms

**Traditional tools** (grep, LSP, search) let you stand in a room and look around.

**graphify** is an x-ray: it shows you the entire building's structure at once. You see:
- Which rooms are busiest (god nodes)
- How the hallways connect (call graph)
- Which rooms form a cohesive wing (communities)
- The shortest route from Room A to Room B (path queries)

---

## Next Steps

- **[Interactive Learning](LEARNING.md)** — Hands-on: run these commands, observe the output
- **[Cookbook](COOKBOOK.md)** — Real scenarios: "Find API endpoints," "Trace a bug," etc.
- **[Developer Guide](DEVELOPER.md)** — Tinker: customize extraction, add a language

---

## FAQ on Concepts

### Q: Isn't this just a call graph?
**A:** Not quite. A call graph only tracks function calls. graphify extracts *all* relationships: imports, inheritance, type references, and more. And it runs algorithms (clustering, centrality) to surface patterns.

### Q: Why not use tree-sitter just to get to an LLM AST?
**A:** We *do* use tree-sitter, but we stop there for code. Why call an LLM to restate what's explicit in the syntax? We save LLM calls for semantics (docs, ambiguous relationships).

### Q: What if two things should be connected but aren't?
**A:** Check the confidence. If it's EXTRACTED and still missing, it's probably code graphify doesn't parse (dynamic calls, string eval, etc.). If it's INFERRED, the heuristic might have missed it. File an issue or add it to your notes in the graph.

### Q: Can I customize what gets extracted?
**A:** Yes! See [Developer Guide](DEVELOPER.md) for how to tweak extraction rules or add a new language.

---

Ready to see these concepts in action? → **[Interactive Learning](LEARNING.md)**
