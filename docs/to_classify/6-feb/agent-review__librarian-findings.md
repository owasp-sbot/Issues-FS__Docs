# Librarian Role: Self-Review Findings

**Document:** agent-review__librarian-findings
**Date:** 2026-02-06
**Source:** AI agent review (Librarian subagent performing meta self-review, 35 tool calls, ~15 min)
**Status:** Raw findings for human review

---

## The Meta Observation

The Librarian role has the **richest conceptual documentation** in the entire ecosystem (a 428-line architecture document grounded in library science) but the **emptiest repository** (just a LICENSE file). The very problem the Librarian exists to solve -- scattered, unconnected, unclassified knowledge -- is exactly what is happening to the Librarian's own documentation.

---

## Current State: What Exists

### In the Librarian Repo (`roles/Issues-FS__Dev__Role__Librarian/`)

| File | Description |
|------|-------------|
| `LICENSE` | MIT License (the only file) |

That's it. No ROLE.md, no CLAUDE.md, no system prompt, no Python package, no CI, no runbooks.

### Librarian Documentation Scattered Elsewhere

| File | Location | Description |
|------|----------|-------------|
| `v0_4_0__issues-fs__librarian-role.md` | `Issues-FS__Docs/docs/to_classify/6-feb/` | The main architecture document. 428 lines. Grounded in library science principles. Defines 4 core workflows, graph-native operations, and ecosystem health metrics. |
| `v0_4_0__issues-fs__librarian-role-side-capture.md` | `Issues-FS__Docs/docs/to_classify/6-feb/` | Side-capture with 4 un-triaged ideas from a conversation. Needs processing into decisions/tasks. |
| `v0.1.0__issues-fs__role-based-agent-coordination.md` | `Issues-FS__Docs/docs/to_classify/` | The overarching role coordination architecture. Defines the Librarian as one of 6 roles with responsibilities, handoff protocols, and ROLE.md template. |
| `v0_4_0__issues-fs__thinking-in-graphs.md` | `Issues-FS__Docs/docs/to_classify/` | Foundational philosophy that the Librarian role is built on. References the Librarian throughout. |
| `v0_4_0__issues-fs__lexicon-architecture-v2.md` | `Issues-FS__Docs/docs/to_classify/` | Lexicon design that the Librarian is responsible for maintaining. |

### Comparison: DevOps Role (the standard to match)

The DevOps role repo has **16 files** including:
- `pyproject.toml` (package definition)
- `README.md`
- `docs/` with runbooks for new repo setup and minimum repo files
- `issues_fs_dev_role_devops/` Python package structure
- `.github/workflows/` CI configuration
- Proper Git setup with branches

---

## The Librarian Role: What It Should Be

Based on the architecture documents, the Librarian is described as the **"most graph-native role"** -- its primary output is *edges* (connectivity), not documents.

### Core Identity (from library science)

| Library Science Concept | Issues-FS Equivalent |
|------------------------|---------------------|
| **Cataloguing** | Ensuring nodes have enough edges to be discoverable |
| **Classification** | Connecting nodes to Lexicon anchors |
| **Authority control** | Linking variant names for the same concept across scopes |
| **Weeding** | Identifying stale, redundant, or superseded nodes |
| **Finding aids** | Curated subgraphs for navigating complex areas |
| **Reference services** | Answering "do we have information about X?" by graph traversal |

### Four Core Workflows

1. **Draft Processing Pipeline** -- Receive raw content from any role, extract structured knowledge, integrate into the graph
2. **Ecosystem Health Scan** -- Systematic review of connectivity, currency, consistency across all repos
3. **Lexicon Maintenance** -- Curate anchor nodes, detect drift, propose new anchors
4. **Knowledge Request Fulfillment** -- Respond to `Knowledge_Request` issues from other roles

### What the Librarian Creates (Issue Types)

- `Knowledge_Entry` -- Structured knowledge integrated into the graph
- `Glossary_Update` -- Changes to shared terminology
- `Index_Update` -- Changes to finding aids
- `Architecture_Doc_Update` -- Updates to architecture documentation
- `Ecosystem_Health_Report` -- Results of health scans

### What the Librarian Consumes

- `Knowledge_Request` (from any role)
- `Decision` / `ADR` (from Architect)
- `Handoff` (from any role, with documentation deliverables)

---

## Document Quality Assessment

### Strengths
- The main architecture document is **excellent** -- deeply thought through, grounded in library science, rich with examples
- Clear articulation of the Librarian as a graph-native role (not just a doc writer)
- Well-defined workflows with concrete inputs and outputs
- Good integration with the broader role coordination architecture

### Issues
- **Scattered across repos** -- The Librarian's own documentation doesn't live in the Librarian's repo
- **No executable artifacts** -- No ROLE.md, no system prompt, no Python package, no CI
- **Side-capture is un-triaged** -- 4 ideas sitting in a raw capture file, not processed into decisions
- **No CLAUDE.md** -- An AI agent stepping into the Librarian role has no briefing file
- **Gap between vision and reality** -- The richest conceptual role has the emptiest implementation

### Overlaps and Redundancies
- The role coordination doc and the librarian-specific doc both describe the Librarian's responsibilities, but at different levels of detail
- The "Thinking in Graphs" doc and the Librarian doc both explain anchor nodes and graph traversal, with the Librarian doc being more role-specific
- Some content in the side-capture overlaps with ideas already in the main doc

---

## Side-Capture: Un-triaged Ideas

The side-capture file (`v0_4_0__issues-fs__librarian-role-side-capture.md`) contains 4 raw ideas:

1. **An idea about document versioning** -- How the Librarian should handle document version tracking
2. **An idea about cross-repo knowledge graphs** -- Connecting knowledge across submodules
3. **An idea about automated health checks** -- Scripted connectivity analysis
4. **An idea about Librarian-as-first-user** -- Bootstrapping the Librarian role by having it catalog itself

These need to be triaged: accepted as tasks, captured as decisions, or rejected.
