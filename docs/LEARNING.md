# Interactive Learning: Hands-On Experiments

Get comfortable with graphify by running commands and observing the output. Each exercise teaches a pattern you'll use repeatedly.

## Prerequisites

- `graphify` installed and working
- A `graph.json` file (run `/graphify .` in your AI assistant to create one)
- A terminal

## Exercise 1: Understand Your Graph

**Goal:** Get a feel for the size and shape of your graph.

```bash
# Count nodes and edges
graphify stats graphify-out/graph.json

# Example output:
# Nodes:        247
# Edges:        1,234
# Communities:  8
# Density:      0.02
```

**Questions to ask:**
- How many nodes did your codebase generate? (Small = simple; large = complex)
- How many communities? (Few = tightly coupled; many = well-separated)
- What's the density? (0.0 = no edges; 1.0 = fully connected; most code is sparse)

---

## Exercise 2: Find God Nodes

**Goal:** Identify the busiest concepts in your codebase.

```bash
# List god nodes (high degree)
graphify explain --god-nodes graphify-out/graph.json

# Or ask for top 10:
graphify explain --god-nodes --limit 10 graphify-out/graph.json
```

**What you'll see:**
```
God Nodes (by degree):

1. requests (degree: 47)
   Source: http_client.py:24
   Community: 2 (HTTP Utilities)

2. BaseModel (degree: 43)
   Source: models.py:5
   Community: 1 (Data Structures)

3. validate() (degree: 38)
   Source: validators.py:12
   Community: 1 (Data Structures)
```

**Experiments:**
- **Pick the top god node.** Would splitting it into smaller pieces improve your code?
- **Find a god node you didn't expect.** Why is it central? Is that a bug or a feature?
- **Compare communities.** Which subsystem has the most god nodes? (Often indicates tight coupling.)

---

## Exercise 3: Trace a Path

**Goal:** Find how two distant concepts are connected.

```bash
# Trace the shortest path between two things
graphify path "APIRouter" "database" graphify-out/graph.json

# Or between two specific functions:
graphify path "authenticate()" "UserModel" graphify-out/graph.json
```

**What you'll see:**
```
Shortest path (4 hops):

  APIRouter
    --imports--> Request [EXTRACTED]
  Request
    --references--> User [INFERRED]
  User
    --calls--> validate_token() [EXTRACTED]
  validate_token()
    --uses--> database [EXTRACTED]
```

**Experiments:**
- **Trace a known dependency.** Does the path match what you expected?
- **Trace something you're unsure about.** Does this explain a bug you've been chasing?
- **Trace between modules from different communities.** These are your cross-boundary dependencies—often the stickiest points.

---

## Exercise 4: Explain a Node

**Goal:** Understand what a concept does by looking at its neighborhood.

```bash
# Get the full picture for a node
graphify explain "FastAPI" graphify-out/graph.json

# Or just incoming edges (who uses this?):
graphify explain "FastAPI" graphify-out/graph.json --incoming

# Or just outgoing edges (what does this use?):
graphify explain "FastAPI" graphify-out/graph.json --outgoing
```

**What you'll see:**
```
Node: FastAPI
  Source:    main.py:5
  Community: 3 (API Core)
  Degree:    23

Incoming edges (11):
  <-- APIRouter [imports] [EXTRACTED]
  <-- middleware [uses] [EXTRACTED]
  <-- error_handler() [references] [INFERRED]
  ...

Outgoing edges (12):
  --> Request [imports] [EXTRACTED]
  --> Response [imports] [EXTRACTED]
  --> mount() [calls] [EXTRACTED]
  ...
```

**Experiments:**
- **Pick a core function.** Who depends on it? Changing it might break all those things.
- **Pick a function you're unfamiliar with.** Seeing its neighborhood teaches you its role.
- **Find a node with many incoming edges and few outgoing.** It's a leaf—safe to modify.

---

## Exercise 5: Find Communities

**Goal:** Understand how your codebase is organized.

```bash
# List all communities
graphify communities graphify-out/graph.json

# Or a specific community:
graphify communities graphify-out/graph.json --community 2
```

**What you'll see:**
```
Community 0: HTTP Utilities (12 nodes)
  God nodes: HTTPServer, Request, Response
  Density: 0.08
  External edges: 7 (connected to communities 1, 3)

Community 1: Data Structures (18 nodes)
  God nodes: BaseModel, Schema, Validator
  Density: 0.11
  External edges: 12 (connected to communities 0, 2, 3)

Community 2: Authentication (8 nodes)
  God nodes: TokenValidator, User
  Density: 0.15
  External edges: 4 (connected to communities 0, 1)
```

**Experiments:**
- **Is the community structure what you expected?** If not, does that suggest a refactoring?
- **Find a community with high external edges.** It's coupled to other parts; might be a candidate for extraction.
- **Find a community with high internal density.** It's well-defined; good architecture candidate.

---

## Exercise 6: Query a Question

**Goal:** Ask the graph a natural language question.

```bash
# Simple question
graphify query "How does authentication work?" graphify-out/graph.json

# More specific
graphify query "What endpoints are exposed?" graphify-out/graph.json

# Looking for a pattern
graphify query "Where is validation performed?" graphify-out/graph.json
```

**What you'll see:**
```
Query: "How does authentication work?"

Relevant nodes (ranked by centrality):
1. TokenValidator (degree: 12, community: 2)
2. User (degree: 8, community: 2)
3. validate_token() (degree: 6, community: 2)
4. middleware (degree: 15, community: 0)
5. Request (degree: 20, community: 0)

Suggested path: TokenValidator -> User -> validate_token() -> middleware
```

**Experiments:**
- **Ask about a system component you understand.** Does the query rank the right nodes?
- **Ask about something you're learning.** Does it guide you to the right starting point?
- **Refine your question.** "How does authentication work?" is broad; try "How are tokens validated?"

---

## Exercise 7: Compare Graphs Over Time

**Goal:** See how your codebase is evolving.

```bash
# Run graphify on your project
/graphify .    (in your AI assistant)

# Save the resulting graph with a date
cp graphify-out/graph.json graphify-out/graph.2024-01.json

# Later, after making changes:
/graphify .

# Compare
graphify compare graphify-out/graph.2024-01.json graphify-out/graph.json
```

**What you'll see:**
```
Comparison: graph.2024-01.json → graph.json

Nodes added: 23
Nodes removed: 5
Edges added: 47
Edges removed: 12

New god nodes:
  - APIv2Handler (degree: 18)
  - MiddlewareRegistry (degree: 14)

Removed concepts:
  - DeprecatedValidator
  - OldAuthScheme

Community changes:
  - Community 1: 12→18 nodes (expanded)
  - Community 3: 8→6 nodes (shrank)
```

**Experiments:**
- **Track how a refactor affects your graph.** Did you reduce coupling? Did a new bottleneck appear?
- **Monitor new god nodes.** If one appears suddenly, is it intentional?
- **Validate architectural decisions.** "After extracting the auth module, are edges between it and core reduced?"

---

## Exercise 8: Dive Into a Community

**Goal:** Understand a subsystem deeply.

```bash
# Export just one community to a subgraph
graphify export --community 2 graphify-out/graph.json graphify-out/community-2.json

# Now explain the nodes in that community
graphify explain --god-nodes graphify-out/community-2.json

# Or query within it
graphify query "How do users flow through this?" graphify-out/community-2.json
```

**Experiments:**
- **Pick the largest community.** Is it too big? Should it be split?
- **Pick a small community.** Is it well-isolated? Could it be a standalone library?
- **Look at cross-community edges.** Are they necessary, or could they be eliminated?

---

## Exercise 9: Benchmark Your Graph

**Goal:** See how well graphify captures your codebase complexity.

```bash
# Run a token-comparison benchmark
graphify benchmark graphify-out/graph.json

# Output: comparison of token counts when answering questions
# on the full corpus vs. the graph
```

**What you'll see:**
```
Benchmark Results:

Full corpus (grepping/reading all files):
  Average tokens per question: 5,432
  Max tokens (one question): 12,104

Subgraph (using graphify queries):
  Average tokens per question: 892
  Max tokens (one question): 2,341

Compression: 6.1x fewer tokens
Trust: Questions answered without losing context
```

---

## Exercise 10: Customize and Tinker

**Goal:** Make graphify your own.

See [Developer Guide](DEVELOPER.md) for:
- Tweaking extraction rules
- Adding a new language
- Customizing node/edge types
- Building a personal "knowledge vault"

---

## Patterns You've Learned

| Pattern | Command | Use Case |
|---------|---------|----------|
| **Understand size** | `graphify stats` | Gauge codebase complexity |
| **Find bottlenecks** | `graphify explain --god-nodes` | Identify high-leverage refactoring targets |
| **Trace dependencies** | `graphify path A B` | Debug coupling issues |
| **Neighborhood analysis** | `graphify explain NODE` | Understand a concept's role |
| **Community structure** | `graphify communities` | Validate architecture |
| **Natural language queries** | `graphify query "..."` | Ask questions in prose |
| **Monitor changes** | `graphify compare` | Track refactoring progress |
| **Subsystem deep-dives** | `graphify export --community` | Focus on one part |
| **Benchmark impact** | `graphify benchmark` | Measure graph fidelity |

---

## Next Steps

- **[Cookbook](COOKBOOK.md)** — Real-world scenarios combining these patterns
- **[Developer Guide](DEVELOPER.md)** — Customize extraction, build tools on top
- **Open `graph.html`** in your browser and play! Click nodes, search, drag, zoom.

---

## Tips for Learning

1. **Run commands with your own codebase.** The patterns are universal, but seeing your own data is 10x more instructive.
2. **Start with `stats` and `--god-nodes`.** They give you a lay of the land.
3. **Then explore specific nodes** that surprise you with `explain`.
4. **Then trace paths** between concepts you understand.
5. **Finally, ask questions** you actually have about your code.

Good luck! 🚀
