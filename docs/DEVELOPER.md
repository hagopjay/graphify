# Developer Guide: Tinker, Customize, Extend

Want to customize graphify's extraction, add a new language, or build tools on top? This guide shows you how.

---

## Before You Start

- **Understanding:** Read [Concepts & Mental Models](CONCEPTS.md)
- **Code structure:** Familiarize yourself with [ARCHITECTURE.md](../ARCHITECTURE.md)
- **Python:** The codebase is Python 3.10+
- **Dependencies:** `pytest`, `networkx`, `tree-sitter`, etc. (see `pyproject.toml`)

---

## Part 1: Understanding the Extraction Pipeline

### The Extract Module

All extraction happens in `graphify/extract.py`. Here's the flow:

```python
def extract(path: Path) -> dict:
    """
    Input: A file path (any language)
    Output: {
        "nodes": [{"id": "unique", "label": "name", "source_file": "path", "source_location": "L42"}],
        "edges": [{"source": "id_a", "target": "id_b", "relation": "calls", "confidence": "EXTRACTED"}]
    }
    """
    if path.suffix == ".py":
        return extract_python(path)
    elif path.suffix == ".js":
        return extract_javascript(path)
    # ...
```

### Each Language Has a Handler

For example, `extract_python()`:

```python
def extract_python(path: Path) -> dict:
    """
    1. Parse the file with tree-sitter-python
    2. Walk the AST
    3. Collect nodes (function defs, class defs, imports)
    4. Collect edges (calls, inherits, imports)
    5. Run call-graph second pass (INFERRED edges)
    6. Validate and return
    """
```

### The Pattern (Pseudo-code)

```python
def extract_<language>(path: Path) -> dict:
    # 1. Read file
    code = path.read_text()
    
    # 2. Parse with tree-sitter
    parser = Parser()
    parser.set_language(Language(...))
    tree = parser.parse(code.encode())
    
    # 3. Walk AST
    nodes = []
    edges = []
    
    def walk(node, parent=None):
        # Is this a function/class/import?
        if node.type == "function_definition":
            nodes.append({
                "id": f"{path.stem}:{node.child_by_field_name('name').text}",
                "label": node.child_by_field_name('name').text.decode(),
                "source_file": str(path),
                "source_location": f"L{node.start_point[0]}"
            })
        
        # Recurse
        for child in node.children:
            walk(child, node)
    
    walk(tree.root_node)
    
    # 4. Second pass: call-graph resolution
    # (match function calls to their definitions)
    edges.extend(resolve_calls(nodes, tree))
    
    # 5. Validate
    from graphify.validate import validate_extraction
    data = {"nodes": nodes, "edges": edges}
    validate_extraction(data)
    
    return data
```

---

## Part 2: Adding a New Language

### Example: Add Rust Support

#### Step 1: Create the Extractor

Add a function to `graphify/extract.py`:

```python
def extract_rust(path: Path) -> dict:
    """Extract nodes and edges from a Rust file."""
    code = path.read_text()
    parser = Parser()
    parser.set_language(Language(build_library("tree-sitter-rust")))
    tree = parser.parse(code.encode())
    
    nodes = []
    edges = []
    
    def walk(node, parent=None):
        # Rust function definitions
        if node.type == "function_item":
            name_node = node.child_by_field_name("name")
            if name_node:
                func_id = f"{path.stem}:{name_node.text.decode()}"
                nodes.append({
                    "id": func_id,
                    "label": name_node.text.decode(),
                    "source_file": str(path),
                    "source_location": f"L{node.start_point[0]}"
                })
        
        # Rust struct definitions
        if node.type == "struct_item":
            name_node = node.child_by_field_name("name")
            if name_node:
                struct_id = f"{path.stem}:{name_node.text.decode()}"
                nodes.append({
                    "id": struct_id,
                    "label": name_node.text.decode(),
                    "source_file": str(path),
                    "source_location": f"L{node.start_point[0]}"
                })
        
        # Rust use statements (imports)
        if node.type == "use_declaration":
            # Parse the use path and link to the imported module
            pass
        
        # Recurse
        for child in node.children:
            walk(child, node)
    
    walk(tree.root_node)
    
    # Validate
    from graphify.validate import validate_extraction
    data = {"nodes": nodes, "edges": edges}
    validate_extraction(data)
    
    return data
```

#### Step 2: Register the File Extension

In `graphify/extract.py`, update the `extract()` dispatch:

```python
def extract(path: Path) -> dict:
    if path.suffix == ".py":
        return extract_python(path)
    elif path.suffix == ".js":
        return extract_javascript(path)
    elif path.suffix == ".rs":  # ADD THIS
        return extract_rust(path)
    # ...
```

#### Step 3: Register in File Detection

In `graphify/detect.py`, add `.rs` to `CODE_EXTENSIONS`:

```python
CODE_EXTENSIONS = {
    ".py", ".js", ".ts", ".tsx", ".jsx",
    ".java", ".go", ".rb", ".php", ".c", ".cpp",
    ".rs",  # ADD THIS
    # ... more extensions
}
```

#### Step 4: Add Tree-Sitter Dependency

In `pyproject.toml`:

```toml
dependencies = [
    # ... existing
    "tree-sitter-rust",  # ADD THIS
]
```

#### Step 5: Watch Files

In `graphify/watch.py`, add to `_WATCHED_EXTENSIONS`:

```python
_WATCHED_EXTENSIONS = {
    ".py", ".js", ".ts", ".jsx", ".tsx",
    ".rs",  # ADD THIS
    # ...
}
```

#### Step 6: Add Tests

Create `tests/fixtures/example.rs`:

```rust
pub fn hello_world() {
    println!("Hello, world!");
}

pub struct User {
    name: String,
    age: u32,
}
```

Add a test in `tests/test_languages.py`:

```python
def test_extract_rust():
    path = Path("tests/fixtures/example.rs")
    result = extract_rust(path)
    
    assert result["nodes"] != []
    assert any(n["label"] == "hello_world" for n in result["nodes"])
    assert any(n["label"] == "User" for n in result["nodes"])
    
    # Test edges (function definition)
    assert len(result["edges"]) > 0
```

Run tests:
```bash
pytest tests/test_languages.py::test_extract_rust -v
```

---

## Part 3: Customizing Extraction for Your Codebase

### Scenario: Skip Certain Patterns

You want to exclude test files or generated code from the graph.

**In `graphify/detect.py`:**

```python
def collect_files(root: Path) -> list[Path]:
    """Collect files, excluding tests and generated code."""
    files = []
    for path in root.rglob("*"):
        # Skip tests
        if "test" in path.parts:
            continue
        # Skip generated
        if "_generated" in path.name or "proto" in path.name:
            continue
        # Skip common junk
        if path.name in (".gitignore", ".DS_Store", "__pycache__"):
            continue
        
        if path.is_file() and path.suffix in CODE_EXTENSIONS:
            files.append(path)
    
    return files
```

### Scenario: Add Custom Node Types

You have a convention in your codebase (e.g., `@api_endpoint`) and want graphify to recognize it.

**In `graphify/extract.py` (for Python example):**

```python
def extract_python(path: Path) -> dict:
    # ... existing code ...
    
    # Add custom node detection
    def walk(node, parent=None):
        # Existing logic...
        
        # NEW: Detect @api_endpoint decorators
        if node.type == "decorated_definition":
            decorator_list = node.child_by_field_name("decorator")
            if decorator_list and "api_endpoint" in decorator_list.text.decode():
                func_node = node.child_by_field_name("definition")
                name = func_node.child_by_field_name("name").text.decode()
                
                nodes.append({
                    "id": f"{path.stem}:{name}:endpoint",
                    "label": f"{name} (endpoint)",
                    "source_file": str(path),
                    "source_location": f"L{node.start_point[0]}",
                    "type": "api_endpoint"  # Custom attribute
                })
        
        # Recurse...
```

---

## Part 4: Building Tools on Top

### Example: A CLI Command for Finding API Endpoints

```python
# Save as graphify/endpoints.py

def find_endpoints(graph_path: Path) -> list[dict]:
    """Find all API endpoints in a graph."""
    import json
    
    with open(graph_path) as f:
        graph_data = json.load(f)
    
    endpoints = []
    for node in graph_data.get("nodes", []):
        if node.get("type") == "api_endpoint":
            endpoints.append({
                "name": node["label"],
                "file": node["source_file"],
                "location": node["source_location"]
            })
    
    return endpoints

# Save as graphify/cli.py (add to the existing CLI)

@click.command()
@click.argument("graph_path", type=click.Path(exists=True))
def endpoints(graph_path):
    """List all API endpoints in the graph."""
    from graphify.endpoints import find_endpoints
    
    eps = find_endpoints(Path(graph_path))
    for ep in eps:
        click.echo(f"{ep['name']:30} {ep['file']}:{ep['location']}")

# Usage:
# $ graphify endpoints graphify-out/graph.json
# hello()                        main.py:L42
# create_user()                  api/users.py:L10
```

---

## Part 5: Integration Patterns

### Pattern 1: Hook into the Build Pipeline

You want to run custom logic after extraction but before clustering.

**In `graphify/build.py`:**

```python
def build_graph(extractions: list[dict]) -> nx.Graph:
    """Build graph with custom post-processing hook."""
    # ... existing code ...
    
    # After building nodes and edges, run custom logic
    from graphify.hooks import apply_custom_rules
    G = apply_custom_rules(G)
    
    return G
```

**In `graphify/hooks.py`:**

```python
def apply_custom_rules(G: nx.Graph) -> nx.Graph:
    """Apply custom rules specific to your codebase."""
    # Example: Mark all test functions as a separate node type
    for node, attrs in G.nodes(data=True):
        if "test" in node.lower():
            attrs["node_type"] = "test"
    
    return G
```

### Pattern 2: Export a Custom Report

You want to generate a report tailored to your team.

**In `graphify/report.py`:**

```python
def render_custom_report(G: nx.Graph, analysis: dict) -> str:
    """Render a team-specific report."""
    report = f"""# Architecture Report

## Critical Paths
These are the most important code paths:

{render_god_nodes(G, limit=5)}

## Risk Areas
These components are frequently modified and highly coupled:

{render_risk_areas(G)}

## Onboarding Guide
New team members should start with:

{render_onboarding_path(G)}
"""
    return report
```

---

## Part 6: Testing Your Changes

### Unit Tests

```bash
pytest tests/test_extract.py -v
pytest tests/test_languages.py -v
```

### Integration Tests

```bash
# Run graphify on a real project
graphify . some_real_project/

# Verify the output
test -f some_real_project/graphify-out/graph.json && echo "✓ Output generated"
```

### Regression Tests

```bash
# Save a baseline graph
cp graphify-out/graph.json graphify-out/graph.baseline.json

# Make changes to the codebase
# (e.g., add a new language extractor)

# Re-run graphify
graphify .

# Compare (should have more or same nodes)
graphify stats graphify-out/graph.baseline.json
graphify stats graphify-out/graph.json
```

---

## Part 7: Performance Optimization

### Profiling Extraction

```bash
python -m cProfile -s cumtime -m graphify . > profile.txt
# Look for slow extractors
```

### Caching Extractions

If extraction is slow, cache results:

```python
# In graphify/cache.py
def cache_key(path: Path) -> str:
    """Generate a cache key based on file content."""
    import hashlib
    content = path.read_bytes()
    return hashlib.md5(content).hexdigest()

def get_cached_extraction(path: Path) -> dict | None:
    """Retrieve cached extraction if file hasn't changed."""
    cache_dir = Path.home() / ".graphify" / "cache"
    cache_file = cache_dir / f"{cache_key(path)}.json"
    
    if cache_file.exists():
        import json
        return json.loads(cache_file.read_text())
    return None

def save_extraction_cache(path: Path, result: dict):
    """Save extraction result to cache."""
    cache_dir = Path.home() / ".graphify" / "cache"
    cache_dir.mkdir(parents=True, exist_ok=True)
    
    cache_file = cache_dir / f"{cache_key(path)}.json"
    import json
    cache_file.write_text(json.dumps(result))
```

---

## Part 8: Debugging Tips

### Print Node/Edge Info

```bash
# Print a raw graph.json for inspection
cat graphify-out/graph.json | jq '.nodes[] | select(.label == "MyClass")'
```

### Trace Extraction for a File

```python
# In Python, add debug output
def extract_python(path: Path) -> dict:
    print(f"Extracting {path}...")
    # ... extraction code ...
    print(f"  Found {len(nodes)} nodes, {len(edges)} edges")
    return {"nodes": nodes, "edges": edges}
```

### Visualize Extraction

```bash
# Generate a small test graph
graphify . tests/fixtures/sample_code/ --out /tmp/test-graph

# Check the output
graphify stats /tmp/test-graph/graph.json
```

---

## Part 9: Contributing Back

If you improve graphify, consider contributing:

1. **Fork** the repository
2. **Create a branch** (`git checkout -b feature/my-improvement`)
3. **Add tests** for your changes
4. **Run the test suite** (`pytest tests/ -v`)
5. **Open a PR** with a clear description

See [CONTRIBUTING.md](../CONTRIBUTING.md) for details.

---

## Reference: Common Tasks

| Task | How To | File |
|------|-------|------|
| **Add a language** | Create `extract_<lang>()` | `graphify/extract.py` |
| **Customize nodes** | Modify walk logic in `extract_<lang>()` | `graphify/extract.py` |
| **Customize edges** | Add edge creation in `extract_<lang>()` | `graphify/extract.py` |
| **Skip files** | Modify `collect_files()` | `graphify/detect.py` |
| **Add a CLI command** | Create function, decorate with `@click.command()` | `graphify/cli.py` |
| **Generate a report** | Create `render_<name>()` function | `graphify/report.py` |
| **Post-process graph** | Add hook in `build_graph()` | `graphify/hooks.py` |
| **Cache results** | Implement in `cache.py` | `graphify/cache.py` |

---

## Getting Help

- **Tree-sitter docs:** https://tree-sitter.github.io/tree-sitter/
- **NetworkX docs:** https://networkx.org/
- **graphify Discord:** https://discord.gg/598Ad9zQZ
- **GitHub Issues:** https://github.com/Graphify-Labs/graphify/issues

Happy tinkering! 🔧
