# Visual Architecture Guide

Visual explanations of how graphify works, from high-level concepts to pipeline details.

---

## 1. The Three Layers (Overview)

```
┌─────────────────────────────────────────────────────────────────┐
│ Your Codebase (Text Files)                                      │
│  ├─ main.py                                                      │
│  ├─ utils.js                                                     │
│  ├─ README.md                                                    │
│  └─ config.yaml                                                  │
└────────────────────┬────────────────────────────────────────────┘
                     │ Parse with tree-sitter
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ Layer 1: Explicit Structure (AST)                               │
│                                                                  │
│  Functions:    def foo():, function bar() {}                    │
│  Classes:      class User:, class ApiResponse {}                │
│  Imports:      import X from Y, from Z import W                 │
│  Calls:        foo(); bar.method()                              │
│  Inherits:     class Child(Parent), extends Base                │
│                                                                  │
│  Confidence: EXTRACTED (100% from syntax)                       │
└────────────────────┬────────────────────────────────────────────┘
                     │ Graph algorithms
                     │ (centrality, clustering)
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ Layer 2: Inferred Structure (Graph Analysis)                    │
│                                                                  │
│  Bottlenecks:  "foo() is a bottleneck" (degree: 47)             │
│  Subsystems:   "These 8 functions form a cohesive unit"         │
│  Paths:        "Here's how A reaches B"                         │
│  Patterns:     "These two functions are suspiciously similar"   │
│                                                                  │
│  Confidence: INFERRED (deduced from structure)                  │
└────────────────────┬────────────────────────────────────────────┘
                     │ LLM (docs, PDFs, images)
                     ▼
┌─────────────────────────────────────────────────────────────────┐
│ Layer 3: Semantic Meaning (LLM-Enhanced)                        │
│                                                                  │
│  Documentation:  "This validates user input"                    │
│  Purpose:        "This is an internal helper"                   │
│  Related Work:   "Mentioned in RFC-042"                         │
│  Examples:       "See example in tutorial.md"                   │
│                                                                  │
│  Confidence: AMBIGUOUS (requires interpretation)                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. The Pipeline (Step by Step)

```
                    ┌──────────────┐
                    │  START: Root │
                    │   Directory  │
                    └──────┬───────┘
                           │
                           ▼
                  ┌─────────────────────┐
            ┌──→ │ STEP 1: DETECT      │ ← Scan filesystem
            │    │ collect_files()     │   Filter by extension
            │    │ Input: Path         │
            │    │ Output: [Path, ...] │
            │    └─────────┬───────────┘
            │              │
        [Optional] │       ▼
        Re-run?  │    ┌──────────────────┐
            │    ├──→ │ STEP 2: EXTRACT  │ ← Tree-sitter parse
            │    │    │ extract()        │   Walk AST
            │    │    │ Input: File path │   Collect nodes & edges
            │    │    │ Output: {nodes,  │
            │    │    │    edges}        │
            │    │    └────────┬─────────┘
            │    │             │ (repeat for each file)
            │    │             │
            │    │             ▼
            │    │    ┌──────────────────────────┐
            │    ├──→ │ STEP 3: BUILD            │ ← Assemble graph
            │    │    │ build_graph()            │   Create nodes
            │    │    │ Input: [{nodes, edges}] │   Add edges
            │    │    │ Output: nx.Graph         │
            │    │    └────────┬────────────────┘
            │    │             │
            │    │             ▼
            │    │    ┌──────────────────────────┐
            │    ├──→ │ STEP 4: CLUSTER          │ ← Leiden algorithm
            │    │    │ cluster()                │   Detect communities
            │    │    │ Input: nx.Graph          │
            │    │    │ Output: Graph with       │
            │    │    │    community attr       │
            │    │    └────────┬────────────────┘
            │    │             │
            │    │             ▼
            │    │    ┌──────────────────────────┐
            │    ├──→ │ STEP 5: ANALYZE          │ ← Compute stats
            │    │    │ analyze()                │   Degree, centrality
            │    │    │ Input: Graph             │   God nodes, surprises
            │    │    │ Output: {analysis}       │
            │    │    └────────┬────────────────┘
            │    │             │
            │    │             ▼
            │    │    ┌──────────────────────────┐
            │    ├──→ │ STEP 6: REPORT           │ ← Render markdown
            │    │    │ render_report()          │   Key concepts
            │    │    │ Input: Graph + Analysis  │   Questions
            │    │    │ Output: GRAPH_REPORT.md  │
            │    │    └────────┬────────────────┘
            │    │             │
            │    │             ▼
            │    │    ┌──────────────────────────┐
            │    ├──→ │ STEP 7: EXPORT           │ ← Write files
            │    │    │ export()                 │   graph.json
            │    │    │ Input: Graph             │   graph.html
            │    │    │ Output: [files]          │   graph.svg
            │    │    └────────┬────────────────┘   vault/
            │    │             │
            │    │             ▼
            │    │    ┌──────────────────────────┐
            └────┴──→ │ END: graphify-out/       │
                     │ ├─ graph.json             │
                     │ ├─ graph.html             │
                     │ └─ GRAPH_REPORT.md        │
                     └──────────────────────────┘
```

---

## 3. Data Flow: A Real Example

```
File: auth.py
┌────────────────────────────────────────────┐
│ def authenticate(user, password):          │ ← Line 10
│     result = validate_password(password)   │
│     if result:                             │
│         create_session(user)               │
│     return result                          │
└────────────────────────────────────────────┘
         │ (tree-sitter parse)
         ▼
Nodes extracted:
  ├─ id: auth.py:authenticate
  │  label: authenticate()
  │  source: auth.py
  │  location: L10
  │
  ├─ id: auth.py:validate_password
  │  label: validate_password()
  │  source: auth.py
  │  location: L15  (somewhere else in file)
  │
  └─ id: auth.py:create_session
     label: create_session()
     source: auth.py
     location: L20

Edges extracted:
  ├─ source: authenticate
     target: validate_password
     relation: calls
     confidence: EXTRACTED
  │
  └─ source: authenticate
     target: create_session
     relation: calls
     confidence: EXTRACTED

    (stored in graph.json)
         ▼
Query result:
  $ graphify explain "authenticate()"
  
  Node: authenticate()
    Source: auth.py L10
    Degree: 3 (incoming: 2, outgoing: 2)
    
  Outgoing edges:
    --> validate_password() [calls] [EXTRACTED]
    --> create_session() [calls] [EXTRACTED]
  
  Incoming edges:
    <-- api_handler() [calls] [EXTRACTED]
    <-- login_endpoint() [calls] [EXTRACTED]
```

---

## 4. Node Types & Attributes

```
┌──────────────────────────────────────────────────┐
│ A Node Represents One Concept                    │
└──────────────────────────────────────────────────┘

Essential Attributes:
  id              : "auth.py:authenticate"
  label           : "authenticate()"
  source_file     : "auth.py"
  source_location : "L10"

Optional Attributes:
  type            : "function" | "class" | "module" | "constant"
  community       : 2 (Leiden community ID)
  degree          : 15 (how many edges it has)
  is_god_node     : true (if degree > threshold)
  description     : "Validates user credentials"
  docstring       : "Authenticate user with password..."

Examples:

┌─────────────────────────────────────────┐
│ Function Node                           │
│                                         │
│ id: "auth.py:validate_password"         │
│ label: "validate_password()"            │
│ type: "function"                        │
│ degree: 12                              │
│ community: 1 (Auth Utilities)           │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Class Node                              │
│                                         │
│ id: "models.py:User"                    │
│ label: "User"                           │
│ type: "class"                           │
│ degree: 47 ← god node!                  │
│ community: 0 (Data Structures)          │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Module Node                             │
│                                         │
│ id: "database"                          │
│ label: "database module"                │
│ type: "module"                          │
│ degree: 23                              │
│ community: 1 (Infrastructure)           │
└─────────────────────────────────────────┘
```

---

## 5. Edge Types

```
┌─────────────────────────────────────────────────────────────┐
│ An Edge Represents One Relationship                         │
└─────────────────────────────────────────────────────────────┘

  source
    │
    ├─[calls]──────────→ function A invokes function B
    │
    ├─[imports]────────→ module A depends on module B
    │
    ├─[uses]───────────→ A uses B (generic dependency)
    │
    ├─[inherits]───────→ class A extends class B
    │
    ├─[references]─────→ code mentions a name
    │
    ├─[defined_in]─────→ this location defines a name
    │
    ├─[mixes_in]───────→ (Ruby/Elixir) module inclusion
    │
    └─[mentions]───────→ comment/doc talks about a concept

Example edges in a real graph:

  APIRouter
    --[imports]──→ Request
    --[uses]──────→ HTTPServer
    --[calls]─────→ route_handler()
    --[references]→ dependency_injection

  Request
    <--[imports]── main.py
    <--[calls]──── route_handler()
    --[references]→ User (type annotation)

Confidence Levels:

  EXTRACTED   ┃ This edge is explicit in code
              ┃ Example: "import X" → 100% confidence
              ┃
  INFERRED    ┃ This edge is deduced from graph structure
              ┃ Example: call-graph second pass
              ┃ Confidence: High (but not 100%)
              ┃
  AMBIGUOUS   ┃ This edge is uncertain
              ┃ Example: dynamic call via reflection
              ┃ Confidence: Low
              ┃ Flagged for manual review
```

---

## 6. Communities (Subsystems)

```
Node Distribution by Community:

Graph (247 nodes total, 8 communities)

 ┌──────────────────────────────────┐
 │ Community 0 (HTTP Utilities)      │
 │ ════════════════════════════════ │
 │ 23 nodes                         │
 │ Internal edges: 89               │
 │ External edges: 7                │
 │ Density: 0.15 (well-connected)   │
 │                                  │
 │ God nodes:                       │
 │   ◆ HTTPServer (degree 24)       │
 │   ◆ Request (degree 18)          │
 │   ◆ Response (degree 15)         │
 └──────────────────────────────────┘

 ┌──────────────────────────────────┐
 │ Community 1 (Data Structures)     │
 │ ════════════════════════════════ │
 │ 34 nodes                         │
 │ Internal edges: 127              │
 │ External edges: 12               │
 │ Density: 0.11 (moderate)         │
 │                                  │
 │ God nodes:                       │
 │   ◆ BaseModel (degree 43)        │
 │   ◆ Schema (degree 22)           │
 │   ◆ Validator (degree 19)        │
 └──────────────────────────────────┘

 ┌──────────────────────────────────┐
 │ Community 2 (Authentication)      │
 │ ════════════════════════════════ │
 │ 12 nodes                         │
 │ Internal edges: 34               │
 │ External edges: 4                │
 │ Density: 0.23 (tight!)           │
 │                                  │
 │ God nodes:                       │
 │   ◆ TokenValidator (degree 12)   │
 │   ◆ User (degree 8)              │
 └──────────────────────────────────┘

 [... more communities ...]

God nodes at a glance:

        Degree
  47 ◆ BaseModel (Community 1)
  24 ◆ HTTPServer (Community 0)
  23 ◆ Schema (Community 1)
  19 ◆ Validator (Community 1)
  18 ◆ Request (Community 0)
  15 ◆ Response (Community 0)
  12 ◆ TokenValidator (Community 2)
  ...
```

---

## 7. Query Execution (Under the Hood)

```
Query: "How does authentication work?"

┌──────────────────────────────────────┐
│ INPUT: Question (natural language)   │
└──────────────┬───────────────────────┘
               │ (LLM embedding)
               ▼
┌──────────────────────────────────────┐
│ EMBEDDING QUERY:                     │
│ "authentication, token, user,        │
│  validate, session, credential"      │
└──────────────┬───────────────────────┘
               │ (graph search)
               ▼
┌──────────────────────────────────────────────────────┐
│ MATCHED NODES (by relevance):                        │
│                                                      │
│ 1. TokenValidator      (degree 12) - perfect match  │
│ 2. User                (degree 8)  - token mentions │
│ 3. Session             (degree 6)  - auth context   │
│ 4. validate_token()    (degree 4)  - explicit match │
│ 5. Middleware          (degree 15) - auth layer     │
│                                                      │
└──────────────┬───────────────────────────────────────┘
               │ (extract subgraph)
               ▼
┌──────────────────────────────────────────────────────┐
│ SUBGRAPH (relevant nodes + their edges):             │
│                                                      │
│ TokenValidator --uses--> User                       │
│ TokenValidator --calls--> validate_credentials()    │
│ validate_credentials() --references--> Token        │
│ Middleware --imports--> TokenValidator              │
│ Session --defines--> SessionID                      │
│                                                      │
└──────────────┬───────────────────────────────────────┘
               │ (format output)
               ▼
┌──────────────────────────────────────────────────────┐
│ OUTPUT (to user):                                    │
│                                                      │
│ Relevant nodes:                                      │
│   1. TokenValidator (degree 12)                      │
│   2. User (degree 8)                                │
│   3. Session (degree 6)                             │
│   ...                                                │
│                                                      │
│ Suggested path:                                      │
│   TokenValidator -> User -> validate_credentials()  │
│   -> Token -> Session                               │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 8. God Nodes: Why They Matter

```
Low-degree node (leaf):
┌─────────────────────┐
│ cleanup_temp_files()│
└─────────┬───────────┘
          │ (degree: 1)
          │
          ▼
    Very safe to modify!
    Only 1 call site depends on it.

High-degree node (god node):
┌─────────────────────┐
│ utils.py:validate() │ ◆ BOTTLENECK
└─────────┬───────────┘
          │ (degree: 47!)
    ┌─────┼─────┬─────┬─────┬─────┬─────────...
    │     │     │     │     │     │
    ▼     ▼     ▼     ▼     ▼     ▼
  form  api  email cache render settings
  
  Risk: Modifying validate() breaks 47 functions!
  
Why god nodes are important:
  ├─ Bottlenecks (fix here = fix many things)
  ├─ Coupling hubs (high impact on architecture)
  ├─ Refactoring targets (split/delegate)
  ├─ Testing priorities (ensure it works!)
  └─ Documentation focus (everyone needs to understand it)

Strategy for god nodes:
  
  If degree is high because it's a GOOD abstraction:
    → Keep it! Don't split it.
    → Document it well (everyone depends on it).
    → Write comprehensive tests.
  
  If degree is high because of POOR design:
    → Split it into smaller, focused functions.
    → Reduce coupling by using dependency injection.
    → Create smaller, more specialized god nodes.
```

---

## 9. Confidence Visualization

```
Edge Trustworthiness:

EXTRACTED (explicit syntax)
   ═══════════════════════════════════════════════
   │ import X from Y                       100%
   │ function foo() { bar() }              100%
   │ class Child extends Parent            100%
   └─ Source: The code literally says this

INFERRED (deduced from structure)
   ═════════════════════════════════════════════
   │ resolve_calls() second pass           95%
   │ type annotation references             92%
   │ pattern matching (similar names)       80%
   └─ Source: Graph structure + heuristics

AMBIGUOUS (uncertain)
   ═══════════════════════════════════════════
   │ dynamic eval() calls                  20%
   │ reflection/metaprogramming            15%
   │ string-based lookups                  10%
   └─ Source: Need human review

Rule of thumb:
  EXTRACTED edges = trust 100%, use directly
  INFERRED edges = trust 90%+, good for queries
  AMBIGUOUS edges = flag for review, don't rely solely on
```

---

## 10. Memory & Performance

```
Typical Project Sizes:

Small (< 5k nodes):
  ├─ Time: 30 seconds - 2 minutes
  ├─ Memory: 100-500 MB
  ├─ Storage: graph.json ~5-20 MB
  └─ Example: Single module, <10k files

Medium (5-50k nodes):
  ├─ Time: 2-20 minutes
  ├─ Memory: 500 MB - 2 GB
  ├─ Storage: graph.json ~50-500 MB
  └─ Example: Large codebase, 10-100k files

Large (50k+ nodes):
  ├─ Time: 20+ minutes (or timeout)
  ├─ Memory: 2+ GB
  ├─ Storage: graph.json 500+ MB
  └─ Example: Huge monorepo, 100k+ files
  
  Solution: Run on subsets
    graphify ./src          (just source)
    graphify --code-only    (no semantic extraction)
    graphify --exclude "test,vendor"

Performance bottleneck:
  1. Semantic extraction (docs/PDFs) ← Usually slowest
  2. Leiden clustering (large graphs) ← Quadratic
  3. File reading (many small files) ← I/O
  4. Tree-sitter parsing (rarely, with huge files) ← CPU

If slow:
  ├─ Try --code-only (skip semantic)
  ├─ Try --exclude "large_vendor_dir"
  ├─ Run on subset of codebase
  └─ Check available memory
```

---

## Summary: The Architecture from 10,000 Feet

```
┌────────────────────────────────────────────────────┐
│                 Your Codebase                      │
│        (files, classes, functions, modules)        │
└──────────────────┬─────────────────────────────────┘
                   │
                   │ (7-step pipeline)
                   ▼
┌────────────────────────────────────────────────────┐
│              Knowledge Graph                       │
│  • 247 nodes (concepts)                            │
│  • 1,234 edges (relationships)                     │
│  • 8 communities (subsystems)                      │
│  • 12 god nodes (bottlenecks)                      │
│                                                    │
│  Every relationship has:                           │
│  - Explicit source location (traceable)            │
│  - Confidence tag (EXTRACTED/INFERRED/AMBIGUOUS)   │
│  - Relation type (calls/imports/inherits/...)      │
└──────────────────┬─────────────────────────────────┘
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
    graph.json  graph.html  GRAPH_REPORT.md
    (query)     (explore)   (insights)
```

---

Next: [Interactive Learning](LEARNING.md) with real commands and output!
