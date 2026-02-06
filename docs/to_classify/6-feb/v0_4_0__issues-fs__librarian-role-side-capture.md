# Side-Capture: Ideas from Librarian Role Voice Memo

**Source:** Voice memo transcript, 2026-02-06  
**Status:** Raw — needs triage and routing  
**Related to:** issues-fs__librarian-role v1.0  

---

## Idea 1: Document Abstraction as Layered Semantic Graphs

### The Problem
Providing full documents (2000+ lines) to LLMs is expensive, overwhelms context windows, and causes attention degradation. Current workarounds (truncation, summarisation) lose structure and provenance.

### The Proposal
Create multi-layered graph abstractions of documents:

```
Layer 1: Top-level abstract
    ├── Key takeaways (3-5 nodes)
    ├── Core objectives (nodes)
    └── Links to Layer 2 sections

Layer 2: Section-level abstractions (7-8 nodes)
    ├── Section summary (node)
    ├── Key concepts (nodes)
    └── Links to Layer 3 sub-sections

Layer 3: Sub-section detail (3-4 nodes per section)
    ├── Detailed points (nodes)
    └── Links to actual document sections (anchored to line ranges or headings)
```

Each layer is a semantic graph — nodes and edges with provenance back to the source document. An LLM receives Layer 1, and if it determines it needs more detail, it can request Layer 2 for a specific section, and so on.

### Why This Is Different from Vector Databases
Vector DBs chunk documents and retrieve by similarity. This approach preserves structure, hierarchy, and provenance. Every abstracted node links back to its source. The LLM can trace *why* a claim exists, not just that it's semantically similar to the query.

### Relationship to Librarian Role
The Librarian would maintain these abstractions and trigger their creation, but the abstraction-as-graph pattern is a broader architectural concept that affects how all roles interact with documentation. This likely belongs in a separate architecture document or as an extension to the thinking-in-graphs foundation.

### Suggested routing
- **Decision issue** for Architect: "Should document abstractions be a first-class graph pattern?"
- **Extension to thinking-in-graphs**: If accepted, add a section on layered abstractions
- **Librarian workflow**: Add "Create/maintain document abstractions" as a workflow

---

## Idea 2: Context Management for LLM Interactions

### The Pattern
Instead of providing full documents as context to LLMs, provide:
1. A compressed graph abstraction (Layer 1 from Idea 1)
2. Metadata about what deeper information is available
3. An explicit instruction: "If you need more data, request [specific document/section]"

This creates a pull-based context model rather than a push-based one. The LLM applies its own judgment about what additional context it needs.

### Implications
- Every significant document needs a "context card" — a minimal graph representation suitable for LLM context windows
- The Librarian could maintain these context cards as part of the accession workflow
- This may influence how role prompts are structured: instead of embedding all reference docs, embed context cards with pointers

### Suggested routing
- Potentially part of a broader "LLM interaction patterns" document
- Affects role repo prompt design (how system_prompt.md references external knowledge)
- Could be a Decision issue: "How should agent contexts reference ecosystem documentation?"

---

## Idea 3: Storing Document Relationships in MGraph-DB

### The Observation
Documents like the Librarian role brief generate multiple artifact types: philosophical principles, operational guidelines, tasks, sub-projects. A single markdown file can't capture the full graph of relationships between these elements.

### The Proposal
For documents that are rich in internal structure and cross-references, consider storing the relationship graph in:
- An MGraph-DB file (`.mgraph`)
- A series of structured files that can be assembled on demand
- A zip file containing both the document and its relationship graph

### Suggested routing
- **Decision issue** for Architect: "Should documents have companion graph files?"
- Relates to the Lexicon's graph storage (MGraph-native representations)
- May affect the Issues-FS file system structure (`.issues-fs/` directory contents)

---

## Idea 4: Every Published Artifact Gets Its Own Ontology

### The Observation
From the transcript: "every document, everything that we publish, starts to have its own ontology and taxonomy — that's how we connect things."

### The Implication
This extends the fractal principle from scopes to individual documents. Each document is itself a scope that can define local concepts, and those concepts can link to Lexicon anchors. The document's "ontology" is just its local subgraph.

### Suggested routing
- This may already be implicit in the thinking-in-graphs document (every scope defines its own vocabulary)
- Worth making explicit: a document IS a fractal scope
- Could be added to the Lexicon architecture as a note on granularity

---

*Side-capture from Librarian role voice memo*  
*Date: 2026-02-06*
