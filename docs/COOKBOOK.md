# Cookbook: Real-World Scenarios

Step-by-step recipes for common tasks. Each scenario builds on the concepts and patterns you've learned.

---

## Scenario 1: "I Need to Add a Feature. Where Do I Start?"

**The Problem:**
You need to add a new feature but don't know the codebase well. You want to avoid breaking things.

**The Recipe:**

### Step 1: Map the codebase
```bash
/graphify .    (in your AI assistant)
```

### Step 2: Find the relevant subsystem
```bash
# If your feature is about "user authentication":
graphify query "Where is user authentication handled?" graphify-out/graph.json
```

Example output:
```
Relevant nodes:
1. AuthMiddleware (degree: 15)
2. TokenValidator (degree: 12)
3. User model (degree: 8)
```

### Step 3: Dive into that community
```bash
# Identify which community they belong to
graphify explain "AuthMiddleware" graphify-out/graph.json

# Output shows it's in Community 2. Export it:
graphify export --community 2 graphify-out/graph.json graphify-out/auth-subsystem.json
```

### Step 4: Understand the interfaces
```bash
# See what depends on this subsystem
graphify explain "AuthMiddleware" graphify-out/auth-subsystem.json --incoming

# This tells you: "These functions depend on AuthMiddleware"
# So if you change AuthMiddleware, these break.
```

### Step 5: Plan your change
- **Adding to an existing node?** Safe if that node has few incoming edges
- **Creating a new node?** Check if you need to link it to god nodes (might cause unexpected side effects)
- **Modifying a god node?** Dangerous—many things depend on it

### Step 6: After you code
```bash
# Re-run graphify to see the new graph
/graphify .

# Compare to the old one
graphify compare graphify-out/graph.backup.json graphify-out/graph.json

# Expected: your feature adds nodes to the auth community, minimal external edges
# Unexpected: nodes added to 5 different communities (sign of poor design)
```

---

## Scenario 2: "There's a Bug. Trace It."

**The Problem:**
User reports a bug when doing X. You need to find where in the code this breaks.

**The Recipe:**

### Step 1: Identify the entry point
The bug happens when user does X. What code does X invoke?

Example: User uploads a file, gets a 500 error.
```bash
graphify query "How are file uploads handled?" graphify-out/graph.json

# Output might point to: upload_handler() in file_service.py
```

### Step 2: Trace the execution path
```bash
# From upload_handler, where does it go?
graphify explain "upload_handler()" graphify-out/graph.json --outgoing

# Output: upload_handler calls:
#   - validate_file()
#   - save_to_storage()
#   - index_database()
#   - notify_user()
```

### Step 3: Narrow down based on the error
- **500 error** = likely a crash, not validation failure
- **Permission denied** = likely in save_to_storage() or database
- **Timeout** = likely in a long-running call like index_database()

```bash
# Trace to the most likely culprit
graphify path "upload_handler()" "database_write()" graphify-out/graph.json

# Shows the chain: upload_handler -> save_to_storage -> database_write
# If the error is "database locked", you found it
```

### Step 4: Check for recent changes
```bash
# Compare current graph to one from last week
graphify compare graphify-out/graph.last-week.json graphify-out/graph.json

# If upload_handler or its dependencies changed recently, that's suspicious
```

### Step 5: Reproduce and fix
- You now know the code path to the bug
- Read the implementation in those 3-4 functions
- Fix the bug
- Re-run graphify to ensure you didn't introduce new issues

---

## Scenario 3: "This Code is a Mess. How Do I Refactor It?"

**The Problem:**
You have a messy module or subsystem. You need to understand it before refactoring.

**The Recipe:**

### Step 1: Identify the problematic module
```bash
# Let's say you're refactoring the "auth" module
graphify export --community 2 graphify-out/graph.json graphify-out/auth-old.json

# Or, by node name:
graphify explain "auth_module" graphify-out/graph.json
```

### Step 2: Check internal health
```bash
# How coupled are nodes within this module?
graphify stats graphify-out/auth-old.json

# Expected (healthy): 
#   Community density: 0.15 (well-connected internally)
#   External edges: 3-5 (minimal external coupling)
#
# Bad sign:
#   Community density: 0.02 (sparse—should be tighter)
#   External edges: 20 (too much coupling outside)
```

### Step 3: Find god nodes within the module
```bash
graphify explain --god-nodes graphify-out/auth-old.json
```

These are the nodes to refactor around:
- If `TokenValidator` has degree 20, splitting it up would reduce internal complexity
- If multiple nodes have degree 10+, the responsibilities are unclear

### Step 4: Identify communication patterns
```bash
# What are the most common edge types?
graphify explain --edge-types graphify-out/auth-old.json

# Output might show:
#   calls: 45 edges (normal function calls)
#   references: 32 edges (code references this type)
#   imports: 8 edges (module-level imports)
#
# If "references" is high, you have lots of type/constant coupling
```

### Step 5: Plan your refactor
Example plan based on findings:
- Extract `TokenValidator` into its own module (god node with degree 20)
- Reduce "references" edges by making interfaces more explicit
- Consolidate the 5 helper functions into a Validator class

### Step 6: Validate the refactor
```bash
# After refactoring, re-run graphify
/graphify .

# Compare:
graphify compare graphify-out/auth-old.json graphify-out/graph.json

# Expected improvements:
#   Nodes in auth community: same or fewer (consolidation)
#   External edges: fewer (reduced coupling)
#   God nodes: more balanced (no single 20-degree bottleneck)
#
# Bad sign:
#   External edges: more (you made it worse)
#   New god nodes: architecture didn't improve
```

---

## Scenario 4: "I'm Adding a New Subsystem. How Do I Design It?"

**The Problem:**
You're designing a new subsystem (e.g., a payment processor). You want to avoid tight coupling.

**The Recipe:**

### Step 1: Understand the boundaries
```bash
# What does your new subsystem need to talk to?
# Example: payment system needs users, orders, notifications

graphify query "What are the key entities in user management?" graphify-out/graph.json
graphify query "What are the key entities in order management?" graphify-out/graph.json
```

### Step 2: Find the minimal interface
```bash
# For each dependency, find the god node you MUST talk to
graphify explain "User model" graphify-out/graph.json --incoming

# Output tells you: "These 12 functions import User model"
# Your payment system should only depend on 1-2 of them, not all 12
```

### Step 3: Design your module to be a good citizen
When you build your payment module:
- Depend on the god nodes (they're stable APIs)
- Don't depend on internal helpers (they might change)
- Create your own god nodes as stable entry points

### Step 4: Before you commit, verify the design
```bash
# After you write the payment module, run graphify
/graphify .

# The payment module should:
#   - Have 0-3 external dependencies (incoming edges to User, Order, etc.)
#   - Have its own god node(s) that others can safely depend on
#   - Form its own tight community (internal density high, external low)
#
# Bad signs:
#   - Payment nodes scattered across 5 communities (no cohesion)
#   - Payment god node has degree 1 (not a real interface)
#   - Payment imports 10 different internal helpers (brittle design)
```

---

## Scenario 5: "My Tests Are Breaking. What Changed?"

**The Problem:**
A refactor broke tests, but the error isn't clear. You need to see what dependencies changed.

**The Recipe:**

### Step 1: Run graphify before and after
```bash
# Before the refactor (assuming you have an old graph)
cp graphify-out/graph.json graphify-out/graph.before-refactor.json

# After the refactor
/graphify .

# Compare
graphify compare graphify-out/graph.before-refactor.json graphify-out/graph.json
```

### Step 2: Look for removed edges
```
Edges removed:
  - test_utils [imports] -> mock_factory
  - integration_test [references] -> legacy_validator
  - conftest [uses] -> old_auth_setup
```

These are the dependencies your tests lost. If your tests fail, one of these removals broke them.

### Step 3: Fix the gap
```bash
# Check if there's a replacement
graphify explain "mock_factory" graphify-out/graph.json

# If it's gone, you have two options:
# 1. Restore the old dependency (revert the refactor)
# 2. Update your tests to use the new API
```

---

## Scenario 6: "This Library Should Be Independent. Is It?"

**The Problem:**
You have a library (e.g., `auth_lib/`) that should be extracted into a separate package. Verify it has no external dependencies.

**The Recipe:**

### Step 1: Export the library's community
```bash
# First, figure out which nodes belong to auth_lib
graphify query "What is in the auth library?" graphify-out/graph.json

# Then export just that community
graphify export --pattern "auth_lib*" graphify-out/graph.json graphify-out/auth-lib.json
```

### Step 2: Check for external edges
```bash
# See if anything outside auth_lib imports from it
graphify explain "auth_lib" graphify-out/graph.json --incoming

# Expected (for a library):
#   - Many incoming edges (others use the library)
#   - No outgoing edges (library is independent)
#
# Bad sign (coupled library):
#   - Outgoing edges to core modules (library depends on main app)
#   - Circular dependencies (library imports from modules that import from it)
```

### Step 3: Remove external dependencies
```bash
# For each outgoing edge from the library, decide:
# Can we pass it as a parameter (dependency injection)?
# Can we move the code into the library?
# Or is it an actual hard dependency (keep it)?
```

### Step 4: Verify after extraction
```bash
# After refactoring to remove external deps:
/graphify .

graphify explain "auth_lib" graphify-out/graph.json --outgoing

# Should now show: 0 outgoing edges (or only explicit dependencies)
```

---

## Scenario 7: "How Similar Are These Two Functions?"

**The Problem:**
You suspect two functions do similar things. Should you merge them?

**The Recipe:**

### Step 1: Compare their neighborhoods
```bash
graphify explain "function_a()" graphify-out/graph.json
graphify explain "function_b()" graphify-out/graph.json

# Look at:
#   - Do they import the same modules?
#   - Do they call the same helpers?
#   - Are they in the same community?
```

### Step 2: Trace their call sites
```bash
# Who calls function_a?
graphify explain "function_a()" graphify-out/graph.json --incoming

# Who calls function_b?
graphify explain "function_b()" graphify-out/graph.json --incoming

# If the same callers use both, they might be duplicates
```

### Step 3: Decide on merging
- **Same community + same callers + same dependencies = merge them**
- **Different communities + different callers = leave them separate**
- **Same behavior, different names = rename one and remove the duplicate**

---

## Scenario 8: "How Do I Onboard Someone to This Codebase?"

**The Problem:**
A new team member needs to understand the architecture. Instead of a 2-hour lecture, use the graph.

**The Recipe:**

### Step 1: Send them the graph
```bash
# Generate the graph
/graphify .

# Send them:
#   - graphify-out/graph.html (open in browser—interactive!)
#   - graphify-out/GRAPH_REPORT.md (read for context)
#   - This cookbook
```

### Step 2: Guide them through the exploration
Have them:
1. Open `graph.html` and play with it (5 min)
2. Read `GRAPH_REPORT.md` (10 min)
3. Run `graphify explain --god-nodes` and pick the top 3 (5 min)
4. For each god node, run `graphify explain NODE` (10 min each)
5. Ask a question they have and run `graphify query` (5 min)

### Step 3: Assign their first task
Now they understand the architecture. Assign a small task in a leaf node (few dependencies).

---

## Scenario 9: "I'm Optimizing for Performance. Where's the Bottleneck?"

**The Problem:**
The app is slow. You need to find the hot path through the code.

**The Recipe:**

### Step 1: Identify the slow operation
```bash
# Let's say "uploading a file is slow"
graphify query "What happens when a file is uploaded?" graphify-out/graph.json
```

### Step 2: Trace the call chain
```bash
graphify path "upload_handler()" "database_write()" graphify-out/graph.json
```

### Step 3: Profile each step
- Use your profiler (cProfile, perf, DevTools) on each function in the chain
- Find the one that takes the most time
- That's where optimization effort should go

### Step 4: Check for redundant calls
```bash
# After profiling, you might find:
# upload_handler calls validate_file() twice
# (once for mimetype, once for size)

# Use graphify to verify:
graphify explain "validate_file()" graphify-out/graph.json

# If it's called 50 times from different places, consolidating
# the validation logic might speed things up
```

---

## Scenario 10: "How Do I Document This System?"

**The Problem:**
You need to write architecture documentation, but the docs are out-of-date as soon as you write them.

**The Recipe:**

### Step 1: Generate the graph
```bash
/graphify .
```

### Step 2: Use GRAPH_REPORT.md as a starting point
The auto-generated report includes:
- God nodes (key concepts)
- Communities (subsystems)
- Suggested questions (onboarding)

### Step 3: Augment with comments
In your code, add `# NOTE:` and `# WHY:` comments:

```python
# NOTE: This validator is called from 47 places
# WHY: We centralize validation to ensure consistency

def validate_input(data):
    ...
```

graphify extracts these and adds them to the graph. Next time you regenerate the graph, they're included.

### Step 4: Keep docs sync'd
Every time you make a major refactor:
```bash
/graphify .
# The GRAPH_REPORT.md updates automatically
# You update the hand-written docs to match
```

---

## Quick Reference

| Task | Command | Learn More |
|------|---------|-----------|
| **Map a codebase** | `/graphify .` | [Getting Started](GETTING_STARTED.md) |
| **Find god nodes** | `graphify explain --god-nodes` | Exercise 2 |
| **Trace a path** | `graphify path A B` | Exercise 3 |
| **Understand a node** | `graphify explain NODE` | Exercise 4 |
| **View communities** | `graphify communities` | Exercise 5 |
| **Ask a question** | `graphify query "..."` | Exercise 6 |
| **Track changes** | `graphify compare old.json new.json` | Exercise 7 |
| **Export a subsystem** | `graphify export --community N` | Exercise 8 |

---

## Tips for Success

1. **Run graphify after major changes.** Keep a historical record.
2. **Use the graph as a conversation starter.** Show it to teammates; let them ask questions.
3. **Combine with your profiler.** graphify shows you the structure; a profiler shows you the cost.
4. **Automate comparison.** Add `graphify compare` to your CI to track architecture drift.
5. **Trust the graph, verify with code.** graphify shows relationships; code shows implementation.

Happy cooking! 🍳
