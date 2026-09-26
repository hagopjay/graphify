# graphify Documentation Index

Welcome to the graphify learning hub! Whether you're brand new or looking to customize the system, find what you need below.

---

## 🚀 Quick Navigation

### I'm New. Where Do I Start?

1. **[Getting Started (10 min read)](GETTING_STARTED.md)** ← Start here!
   - Install graphify
   - Run it on your first project
   - Open the interactive graph
   - Ask your first questions

2. **[Concepts & Mental Models (15 min read)](CONCEPTS.md)**
   - Understand what a knowledge graph is
   - Learn the three layers (code, relations, meaning)
   - See how the pipeline works
   - Grasp confidence levels and edge types

3. **[Interactive Learning (hands-on)](LEARNING.md)**
   - 10 exercises with real commands
   - Watch actual output
   - Discover patterns you'll use daily
   - Learn by doing

### I Have a Specific Task

→ **[Cookbook: Real-World Scenarios](COOKBOOK.md)**

Pick your scenario:
- [Scenario 1: "I need to add a feature. Where do I start?"](COOKBOOK.md#scenario-1-i-need-to-add-a-feature-where-do-i-start)
- [Scenario 2: "There's a bug. Trace it."](COOKBOOK.md#scenario-2-theres-a-bug-trace-it)
- [Scenario 3: "This code is a mess. How do I refactor it?"](COOKBOOK.md#scenario-3-this-code-is-a-mess-how-do-i-refactor-it)
- [Scenario 4: "I'm adding a new subsystem. How do I design it?"](COOKBOOK.md#scenario-4-im-adding-a-new-subsystem-how-do-i-design-it)
- [Scenario 5: "My tests are breaking. What changed?"](COOKBOOK.md#scenario-5-my-tests-are-breaking-what-changed)
- [Scenario 6: "This library should be independent. Is it?"](COOKBOOK.md#scenario-6-this-library-should-be-independent-is-it)
- [Scenario 7: "How similar are these two functions?"](COOKBOOK.md#scenario-7-how-similar-are-these-two-functions)
- [Scenario 8: "How do I onboard someone to this codebase?"](COOKBOOK.md#scenario-8-how-do-i-onboard-someone-to-this-codebase)
- [Scenario 9: "I'm optimizing for performance. Where's the bottleneck?"](COOKBOOK.md#scenario-9-im-optimizing-for-performance-wheres-the-bottleneck)
- [Scenario 10: "How do I document this system?"](COOKBOOK.md#scenario-10-how-do-i-document-this-system)

### I Want Visual Explanations

→ **[Architecture Visual Guide](ARCHITECTURE_VISUAL.md)**

Includes diagrams of:
- The three layers
- The 7-step pipeline
- Node and edge types
- Communities and god nodes
- Query execution
- Performance characteristics

### Something's Not Working

→ **[FAQ & Troubleshooting](FAQ.md)**

Find quick answers to:
- Installation issues
- Performance problems
- Extraction bugs
- Query tips
- Common errors

### I Want to Tinker & Customize

→ **[Developer Guide](DEVELOPER.md)**

Learn how to:
- Add a new language (detailed example: Rust)
- Customize extraction rules
- Build tools on top
- Optimize performance
- Run tests

---

## 📚 Learning Paths

### Path 1: New User (30 minutes)
```
Getting Started (10 min)
  ↓
Concepts (15 min)
  ↓
Open graph.html and play (5 min)
```

### Path 2: Daily User (1-2 hours)
```
Getting Started (10 min)
  ↓
Concepts (15 min)
  ↓
Interactive Learning exercises (45 min)
  ↓
Try Cookbook scenarios (30 min)
```

### Path 3: Deep Dive (3-4 hours)
```
All of Path 2
  ↓
Architecture Visual Guide (30 min)
  ↓
ARCHITECTURE.md (technical) (30 min)
  ↓
Read the source code (you'll understand it now!) (30 min)
```

### Path 4: Power User / Contributor (5+ hours)
```
All of Path 3
  ↓
Developer Guide (1-2 hours)
  ↓
Add a language or feature (2+ hours)
  ↓
Open a PR! (contrib)
```

---

## 🎯 By Task Type

### Understanding a Codebase
1. [Getting Started](GETTING_STARTED.md) — Map it
2. [ARCHITECTURE_VISUAL.md](ARCHITECTURE_VISUAL.md) — See the shape
3. [COOKBOOK.md](COOKBOOK.md) — Use it to answer questions

### Refactoring Code
1. [CONCEPTS.md](CONCEPTS.md) — Understand coupling
2. [COOKBOOK.md](COOKBOOK.md#scenario-3-this-code-is-a-mess-how-do-i-refactor-it) — Plan the refactor
3. [LEARNING.md](LEARNING.md#exercise-7-compare-graphs-over-time) — Validate improvement

### Adding Features
1. [COOKBOOK.md](COOKBOOK.md#scenario-1-i-need-to-add-a-feature-where-do-i-start) — Find where
2. [LEARNING.md](LEARNING.md#exercise-3-trace-a-path) — Understand impacts
3. [COOKBOOK.md](COOKBOOK.md#scenario-4-im-adding-a-new-subsystem-how-do-i-design-it) — Avoid coupling

### Debugging Bugs
1. [COOKBOOK.md](COOKBOOK.md#scenario-2-theres-a-bug-trace-it) — Trace the path
2. [LEARNING.md](LEARNING.md#exercise-3-trace-a-path) — Use path queries
3. [FAQ.md](FAQ.md#q-query-returns-0-results) — If stuck

### Optimizing Performance
1. [COOKBOOK.md](COOKBOOK.md#scenario-9-im-optimizing-for-performance-wheres-the-bottleneck) — Find bottlenecks
2. [LEARNING.md](LEARNING.md#exercise-2-find-god-nodes) — Identify hot spots
3. [ARCHITECTURE_VISUAL.md](ARCHITECTURE_VISUAL.md#10-memory--performance) — Understand limits

### Onboarding New People
1. [GETTING_STARTED.md](GETTING_STARTED.md) — Give them this
2. [COOKBOOK.md](COOKBOOK.md#scenario-8-how-do-i-onboard-someone-to-this-codebase) — Guide them through
3. [LEARNING.md](LEARNING.md) — Let them do exercises

### Documenting Architecture
1. [COOKBOOK.md](COOKBOOK.md#scenario-10-how-do-i-document-this-system) — Auto-generate docs
2. [ARCHITECTURE_VISUAL.md](ARCHITECTURE_VISUAL.md) — Add visual context
3. [CONCEPTS.md](CONCEPTS.md) — Explain the mental models

### Customizing graphify
1. [DEVELOPER.md](DEVELOPER.md) — Main guide
2. [ARCHITECTURE.md](../ARCHITECTURE.md) — Technical details
3. [Source code](../graphify/) — Read the implementation

---

## 🔍 By Question Type

### "How do I...?" (GETTING_STARTED.md)
- Install graphify
- Run it on my project
- Open the interactive graph
- Ask questions

### "What is...?" (CONCEPTS.md)
- A knowledge graph
- A node / edge
- A community
- EXTRACTED / INFERRED / AMBIGUOUS confidence
- A god node

### "Show me how to..." (LEARNING.md)
- Understand a graph
- Find god nodes
- Trace a path
- Explain a node
- Find communities
- Query a question
- Compare graphs
- Dive into a subsystem

### "I need to..." (COOKBOOK.md)
- Add a feature
- Debug a bug
- Refactor code
- Design a subsystem
- Fix broken tests
- Extract a library
- Find similar functions
- Onboard someone
- Optimize performance
- Document the system

### "How do I fix...?" (FAQ.md)
- Installation errors
- Performance issues
- Extraction problems
- Query problems
- Semantic extraction issues
- Community detection issues

### "How do I customize...?" (DEVELOPER.md)
- Add a new language
- Tweak extraction rules
- Build tools on top
- Integrate into workflows
- Run tests
- Optimize performance

---

## 📖 Document Summaries

| Document | Length | Purpose | Best For |
|----------|--------|---------|----------|
| **[Getting Started](GETTING_STARTED.md)** | 10 min | Install, run, explore | First-time users |
| **[Concepts](CONCEPTS.md)** | 20 min | Understand how it works | Building mental models |
| **[Interactive Learning](LEARNING.md)** | 1 hour | Hands-on practice | Learning by doing |
| **[Cookbook](COOKBOOK.md)** | 2 hours | Real-world scenarios | Solving actual problems |
| **[Architecture Visual](ARCHITECTURE_VISUAL.md)** | 20 min | Diagrams & explanations | Visual learners |
| **[Developer Guide](DEVELOPER.md)** | 1+ hour | Customize & extend | Tinkerers & contributors |
| **[FAQ](FAQ.md)** | 30 min | Troubleshooting | Solving specific issues |

---

## 🌟 Learning Approaches

### Visual Learner?
→ Start with [Architecture Visual Guide](ARCHITECTURE_VISUAL.md), then [Getting Started](GETTING_STARTED.md)

### Hands-On Learner?
→ Start with [Getting Started](GETTING_STARTED.md), jump to [Interactive Learning](LEARNING.md)

### Read-First Learner?
→ Start with [Concepts](CONCEPTS.md), then [Getting Started](GETTING_STARTED.md)

### Problem-Solver?
→ Go straight to [Cookbook](COOKBOOK.md), use [FAQ](FAQ.md) as reference

### Deep-Diver?
→ Read in order: Getting Started → Concepts → Architecture Visual → ARCHITECTURE.md → Source code

---

## 🛠️ Quick Commands Reference

```bash
# Install
uv tool install graphifyy
graphify install

# Run
/graphify .          (in AI assistant)
graphify .           (CLI)

# Query
graphify stats graphify-out/graph.json
graphify explain --god-nodes graphify-out/graph.json
graphify explain "MyFunction" graphify-out/graph.json
graphify path "A" "B" graphify-out/graph.json
graphify query "How does X work?" graphify-out/graph.json
graphify communities graphify-out/graph.json

# Compare
graphify compare old.json new.json

# Export
graphify export --community 2 graphify-out/graph.json community-2.json

# Benchmark
graphify benchmark graphify-out/graph.json
```

See [LEARNING.md](LEARNING.md) for detailed explanations of each command.

---

## 📞 Getting Help

1. **First:** Check [FAQ.md](FAQ.md)
2. **Then:** Search [GitHub Issues](https://github.com/Graphify-Labs/graphify/issues)
3. **Finally:** Ask on [Discord](https://discord.gg/598Ad9zQZ)

If you find a bug:
- Reproduce it with `--verbose`
- Include your OS and Python version
- Share minimal code that triggers it

If you have a feature request:
- Check [DEVELOPER.md](DEVELOPER.md) (you might implement it yourself!)
- Describe the use case clearly
- Suggest implementation approach if you have one

---

## 🤝 Contributing

Want to improve graphify?

1. **Read [DEVELOPER.md](DEVELOPER.md)** to understand the codebase
2. **Check [CONTRIBUTING.md](../CONTRIBUTING.md)** for guidelines
3. **Pick an issue** or propose a feature
4. **Open a PR** with tests and documentation

Areas we'd love help with:
- New language extractors (see [DEVELOPER.md](DEVELOPER.md#part-2-adding-a-new-language))
- Performance optimizations
- Better semantic extraction
- Improved visualizations
- Documentation improvements
- Language translations

---

## 📜 License & Attribution

graphify is open source (Apache 2.0). See [LICENSE](../LICENSE) for details.

These docs are MIT licensed. Feel free to remix and redistribute!

---

## 🎓 Learning Philosophy

This documentation is built on the principle that **you learn best by doing**.

- **Getting Started** gets you to "wow" in 10 minutes
- **Concepts** builds mental models so you understand *why*
- **Interactive Learning** gives you exercises to reinforce understanding
- **Cookbook** shows real scenarios you'll encounter
- **Architecture Visual** explains the "how" with pictures
- **Developer Guide** lets you tinker with the system itself
- **FAQ** solves problems when you get stuck

Don't read it all cover-to-cover. Pick what you need now, reference what you need later.

---

## 🚀 Next Steps

1. **Never used graphify?**
   → [Getting Started](GETTING_STARTED.md) (10 minutes)

2. **Installed and ran it?**
   → [Concepts](CONCEPTS.md) (15 minutes)

3. **Want to use it daily?**
   → [Interactive Learning](LEARNING.md) (1 hour)

4. **Have a specific problem?**
   → [Cookbook](COOKBOOK.md) (find your scenario)

5. **Want to customize it?**
   → [Developer Guide](DEVELOPER.md) (1+ hours)

---

**Ready to explore?** Open `graphify-out/graph.html` in your browser and start clicking! 🎨

Questions? [Check the FAQ](FAQ.md) or [ask on Discord](https://discord.gg/598Ad9zQZ).

Happy mapping! 🗺️
