# Interactive Examples: Tinker & Learn

Hands-on, runnable examples that teach graphify concepts. Each example is a complete, self-contained snippet you can copy and modify.

---

## Part 1: Working with graph.json in Python

### Example 1: Load and Explore a Graph

```python
# Load a graph and get basic stats
import json
from pathlib import Path

def analyze_graph(graph_path: Path):
    """Quick analysis of a graph."""
    
    with open(graph_path) as f:
        graph = json.load(f)
    
    nodes = graph.get('nodes', [])
    edges = graph.get('edges', [])
    
    # Basic stats
    print(f"Nodes: {len(nodes)}")
    print(f"Edges: {len(edges)}")
    print(f"Density: {2 * len(edges) / (len(nodes) * (len(nodes) - 1)):.3f}")
    
    # God nodes (high degree)
    degree = {}
    for edge in edges:
        source = edge['source']
        target = edge['target']
        degree[source] = degree.get(source, 0) + 1
        degree[target] = degree.get(target, 0) + 1
    
    sorted_by_degree = sorted(degree.items(), key=lambda x: x[1], reverse=True)
    print(f"\nTop 5 god nodes:")
    for node_id, deg in sorted_by_degree[:5]:
        label = next((n['label'] for n in nodes if n['id'] == node_id), node_id)
        print(f"  {label:30} (degree: {deg})")

# Run it
analyze_graph(Path("graphify-out/graph.json"))
```

**Output:**
```
Nodes: 247
Edges: 1234
Density: 0.040

Top 5 god nodes:
  BaseModel                      (degree: 47)
  HTTPServer                     (degree: 24)
  Schema                         (degree: 22)
  Validator                      (degree: 19)
  Request                        (degree: 18)
```

### Example 2: Find Paths Between Nodes

```python
import json
from pathlib import Path
from collections import deque

def find_path(graph_path: Path, start: str, end: str):
    """BFS to find shortest path between two nodes."""
    
    with open(graph_path) as f:
        graph = json.load(f)
    
    edges = graph.get('edges', [])
    nodes_by_id = {n['id']: n for n in graph['nodes']}
    
    # Build adjacency list
    adjacency = {}
    for edge in edges:
        src, tgt = edge['source'], edge['target']
        if src not in adjacency:
            adjacency[src] = []
        adjacency[src].append((tgt, edge['relation']))
    
    # BFS
    queue = deque([(start, [start])])
    visited = {start}
    
    while queue:
        current, path = queue.popleft()
        
        if current == end:
            return path
        
        for neighbor, relation in adjacency.get(current, []):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))
    
    return None  # No path found

def display_path(graph_path: Path, start_label: str, end_label: str):
    """Find and display path between two concepts."""
    
    with open(graph_path) as f:
        graph = json.load(f)
    
    nodes = graph['nodes']
    edges = graph['edges']
    
    # Find node IDs by label
    start_id = next((n['id'] for n in nodes if start_label in n['label']), None)
    end_id = next((n['id'] for n in nodes if end_label in n['label']), None)
    
    if not start_id or not end_id:
        print(f"Nodes not found. Start: {start_id}, End: {end_id}")
        return
    
    path = find_path(graph_path, start_id, end_id)
    
    if path:
        print(f"Path from '{start_label}' to '{end_label}' ({len(path)} hops):")
        for i, node_id in enumerate(path):
            label = next(n['label'] for n in nodes if n['id'] == node_id)
            print(f"  {i+1}. {label}")
            
            if i < len(path) - 1:
                next_id = path[i+1]
                edge = next((e for e in edges if e['source'] == node_id and e['target'] == next_id), None)
                if edge:
                    print(f"     --[{edge['relation']}]--> ")
    else:
        print(f"No path found from {start_label} to {end_label}")

# Run it
display_path(Path("graphify-out/graph.json"), "APIRouter", "DatabaseConnection")
```

### Example 3: Detect Communities Manually

```python
import json
from pathlib import Path
from collections import defaultdict

def analyze_communities(graph_path: Path):
    """Analyze community structure."""
    
    with open(graph_path) as f:
        graph = json.load(f)
    
    nodes = graph['nodes']
    
    # Group by community
    communities = defaultdict(list)
    for node in nodes:
        comm_id = node.get('community', -1)
        communities[comm_id].append(node)
    
    # Analyze each
    for comm_id in sorted(communities.keys()):
        comm_nodes = communities[comm_id]
        print(f"\nCommunity {comm_id}: {len(comm_nodes)} nodes")
        
        # Find god nodes in this community
        comm_ids = {n['id'] for n in comm_nodes}
        degree = {}
        
        for edge in graph['edges']:
            if edge['source'] in comm_ids:
                degree[edge['source']] = degree.get(edge['source'], 0) + 1
            if edge['target'] in comm_ids:
                degree[edge['target']] = degree.get(edge['target'], 0) + 1
        
        top_nodes = sorted(degree.items(), key=lambda x: x[1], reverse=True)[:3]
        
        print("  God nodes:")
        for node_id, deg in top_nodes:
            label = next(n['label'] for n in nodes if n['id'] == node_id)
            print(f"    - {label} (degree: {deg})")

# Run it
analyze_communities(Path("graphify-out/graph.json"))
```

---

## Part 2: Browser-Based Interactive Tools

### Interactive Tool: Graph Explorer

Save this as `graph_explorer.html` in your `graphify-out/` directory:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Graph Explorer</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: #f5f5f5;
            padding: 20px;
        }
        
        .container {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
            overflow: hidden;
        }
        
        header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 30px;
            text-align: center;
        }
        
        h1 { font-size: 2em; margin-bottom: 10px; }
        p { opacity: 0.9; }
        
        .controls {
            display: grid;
            grid-template-columns: 1fr 1fr 1fr auto;
            gap: 15px;
            padding: 20px;
            background: #f9f9f9;
            border-bottom: 1px solid #eee;
        }
        
        .control-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }
        
        label {
            font-size: 0.9em;
            color: #666;
            font-weight: 500;
        }
        
        input, select, button {
            padding: 8px 12px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1em;
            font-family: inherit;
        }
        
        input:focus, select:focus {
            outline: none;
            border-color: #667eea;
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
        }
        
        button {
            background: #667eea;
            color: white;
            border: none;
            cursor: pointer;
            font-weight: 600;
        }
        
        button:hover {
            background: #5568d3;
        }
        
        .content {
            display: grid;
            grid-template-columns: 300px 1fr;
            min-height: 600px;
        }
        
        .sidebar {
            background: #fafafa;
            border-right: 1px solid #eee;
            padding: 20px;
            overflow-y: auto;
        }
        
        .sidebar h3 {
            margin: 20px 0 10px 0;
            color: #333;
            font-size: 0.95em;
        }
        
        .sidebar ul {
            list-style: none;
        }
        
        .sidebar li {
            padding: 8px;
            margin: 4px 0;
            border-radius: 4px;
            cursor: pointer;
            transition: background 0.2s;
        }
        
        .sidebar li:hover {
            background: #e8e8e8;
        }
        
        .sidebar li.active {
            background: #667eea;
            color: white;
        }
        
        .main {
            padding: 20px;
            overflow: auto;
        }
        
        .node-info {
            background: #f0f0f0;
            border-left: 4px solid #667eea;
            padding: 15px;
            border-radius: 4px;
            margin-bottom: 15px;
        }
        
        .node-info h4 {
            color: #333;
            margin-bottom: 10px;
        }
        
        .info-row {
            display: flex;
            justify-content: space-between;
            padding: 5px 0;
            font-size: 0.9em;
        }
        
        .info-label {
            color: #666;
            font-weight: 500;
        }
        
        .info-value {
            color: #333;
            font-family: monospace;
        }
        
        .edges-list {
            font-size: 0.85em;
        }
        
        .edge-item {
            padding: 8px;
            margin: 4px 0;
            background: white;
            border-left: 3px solid #667eea;
            border-radius: 2px;
        }
        
        .edge-label {
            color: #667eea;
            font-weight: 600;
        }
        
        .confidence {
            display: inline-block;
            padding: 2px 6px;
            border-radius: 3px;
            font-size: 0.8em;
            margin-left: 5px;
        }
        
        .confidence.extracted {
            background: #d4edda;
            color: #155724;
        }
        
        .confidence.inferred {
            background: #fff3cd;
            color: #856404;
        }
        
        .confidence.ambiguous {
            background: #f8d7da;
            color: #721c24;
        }
        
        .stats {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 15px;
            margin-bottom: 20px;
        }
        
        .stat-card {
            background: white;
            border: 1px solid #eee;
            border-radius: 4px;
            padding: 15px;
            text-align: center;
        }
        
        .stat-number {
            font-size: 2em;
            font-weight: bold;
            color: #667eea;
        }
        
        .stat-label {
            color: #666;
            font-size: 0.9em;
            margin-top: 5px;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>📊 Knowledge Graph Explorer</h1>
            <p>Load graph.json to explore your codebase</p>
        </header>
        
        <div class="controls">
            <div class="control-group">
                <label>Load Graph File</label>
                <input type="file" id="graphFile" accept=".json">
            </div>
            <div class="control-group">
                <label>Search Nodes</label>
                <input type="text" id="searchInput" placeholder="Function name, class, etc.">
            </div>
            <div class="control-group">
                <label>Filter by Degree</label>
                <select id="degreeFilter">
                    <option value="">All nodes</option>
                    <option value="10">10+</option>
                    <option value="20">20+ (important)</option>
                    <option value="30">30+ (critical)</option>
                </select>
            </div>
            <button id="clearBtn" onclick="clearAll()">Clear</button>
        </div>
        
        <div class="stats" id="stats"></div>
        
        <div class="content">
            <div class="sidebar">
                <h3>God Nodes</h3>
                <ul id="godNodesList"></ul>
                
                <h3>Communities</h3>
                <ul id="communitiesList"></ul>
                
                <h3>Search Results</h3>
                <ul id="searchResults"></ul>
            </div>
            
            <div class="main">
                <div id="nodeDetails"></div>
            </div>
        </div>
    </div>
    
    <script>
        let graphData = null;
        
        document.getElementById('graphFile').addEventListener('change', function(e) {
            const file = e.target.files[0];
            const reader = new FileReader();
            reader.onload = function(event) {
                try {
                    graphData = JSON.parse(event.target.result);
                    initializeUI();
                } catch(err) {
                    alert('Invalid JSON: ' + err.message);
                }
            };
            reader.readAsText(file);
        });
        
        function initializeUI() {
            displayStats();
            displayGodNodes();
            displayCommunities();
            setupSearch();
        }
        
        function displayStats() {
            const nodes = graphData.nodes.length;
            const edges = graphData.edges.length;
            const communities = new Set(graphData.nodes.map(n => n.community)).size;
            
            const density = (2 * edges) / (nodes * (nodes - 1));
            
            document.getElementById('stats').innerHTML = `
                <div class="stat-card">
                    <div class="stat-number">${nodes}</div>
                    <div class="stat-label">Nodes</div>
                </div>
                <div class="stat-card">
                    <div class="stat-number">${edges}</div>
                    <div class="stat-label">Edges</div>
                </div>
                <div class="stat-card">
                    <div class="stat-number">${communities}</div>
                    <div class="stat-label">Communities</div>
                </div>
                <div class="stat-card">
                    <div class="stat-number">${density.toFixed(3)}</div>
                    <div class="stat-label">Density</div>
                </div>
            `;
        }
        
        function displayGodNodes() {
            const degree = {};
            graphData.edges.forEach(edge => {
                degree[edge.source] = (degree[edge.source] || 0) + 1;
                degree[edge.target] = (degree[edge.target] || 0) + 1;
            });
            
            const sorted = Object.entries(degree)
                .sort((a, b) => b[1] - a[1])
                .slice(0, 10);
            
            const list = document.getElementById('godNodesList');
            list.innerHTML = sorted.map(([nodeId, deg]) => {
                const node = graphData.nodes.find(n => n.id === nodeId);
                return `
                    <li onclick="showNode('${nodeId}')">
                        <strong>${node.label}</strong><br>
                        <small style="opacity:0.7;">degree: ${deg}</small>
                    </li>
                `;
            }).join('');
        }
        
        function displayCommunities() {
            const communities = {};
            graphData.nodes.forEach(node => {
                const id = node.community || -1;
                if (!communities[id]) communities[id] = [];
                communities[id].push(node);
            });
            
            const list = document.getElementById('communitiesList');
            list.innerHTML = Object.entries(communities).map(([id, nodes]) => `
                <li onclick="filterByCommunity('${id}')">
                    Community ${id}<br>
                    <small style="opacity:0.7;">${nodes.length} nodes</small>
                </li>
            `).join('');
        }
        
        function setupSearch() {
            document.getElementById('searchInput').addEventListener('input', function(e) {
                const term = e.target.value.toLowerCase();
                if (!term) return;
                
                const results = graphData.nodes.filter(n => 
                    n.label.toLowerCase().includes(term)
                ).slice(0, 20);
                
                const list = document.getElementById('searchResults');
                list.innerHTML = results.map(n => `
                    <li onclick="showNode('${n.id}')">${n.label}</li>
                `).join('');
            });
        }
        
        function showNode(nodeId) {
            const node = graphData.nodes.find(n => n.id === nodeId);
            if (!node) return;
            
            const incoming = graphData.edges.filter(e => e.target === nodeId);
            const outgoing = graphData.edges.filter(e => e.source === nodeId);
            
            const degree = incoming.length + outgoing.length;
            
            let html = `
                <div class="node-info">
                    <h4>${node.label}</h4>
                    <div class="info-row">
                        <span class="info-label">File:</span>
                        <span class="info-value">${node.source_file}</span>
                    </div>
                    <div class="info-row">
                        <span class="info-label">Location:</span>
                        <span class="info-value">${node.source_location}</span>
                    </div>
                    <div class="info-row">
                        <span class="info-label">Community:</span>
                        <span class="info-value">${node.community || 'N/A'}</span>
                    </div>
                    <div class="info-row">
                        <span class="info-label">Degree:</span>
                        <span class="info-value">${degree}</span>
                    </div>
                </div>
                
                <h4>Incoming (${incoming.length})</h4>
                <div class="edges-list">
                    ${incoming.map(e => {
                        const source = graphData.nodes.find(n => n.id === e.source);
                        return `
                            <div class="edge-item">
                                <span class="edge-label">${source.label}</span>
                                <span class="confidence ${e.confidence.toLowerCase()}">${e.confidence}</span>
                                <br><small>${e.relation}</small>
                            </div>
                        `;
                    }).join('')}
                </div>
                
                <h4 style="margin-top:20px;">Outgoing (${outgoing.length})</h4>
                <div class="edges-list">
                    ${outgoing.map(e => {
                        const target = graphData.nodes.find(n => n.id === e.target);
                        return `
                            <div class="edge-item">
                                <span class="edge-label">${target.label}</span>
                                <span class="confidence ${e.confidence.toLowerCase()}">${e.confidence}</span>
                                <br><small>${e.relation}</small>
                            </div>
                        `;
                    }).join('')}
                </div>
            `;
            
            document.getElementById('nodeDetails').innerHTML = html;
        }
        
        function filterByCommunity(commId) {
            // TODO: Filter sidebar to show only nodes in community
        }
        
        function clearAll() {
            graphData = null;
            document.getElementById('graphFile').value = '';
            document.getElementById('nodeDetails').innerHTML = '';
            document.getElementById('stats').innerHTML = '';
        }
    </script>
</body>
</html>
```

**Use it:**
```bash
# Copy to your graphify-out directory
cp graph_explorer.html graphify-out/

# Open in browser
open graphify-out/graph_explorer.html

# Then drag and drop graph.json onto the page
```

### Example 3: Jupyter Notebook Query Tool

```python
# Run in Jupyter notebook

import json
from pathlib import Path
from collections import defaultdict

# Load graph
with open('graphify-out/graph.json') as f:
    graph = json.load(f)

# Create lookup tables
nodes_by_id = {n['id']: n for n in graph['nodes']}
nodes_by_label = {n['label']: n for n in graph['nodes']}

# Query functions
def explain(label):
    """Get full info about a node."""
    node = nodes_by_label.get(label)
    if not node:
        print(f"Node '{label}' not found")
        return
    
    incoming = [e for e in graph['edges'] if e['target'] == node['id']]
    outgoing = [e for e in graph['edges'] if e['source'] == node['id']]
    
    print(f"📍 {node['label']}")
    print(f"   Source: {node['source_file']} {node['source_location']}")
    print(f"   Community: {node.get('community', 'N/A')}")
    print(f"   Degree: {len(incoming) + len(outgoing)}")
    
    if incoming:
        print(f"\n  ← Incoming ({len(incoming)}):")
        for e in incoming[:5]:
            src = nodes_by_id[e['source']]['label']
            print(f"     {src} [{e['relation']}]")
    
    if outgoing:
        print(f"\n  → Outgoing ({len(outgoing)}):")
        for e in outgoing[:5]:
            tgt = nodes_by_id[e['target']]['label']
            print(f"     {tgt} [{e['relation']}]")

def god_nodes(limit=10):
    """Show top god nodes."""
    degree = {}
    for e in graph['edges']:
        degree[e['source']] = degree.get(e['source'], 0) + 1
        degree[e['target']] = degree.get(e['target'], 0) + 1
    
    sorted_nodes = sorted(degree.items(), key=lambda x: x[1], reverse=True)[:limit]
    
    print("👑 God Nodes:\n")
    for node_id, deg in sorted_nodes:
        node = nodes_by_id[node_id]
        print(f"  {node['label']:30} (degree: {deg:3}, community: {node.get('community', 'N/A')})")

# Use it!
explain('HTTPServer')
god_nodes()
```

---

## Part 3: Command-Line Examples

### Generate a Quick Report

```bash
#!/bin/bash
# save as analyze.sh

GRAPH="graphify-out/graph.json"

echo "=== Graph Analysis Report ==="
echo
echo "File Size: $(du -h $GRAPH | cut -f1)"
echo "Nodes: $(jq '.nodes | length' $GRAPH)"
echo "Edges: $(jq '.edges | length' $GRAPH)"
echo
echo "God Nodes:"
jq '.edges | group_by(.source) | map({id: .[0].source, count: length}) | sort_by(.count) | reverse | .[0:5] | .[] | "\(.id): \(.count)"' $GRAPH
```

Run it:
```bash
chmod +x analyze.sh
./analyze.sh
```

---

## Part 4: Integration Examples

### With make/Makefile

```makefile
# Makefile

.PHONY: graph-gen graph-explore graph-analyze

graph-gen:
	@echo "Generating graph..."
	graphify .

graph-explore:
	@echo "Opening explorer..."
	open graphify-out/graph_explorer.html

graph-analyze:
	@echo "Running analysis..."
	python analyze.py graphify-out/graph.json

graph-all: graph-gen graph-analyze graph-explore
```

Usage:
```bash
make graph-all
```

---

## Part 5: Real-World Example Projects

### Django Project Example

```python
# django_analysis.py
import json
from pathlib import Path

with open('graphify-out/graph.json') as f:
    graph = json.load(f)

# Find models
models = [n for n in graph['nodes'] if 'models.py' in n['source_file']]
print(f"Models found: {len(models)}")

# Find views
views = [n for n in graph['nodes'] if 'views.py' in n['source_file']]
print(f"Views found: {len(views)}")

# Model-to-view connections
for view in views[:5]:
    connections = [e for e in graph['edges'] if e['source'] == view['id']]
    connected_models = [c for c in connections if any(c['target'] == m['id'] for m in models)]
    if connected_models:
        print(f"  {view['label']} uses {len(connected_models)} models")
```

---

## Tips for Interactive Learning

1. **Modify & experiment:** Change thresholds, filters, rankings
2. **Combine tools:** Use Python script + HTML visualizer
3. **Automate:** Add to CI for continuous graph analysis
4. **Share:** Send the HTML explorer to teammates
5. **Version:** Keep historical graphs to track evolution

---

Next: [Video Transcript Guidelines](VIDEO_TRANSCRIPTS.md) for creating educational content!
