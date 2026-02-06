# Next Steps and Recommendations

**Document:** agent-review__next-steps
**Date:** 2026-02-06
**Source:** AI agent review (synthesis of Project Reviewer and Librarian subagent findings)
**Status:** Recommendations for human review

---

## Priority 1: Bootstrap the Librarian Role Repo

The Librarian repo is the most urgent gap. It has the richest design documentation but the emptiest implementation. Recommended bootstrapping sequence (mirroring what DevOps already has):

### Step 1: Minimum Viable Structure
```
Issues-FS__Dev__Role__Librarian/
  LICENSE                          (exists)
  README.md                        (create)
  pyproject.toml                   (create, mirror DevOps pattern)
  CLAUDE.md                        (create -- critical for AI agent briefing)
  ROLE.md                          (create from template in role coordination doc)
  issues_fs_dev_role_librarian/
    __init__.py
  .github/
    workflows/
      ci.yml
```

### Step 2: Populate Core Documents
- Extract ROLE.md content from the role coordination architecture doc
- Write CLAUDE.md as an AI-agent-optimized briefing (what the Librarian does, how to behave, key references)
- Write system_prompt.md for when the Librarian is instantiated as an agent

### Step 3: Move Architecture Doc
- Copy (or symlink) the main Librarian architecture doc into the Librarian repo's own `docs/`
- This is the Librarian's primary reference document and should live with the role

### Step 4: Triage the Side-Capture
- Process the 4 un-triaged ideas into proper Issues-FS issues (Decision, Task, or reject)
- This is itself a Librarian workflow ("Draft Processing Pipeline")

### Step 5: Create Initial Runbooks
Extract the 4 core workflows from the architecture doc into standalone runbook files:
- `runbook__draft-processing-pipeline.md`
- `runbook__ecosystem-health-scan.md`
- `runbook__lexicon-maintenance.md`
- `runbook__knowledge-request-fulfillment.md`

### Step 6: First Ecosystem Health Scan
Have the Librarian (agent or human) perform its first ecosystem health scan -- this produces the initial baseline for connectivity, currency, and consistency metrics.

---

## Priority 2: Create Core Briefing Documents

The agent review surfaced that there is no single "start here" document for new contributors or AI subagents. Recommended:

### For AI Subagents (CLAUDE.md files)
- **Issues-FS__Dev/CLAUDE.md** -- Already exists but may need updating with ecosystem context
- **Each submodule** should have its own CLAUDE.md (some do, some don't -- needs audit)
- **Each role repo** needs a CLAUDE.md tailored to that role's perspective

### For Human Contributors
- A "Start Here" doc that links to: ecosystem overview, component inventory, architecture overview, getting started guide
- Currently the closest thing is the executive briefing at `llm-briefs/v0.2.14__briefing__graph-based-issue-tracking.md`

---

## Priority 3: Create Remaining Role Repos

Four roles are fully specified in documentation but have no repos:

| Role | Priority | Reasoning |
|------|----------|-----------|
| **Conductor** | High | Orchestration role -- needed to coordinate the other roles |
| **Dev** | Medium | The most active role in practice, but currently implicit |
| **QA** | Medium | Quality gates become important as the system matures |
| **Architect** | Lower | Architecture decisions are being made, but the role is effectively performed by the project owner |

Each should be bootstrapped following the DevOps pattern (pyproject.toml, README, ROLE.md, CLAUDE.md, CI).

---

## Priority 4: Build the Lexicon

The `Issues-FS__Lexicon` has extensive v2.0 architecture documentation and a 5-phase migration plan. The architecture docs suggest starting with Phase 1:

- **Phase 1:** Basic data files (issue-types.json, link-types.json) migrated from the core library into a standalone package
- **Phase 2:** Add anchor node definitions as Python classes
- **Phase 3:** Add reference patterns and analysis tools
- **Phase 4:** MGraph-native graph representations
- **Phase 5:** Live graph traversal and UI visualization

Phase 1 is relatively straightforward and would immediately deliver the "two universal dependencies" invariant (osbot-utils + lexicon).

---

## Priority 5: Classify the `to_classify/` Directory

The `Issues-FS__Docs/docs/to_classify/` directory contains some of the most important architecture documents in the entire ecosystem, sitting in an explicitly "unclassified" state:

| Document | Suggested Classification |
|----------|------------------------|
| `v0_4_0__issues-fs__thinking-in-graphs.md` | **Foundational** -- should be in `issues_fs/architecture/` |
| `v0_4_0__issues-fs__lexicon-architecture-v2.md` | **Architecture** -- should be in `issues_fs/architecture/` |
| `v0.1.0__issues-fs__role-based-agent-coordination.md` | **Architecture** -- should be in `issues_fs/architecture/` |
| `v0.1.0__issues-fs__role-architecture-framework-analysis.md` | **Reference** -- supporting analysis for the role architecture |
| `6-feb/v0_4_0__issues-fs__librarian-role.md` | **Role spec** -- should move to Librarian repo |
| `6-feb/v0_4_0__issues-fs__use-case-pattern.md` | **Architecture** -- use case template |
| `6-feb/v0_4_0__issues-fs__use-case__github-backup.md` | **Use case** -- specific use case instance |
| `already-legacy/` | **Archive** -- already flagged as legacy |

This is a natural first task for the Librarian role.

---

## Meta Observations

### The Subagent Capture Problem
This very review demonstrated both the value and the current limitation of the subagent pattern. Two agents ran in parallel, each with deep context, producing rich findings -- but the mechanism for capturing and preserving their outputs was fragile (background task outputs were lost to garbage collection). Issues-FS is designed to solve exactly this: capturing agent work products as persistent, graph-connected nodes.

### Documentation Outpacing Implementation
The ecosystem has a notable pattern: architectural documentation is extensive and high-quality, but implementation trails behind. This is not necessarily a problem -- the documentation serves as a specification and design record. But the gap between "designed" and "built" is something to be aware of, especially for the Lexicon and role repos.

### The Dogfooding Loop
The most powerful accelerator would be completing the loop: use Issues-FS to track the work of building Issues-FS, with AI agents in defined roles creating and managing issues. Each improvement to the system immediately improves the development process. The Librarian and Conductor roles are the critical path for this loop.

### Graph Connectivity of This Review
In graph-first terms, this review has *created edges* between previously disconnected concepts:
- Connected the Librarian's empty repo to its rich documentation
- Connected the DevOps repo (as implementation standard) to the other role repos (as targets)
- Connected the side-capture ideas to the Librarian's draft processing workflow
- Connected the `to_classify/` directory to the Librarian's responsibilities

These connections didn't exist in any single document before. They existed implicitly in the project owner's head. Now they are explicit nodes in the documentation graph.
