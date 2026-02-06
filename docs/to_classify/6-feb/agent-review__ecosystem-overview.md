# Issues-FS Ecosystem Overview

**Document:** agent-review__ecosystem-overview
**Date:** 2026-02-06
**Source:** AI agent review (Project Reviewer subagent, 47 tool calls, ~16 min exploration)
**Status:** Raw findings for human review

---

## The Big Picture

The Issues-FS ecosystem lives within a larger **MGraph-AI** project (~27 repositories) authored by Dinis Cruz (OWASP SBOT), focused on graph-based data processing with specializations for HTML, text, LLMs, AWS, and more.

The repo hierarchy:

```
mgraph-ai/                                                  (ecosystem root, ~27 repos)
  MGraph-AI__UI__Html-Transformation-Workbench/              (parent repo)
    modules/
      Issues-FS__Dev/                                        (development hub)
        modules/                                             (6 Issues-FS submodules)
        roles/                                               (2 agent role submodules)
```

---

## What Is Issues-FS?

**Issues-FS is a file-system-based, graph-native issue tracking system designed to live inside Git repositories.**

Issues are stored as JSON files in a `.issues/` directory structure, committed and versioned alongside code. No external services, databases, or API tokens required.

### Core Design Principles

1. **Git-native:** Issues branch/merge with code, appear in `git diff`
2. **Graph data model:** Everything is a node. Relationships (blocks, depends-on, assigned-to) are first-class bidirectional links
3. **Pluggable storage:** Memory-FS abstraction with backends for memory (tests), local disk (Git), SQLite, ZIP, S3/GCS
4. **Type-safe:** All Python code uses osbot-utils `Type_Safe` with `Safe_*` primitives for runtime validation
5. **AI-agent first:** Designed for AI agents to create, manage, and query issues without external auth
6. **Hierarchical:** Issues nest inside other issues via `issues/` subdirectories (fractal structure)

### Storage Layout

```
.issues/
  config/
    node-types.json         # Type definitions (bug, task, feature, person, project, etc.)
    link-types.json         # Relationship definitions (blocks, depends-on, etc.)
  data/
    {type}/
      {Label}/
        issue.json          # Node data (node_id, type, index, label, title, status, tags, links, properties)
        issues/             # Child issues (fractal nesting)
  indexes/
    issues.mgraph.json      # MGraph-DB cache for graph queries
```

---

## The Graph-First Philosophy

Documented extensively in "Thinking in Graphs: Meaning Through Connectivity" (Feb 5, 2026). Ten core principles:

1. **Everything is a node.** Nodes carry local properties but have no obligation to declare what they are.
2. **Meaning comes from edges.** What a node "is" emerges from graph relationships traceable from it.
3. **Confidence is proportional to connectivity.** More edges to well-defined reference points = higher confidence.
4. **The system is fractal.** Every scope can define its own nodes, edges, types, vocabulary.
5. **Anchor nodes enable interoperability without enforcing conformity.**
6. **Compatibility is computed, not declared.** Subgraph overlap determines compatibility.
7. **Honest uncertainty is the default.** Report what the graph supports, never assume.
8. **Enrichment, not enforcement.** Low confidence is remedied by adding edges, not validation rules.
9. **Cross-graph edges are first-class.** The most powerful connections span graphs.
10. **No node is aware of how it's used.** Meaning is extracted from surrounding structure.

This is explicitly distinguished from schema-first thinking where types are declared and validated. Here types are *discovered* through graph traversal.

---

## The Lexicon (Planned)

`Issues-FS__Lexicon` is designed as the root graph of the ecosystem, providing:

- **Anchor nodes** for shared concepts (Task, Bug, Decision, Handoff, Review_Request, etc.)
- **Reference patterns** (Workflow, State Machine, Review Process, Fractal Scope)
- **Analysis tools** (connectivity, compatibility, coverage, gaps, conflicts)
- **Bootstrap definitions** (opinionated but overridable defaults)

Together with `osbot-utils`, the Lexicon forms one of two universal dependencies. The Lexicon repo does not yet exist but has extensive architectural documentation (v2.0).

---

## Role-Based Agent Coordination

Six specialized AI agent roles, each encoded as a Role Repo:

| Role | Responsibility | Status |
|------|---------------|--------|
| **Conductor** | Orchestration, priorities, blockers | Designed, not built |
| **Architect** | Technical decisions, API design, ADRs | Designed, not built |
| **Dev** | Implementation, bug fixes, unit tests | Designed, not built |
| **QA** | Test strategy, quality gates | Designed, not built |
| **DevOps** | CI/CD, deployment, releases | **Implemented** (16 files) |
| **Librarian** | Documentation curation, knowledge coherence | **Repo exists, empty** |

Roles coordinate via **typed issues as a state machine**: Decision, Handoff, Review_Request, Approval, Blocker, Task, Defect, Release, Knowledge_Request, ADR.

---

## Key Technologies

| Technology | Purpose |
|-----------|---------|
| **Python 3.12+** | All repos |
| **Poetry** | Build system |
| **osbot-utils / Type_Safe** | Runtime type validation, `Safe_*` primitives |
| **Memory-FS** | Pluggable storage abstraction |
| **MGraph-DB** | Graph database for traversal and visualization |
| **FastAPI** | REST API service layer |
| **osbot-fast-api / osbot-fast-api-serverless** | FastAPI helpers and Lambda deployment |

---

## Dogfooding

Issues-FS manages itself using Issues-FS. Both the HTML Transformation Workbench and the Service UI have active `.issues/` directories tracking their own bugs, features, tasks, projects, and releases.
