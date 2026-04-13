# Relational Identity: How Zora Should Store Understanding

## The Problem

Current information retrieval treats identity as a collection of independent facts to be searched. Vector search finds similar things. BM25 finds matching keywords. Knowledge graphs store explicit triples. All of these miss what actually constitutes understanding of a person: the **relationships between** observations, not the observations themselves.

## Three Influences

### Karpathy's LLM Knowledge Bases

Andrej Karpathy described a pattern emerging in practice:

> Raw data from sources is collected, then compiled by an LLM into a markdown wiki, then operated on by various CLIs by the LLM to do Q&A and to incrementally enhance the wiki. You rarely ever write or edit the wiki manually — it's the domain of the LLM. I think there is room here for an incredible new product instead of a hacky collection of scripts.

Key properties:
- The LLM is both writer and reader of the knowledge base
- The human rarely touches the wiki directly
- Queries get filed back, so the wiki improves through use
- An index maintained by the LLM is sufficient for retrieval at moderate scale

This describes the **mechanism** — LLM as compiler/maintainer of structured markdown. But Karpathy is talking about knowledge (research, facts, datasets). Zora applies this to something harder: behavioral identity.

### Calvino's Zora (*Invisible Cities*)

> *Zora has the quality of remaining in your memory point by point... Zora's secret lies in the way your gaze runs over patterns following one another as in a musical score where not a note can be altered or displaced... This city which cannot be expunged from the mind is like an armature, a honeycomb in whose cells each of us can place the things he wants to remember... Between each idea and each point of the itinerary an affinity or a contrast can be established, serving as an immediate aid to memory.*

> *But in vain I set out to visit the city: forced to remain motionless and always the same, in order to be more easily remembered, Zora has languished, disintegrated, disappeared.*

Three insights from Calvino:

1. **Memory through structure, not content.** Zora isn't memorable because anything in it is beautiful. It's memorable because of how things relate — patterns following one another like a musical score. You don't understand a person by their facts. You understand them by their patterns: how they react, what contradicts what, the rhythm of how they think.

2. **Affinities AND contrasts.** Between any two points, "an affinity or a contrast can be established." Contradictions are the most revealing relationships. "Meticulous about APIs but impatient with boilerplate" — that contrast is more revealing than either fact alone.

3. **The warning.** A static structure dies. Calvino's Zora "languished, disintegrated, disappeared" because it couldn't change. The persona engine must grow — signals accumulate, reflections synthesize new patterns, dispositions evolve — or it becomes irrelevant.

Calvino provides the **architecture** (identity as a walkable lattice of patterns and contradictions) and the **constraint** (the lattice must evolve or die).

### The Existing Prototype

The `~/.claude/memory/` repo is already Karpathy's pattern running in production:
- Raw signals come from conversations
- The LLM compiles them into structured markdown (`user/`, `feedback/`, `projects/`, `reference/`)
- `MEMORY.md` is the LLM-maintained index
- The human rarely writes directly
- It gets queried at session start for context
- It accumulates over time

It works at small scale because the LLM can read the index and pull what's relevant. But it's held together with CLAUDE.md instructions and flat file reads — no real search, no relationship tracking, no reflection cycle. The memory repo is the prototype; Zora is the product.

## The Insight: LLM-Native Information Retrieval

Traditional IR approaches all fail to capture what a psychiatrist does:

| Approach | What it captures | What it misses |
|----------|-----------------|----------------|
| **Vector search** | Semantic similarity — finds things that are "near" each other | Contradiction. Two opposing behaviors can be the most important connection, but they're far apart in embedding space |
| **BM25 / keyword** | Lexical matches | Any relationship not expressed through shared vocabulary |
| **Knowledge graphs** | Explicit triples (`Jason -> prefers -> Rust`) | Nuance, tension, interpretation. A triple is a skeleton of understanding |
| **RAG** | Reconstructs connections at query time | Connections are ephemeral — re-derived every session, never accumulating |

A psychiatrist doesn't store "patient is meticulous" and "patient is impatient" as two separate nodes with a "contradicts" edge. They write: "meticulous about API boundaries but impatient with boilerplate — the precision is about craft, the impatience is about what they consider beneath the craft. The threshold between the two reveals what they respect."

The relationship is **narrative**. It has interpretation. And it's **durable** — written once, referenced many times.

### Relationships as First-Class Memories

The key idea: **relationships should be first-class memories, not derived at query time.**

The reflection cycle doesn't just consolidate signals into dispositions — it synthesizes connections between dispositions. "This person's spatial thinking (cognitive) explains why they reject prose explanations (correction) and reach for diagrams (preference) — these three signals are one pattern." That connection, once synthesized, gets stored as its own artifact. Not a graph edge. A narrative.

Then retrieval isn't "find the 6 most similar chunks." It's **walk the structure** — follow connections the LLM already built, the way you walk Calvino's city. The LLM wrote the city; now it walks through it.

The `axes` field in the disposition spec is a primitive version of this — `[meticulous, impatient]` encodes a relationship as structure. But the full version would have the reflection process producing durable, narrative connections between any aspects of identity.

## Open Questions

- What's the right representation for relationship memories? Prose with structured frontmatter? A new memory type alongside identity/disposition/context/signal?
- How does retrieval "walk the structure" in practice? Does the LLM follow explicit backlinks, or does the narrative style create implicit paths that embedding search can follow?
- How do relationship memories evolve? When new signals challenge an existing connection, does the reflection cycle update the relationship or create a competing one?
- At what scale does the LLM-maintained index break down and require actual graph traversal?

## Synthesis

Zora sits at the intersection of three ideas:

- **Karpathy's mechanism** — LLM as compiler/maintainer of structured markdown, improving through use
- **Calvino's architecture** — identity as a walkable lattice of patterns and contradictions, not a bag of facts
- **Calvino's constraint** — the lattice must evolve or die

The product opportunity isn't better RAG for personal data. It's a new kind of information retrieval where the LLM builds durable, narrative relationships between observations, and then traverses those relationships to reconstruct understanding. The relationships are the product. Everything else is plumbing.
