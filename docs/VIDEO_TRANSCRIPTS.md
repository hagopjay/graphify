# Video Transcript Guidelines

Create educational video content about graphify and incorporate video transcripts into your knowledge graph. This guide covers both creating videos and extracting value from them.

---

## Part 1: Planning Your graphify Video

### Video Types & Lengths

| Type | Length | Goal | Audience |
|------|--------|------|----------|
| **Quick Tip** | 2-3 min | Show one specific command | Busy developers |
| **Tutorial** | 5-10 min | Walk through a scenario | New users |
| **Deep Dive** | 20-30 min | Explain concepts in detail | Learners |
| **Case Study** | 15-20 min | Real-world example | Decision makers |
| **Playlist** | 2-5 videos | Complete learning path | Structured learners |

### Planning Template

```yaml
Title: "Debugging with graphify: Trace a Bug in 5 Minutes"
Length: 5 minutes
Audience: Intermediate developers
Goal: Show how to use path queries for debugging
Prerequisites: graph.json already generated
Key Concepts:
  - Path queries
  - Reading output
  - Interpreting confidence levels
Segments:
  0:00 - Intro (15 sec)
  0:15 - Problem setup (30 sec)
  0:45 - Run query (45 sec)
  1:30 - Interpret results (90 sec)
  3:00 - Conclusion (30 sec)
```

---

## Part 2: Creating Effective Video Content

### The Formula: PECO

**P**roblem → **E**xample → **C**oncept → **O**utcome

```
Problem (20%):
  "You have a bug but don't know where it originates"

Example (40%):
  "Let's trace from upload_handler to database_error"
  (Show actual graphify output)

Concept (20%):
  "This is what the path tells us..."

Outcome (20%):
  "Now you know where to look. Here's what we'd investigate next."
```

### Script Template

```
# Graphify Tutorial: [Topic]

## INTRO (15 sec)
[Voiceover] "In this video, we'll learn how to [outcome]. 
By the end, you'll be able to [skill]."

[Visual] Animated title card with topic

## PROBLEM (30 sec)
[Voiceover] "Imagine you're working on a large codebase 
and you run into [specific problem]. 
Traditional tools like grep show you 300 matches. 
What you really need is to understand the [concept]."

[Visual] Show the problem (code snippet, error, or confusion)

## EXAMPLE (2 min)
[Voiceover] "Let's see how graphify solves this."

[Visual] Demo at normal speed once
[Visual] Slow-motion replay with callouts
[Visual] Highlight key parts of output

[Voiceover] "Notice how the output shows [specific detail]..."

## CONCEPT (1 min)
[Voiceover] "This works because graphify builds a graph 
of your codebase's relationships. Each edge is tagged 
with confidence: EXTRACTED, INFERRED, or AMBIGUOUS."

[Visual] Simple animation of concepts
[Visual] Diagram of how relationships are tagged

## OUTCOME (30 sec)
[Voiceover] "Now you can use this technique to debug 
any code path in your project. Try it on your own codebase."

[Visual] Closing slide: "Next: [Next Video Topic]"
[CTA] "Subscribe / Check out the docs / Try it now"
```

---

## Part 3: Transcript Management

### Transcript Format

Keep transcripts in this format for easy parsing:

```
[Title: Debugging with graphify]
[Duration: 5:00]
[Date: 2026-09-26]

0:00 - INTRO
Welcome to graphify debugging tutorial. Today we'll learn...

0:15 - PROBLEM SETUP
Imagine you have a 50k-line codebase and encounter a bug...

0:45 - QUERY DEMONSTRATION
Let's run graphify path "upload_handler" "database_write"

1:30 - INTERPRETING RESULTS
The output shows 4 hops. Here's what each one means...

3:00 - KEY TAKEAWAY
This path tells us exactly where the bug originates...

3:30 - NEXT STEPS
Try this on your own codebase. Next video will cover...
```

### Auto-Extract Key Moments

```bash
# Extract timestamps and sections
grep "^\[0:[0-9]" transcript.txt | sort

# Extract commands shown
grep "graphify " transcript.txt

# Extract concepts mentioned
grep "graph\|node\|edge\|community\|confidence" transcript.txt
```

### Convert Transcript to Markdown

```bash
#!/bin/bash
# transcript_to_md.sh

TRANSCRIPT=$1
OUTPUT=${2:-output.md}

{
    echo "# Video Transcript"
    echo ""
    
    while IFS= read -r line; do
        if [[ $line =~ ^\[Title ]]; then
            TITLE=$(echo "$line" | sed 's/\[Title: //;s/\]//')
            echo "## $TITLE"
        elif [[ $line =~ ^\[0:[0-9] ]]; then
            echo ""
            echo "### $(echo "$line" | sed 's/^\[//;s/\]//')"
        else
            echo "$line"
        fi
    done < "$TRANSCRIPT"
} > "$OUTPUT"

echo "Generated $OUTPUT"
```

---

## Part 4: Integrating Transcripts into Graphs

### Add Transcripts as Nodes

When you transcribe a video, add it to your graph:

```json
{
  "id": "video:debugging-with-graphify",
  "label": "Video: Debugging with graphify",
  "source_file": "transcripts/debugging-with-graphify.txt",
  "source_location": "L1",
  "type": "video",
  "duration_seconds": 300,
  "topics": ["debugging", "path-queries", "confidence"]
}
```

Link to related concepts:

```json
{
  "source": "video:debugging-with-graphify",
  "target": "graphify:path",
  "relation": "demonstrates",
  "confidence": "EXTRACTED"
}
```

### Extract Topics Automatically

```python
import re

def extract_topics_from_transcript(transcript_text):
    """Extract key concepts mentioned in video."""
    
    # Graphify-specific concepts
    concepts = {
        'god nodes': r'\bgod node',
        'confidence': r'\bconfidence|EXTRACTED|INFERRED',
        'path queries': r'\bgraphify path\b|path query|shortest path',
        'communities': r'\bcommunities|subsystem',
        'call graph': r'\bcall graph|function call',
    }
    
    found = {}
    for concept, pattern in concepts.items():
        if re.search(pattern, transcript_text, re.IGNORECASE):
            count = len(re.findall(pattern, transcript_text, re.IGNORECASE))
            found[concept] = count
    
    return sorted(found.items(), key=lambda x: x[1], reverse=True)

# Use it
with open('transcript.txt') as f:
    topics = extract_topics_from_transcript(f.read())
    
print("Key topics discussed:")
for topic, count in topics:
    print(f"  - {topic} ({count} mentions)")
```

### Create a Video Knowledge Base

```
video-knowledge-base/
├── quick-tips/
│   ├── god-nodes-explained.txt
│   ├── path-queries-101.txt
│   └── communities-visualized.txt
├── tutorials/
│   ├── complete-walkthrough.txt
│   ├── debugging-scenario.txt
│   └── architecture-review.txt
├── deep-dives/
│   ├── extraction-pipeline.txt
│   ├── confidence-explained.txt
│   └── graph-algorithms.txt
└── index.md        # Links and metadata
```

---

## Part 5: Video Series Planning

### Beginner Series (5 videos)

1. **"graphify in 60 Seconds"** (1 min)
   - What is it?
   - Why use it?
   - Quick demo

2. **"Your First Graph"** (5 min)
   - Install
   - Run command
   - Explore graph.html

3. **"Understanding Nodes and Edges"** (8 min)
   - What are they?
   - Real examples
   - Why it matters

4. **"Finding God Nodes"** (5 min)
   - What's a god node?
   - Why do they matter?
   - How to find them

5. **"Your First Query"** (5 min)
   - Explain a node
   - Ask a question
   - Trace a path

### Intermediate Series (4 videos)

1. **"Debugging a Bug Path"** (8 min)
   - Real scenario
   - Using path queries
   - Reading output

2. **"Refactoring with graphify"** (10 min)
   - Understanding coupling
   - Finding opportunities
   - Validating changes

3. **"Designing New Systems"** (8 min)
   - Using god nodes
   - Avoiding tight coupling
   - Validating architecture

4. **"Onboarding with graphify"** (7 min)
   - Show architecture
   - Explain relationships
   - Guide exploration

---

## Part 6: Transcript SEO & Discoverability

### Add Timestamps to Video Descriptions

```
My graphify Tutorial: Understanding Communities

0:00 - Introduction
0:30 - What is a community?
2:15 - How to view communities
3:45 - Community structure example
5:20 - Practical use: refactoring
6:45 - Conclusion & next steps

Full transcript: [link]
Key concepts: god nodes, communities, Leiden clustering
```

### Create a Searchable Transcript Index

```markdown
# graphify Transcript Index

## Videos Mentioning "god nodes"
- Quick Tips: God Nodes Explained (2:15)
- Tutorial: Finding Bottlenecks (4:30)
- Deep Dive: Call Graph Analysis (15:45)

## Videos Mentioning "communities"
- Beginner: Communities Visualized (5:15)
- Tutorial: Architecture Refactoring (8:20)
- Deep Dive: Leiden Clustering (22:10)

## Commands Demonstrated
- `graphify explain --god-nodes` (video 1, 2:45)
- `graphify query "..."` (video 2, 1:20)
- `graphify path A B` (video 3, 4:15)
```

---

## Part 7: Quality Checklist

### Before Publishing

- [ ] **Audio:** Clear, no background noise, consistent volume
- [ ] **Video:** 1080p minimum, 60fps preferred
- [ ] **Pace:** Not too fast, time for viewers to read output
- [ ] **Captions:** Burned-in or SRT file provided
- [ ] **Transcript:** Complete, with timestamps
- [ ] **Examples:** Runnable, tested before recording
- [ ] **Conclusion:** Clear takeaway, next steps

### Transcript Checklist

- [ ] Accurate to video (watch twice before finalizing)
- [ ] All commands transcribed exactly
- [ ] Technical terms spelled correctly
- [ ] Links in video are written in transcript
- [ ] Timestamps at segment changes
- [ ] Searchable format (plain text or markdown)
- [ ] Topics/tags listed at top

---

## Part 8: Distribution

### Where to Share

1. **YouTube**
   - Full videos with descriptions
   - Links to docs, code, transcripts
   - Playlists by skill level

2. **Your Documentation**
   - Embed videos in tutorial pages
   - Link transcripts from docs
   - Cross-reference videos in FAQ

3. **Blog/Medium**
   - Write-up of concept with embedded video
   - Pull quotes from transcript
   - Link to full docs

4. **GitHub Discussions**
   - Share relevant videos when answering questions
   - Link to transcripts for text search

5. **Conferences/Podcasts**
   - Reference your videos
   - Include transcripts in show notes

### Markdown Embed Template

```markdown
## Learning Video

<div class="video-container">
  <iframe width="560" height="315" 
    src="https://www.youtube.com/embed/..." 
    frameborder="0" allowfullscreen></iframe>
</div>

**Duration:** 5 minutes  
**Level:** Beginner  
**Key Concepts:** God nodes, analysis, bottlenecks

[📄 Full Transcript](transcripts/video-name.md)

### Summary
In this video, you'll learn about god nodes—the most-connected 
concepts in your graph that act as critical bottlenecks.
```

---

## Part 9: Transcript Metadata Format

```yaml
---
title: "Debugging a Bug Path with graphify"
video_id: "youtube:xyz123"
length: "8:35"
difficulty: "intermediate"
created: 2026-09-26
topics:
  - path queries
  - debugging
  - confidence levels
  - call graphs
key_commands:
  - graphify path "upload_handler()" "database_write()"
  - graphify explain "validate_file()"
related_docs:
  - docs/LEARNING.md (Exercise 3)
  - docs/COOKBOOK.md (Scenario 2)
---

[Full transcript below...]
```

---

## Part 10: Building Your Video Library

### Recommended Schedule

**Month 1:** Quick Tips (5 × 2 min videos)
- Focus on individual commands
- Low production burden
- High replay value

**Month 2:** Beginner Tutorials (5 × 5 min videos)
- Complete learning path
- Reference guides
- Built-in search via transcript

**Month 3:** Intermediate (4 × 8 min videos)
- Real-world scenarios
- Problem solving
- Best practices

**Ongoing:** Deep Dives & Case Studies
- Community contributions
- Advanced features
- Success stories

### Growth Milestones

| Milestone | Effort | Impact |
|-----------|--------|--------|
| **10 videos** | 1 month | Cover basics + scenarios |
| **25 videos** | 2 months | Complete learning path |
| **50 videos** | 4 months | Reference library |
| **100 videos** | 8+ months | Comprehensive resource |

---

## Examples of Good Video Concepts

✅ **"God Nodes Explained in 3 Minutes"**
- Clear problem
- One command shown
- Easy to remember

✅ **"Trace a Bug with graphify"**
- Real scenario
- Step-by-step walkthrough
- Multiple related concepts

✅ **"Why Communities Matter"**
- Concept explanation
- Real example
- Practical applications

❌ **"graphify Overview"** (too broad)
❌ **"How to Build a Graph"** (too technical)
❌ **"All graphify Features"** (too much)

---

## Tools & Resources

### Recording
- **OBS Studio** (free, cross-platform)
- **ScreenFlow** (Mac, simple)
- **Camtasia** (professional)

### Editing
- **DaVinci Resolve** (free, professional)
- **Final Cut Pro** (Mac, professional)
- **Adobe Premiere** (industry standard)

### Transcription
- **Rev** ($0.25/min, accurate)
- **Otter.ai** (AI-powered, budget)
- **OpenAI Whisper** (free, local)

### Hosting
- **YouTube** (free, discoverable)
- **Vimeo** (paid, professional)
- **Your website** (self-hosted)

---

## Summary: Video Content Strategy

1. **Start small:** 2-3 min quick tips first
2. **Build series:** Group by learning level
3. **Transcribe everything:** For SEO + accessibility
4. **Link & reference:** Connect videos to docs
5. **Gather feedback:** Ask viewers what they want next
6. **Iterate:** Improve based on comments
7. **Build library:** 50+ videos over time

---

## Next Steps

1. **Plan your first video series** using the templates
2. **Record a quick tip** (2-3 minutes)
3. **Create transcript** with timestamps
4. **Add to docs** with links
5. **Gather feedback** from community
6. **Plan series 2** based on what resonated

Happy creating! 🎬

---

**Questions about video production?** Check out:
- [Mux Video Production Guide](https://mux.com/)
- [YouTube Creator Academy](https://creatoracademy.youtube.com/)
- [Descript Video Editing](https://www.descript.com/)
