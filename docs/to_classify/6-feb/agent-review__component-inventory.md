# Issues-FS Component Inventory

**Document:** agent-review__component-inventory
**Date:** 2026-02-06
**Source:** AI agent review (Project Reviewer subagent)
**Status:** Raw findings for human review

---

## Issues-FS__Dev Submodules (`modules/`)

| Submodule | GitHub | Package | Purpose | Status |
|-----------|--------|---------|---------|--------|
| **Issues-FS** | `owasp-sbot/Issues-FS` | `issues_fs` | Core library: `Graph__Repository`, `Node__Service`, `Link__Service`, `Type__Service`, `Comments__Service`, MGraph integration, schemas, storage path handlers | Active, working |
| **Issues-FS__CLI** | `owasp-sbot/Issues-FS__CLI` | `issues_fs_cli` | CLI tool (`issues-fs` command): init, create, show, list, update, delete, link, comment, types | Active, working |
| **Issues-FS__Service** | `owasp-sbot/Issues-FS__Service` | `issues_fs_service` | FastAPI REST server: routes for Nodes, Links, Comments, Types, Graph, plus phase_1 routes for Issues and Roots | Active, working |
| **Issues-FS__Service__Client__Python** | `owasp-sbot/Issues-FS__Service__Client__Python` | `issues_fs_service_client_python` | Python API client + all request/response schemas (schema foundation for core and service) | Active, working |
| **Issues-FS__Service__UI** | `owasp-sbot/Issues-FS__Service__UI` | `issues_fs_service_ui` | Web UI: Python FastAPI backend serving the web interface. Has its own `.issues/` with tracked bugs, features, tasks | Active, working |
| **Issues-FS__Docs** | `owasp-sbot/Issues-FS__Docs` | `issues_fs_docs` | Documentation: architecture docs, dev briefs, LLM briefs, type-safety guides | Active, growing rapidly |

## Issues-FS__Dev Role Repos (`roles/`)

| Submodule | GitHub | Purpose | Status |
|-----------|--------|---------|--------|
| **Issues-FS__Dev__Role__DevOps** | `owasp-sbot/Issues-FS__Dev__Role__DevOps` | CI/CD helpers, runbooks (new repo setup, minimum repo files), GitHub workflows | **Implemented** (16 files) |
| **Issues-FS__Dev__Role__Librarian** | `owasp-sbot/Issues-FS__Dev__Role__Librarian` | Documentation curation, knowledge coherence, graph connectivity | **Bare** (LICENSE only) |

## Planned/Designed But Not Yet Created

| Component | Type | Documentation Status |
|-----------|------|---------------------|
| **Issues-FS__Lexicon** | Package/Repo | Extensively designed (v2.0 architecture doc, 5-phase migration plan) |
| **Issues-FS__Dev__Role__Conductor** | Role Repo | Fully specified (ROLE.md template, system prompts, handoff protocols) |
| **Issues-FS__Dev__Role__Architect** | Role Repo | Fully specified |
| **Issues-FS__Dev__Role__Dev** | Role Repo | Fully specified |
| **Issues-FS__Dev__Role__QA** | Role Repo | Fully specified |
| **Issues-FS__Service__GitHub** | Integration | Architecture documented |
| **Issues-FS__Service__Client__JS** | Client | Designed |

---

## Parent: HTML Transformation Workbench

| Attribute | Value |
|-----------|-------|
| **Repo** | `MGraph-AI__UI__Html-Transformation-Workbench` |
| **Package** | `mgraph-ai-ui-html-transformation-workbench` v0.2.34 |
| **Purpose** | Web workbench for loading, transforming, and saving HTML pages with MGraph-AI services |
| **Architecture** | FastAPI backend + vanilla HTML/JS/CSS mini-app (event bus, API client, config manager) |
| **Dependencies** | `osbot-fast-api-serverless`, `memory_fs`, `issues_fs` |
| **Submodules** | Issues-FS__Dev, Memory-FS, OSBot-Fast-API, OSBot-Fast-API-Serverless |

---

## Wider MGraph-AI Ecosystem (~27 repos)

Repos at `/Users/diniscruz/_dev/mgraph-ai/` include:

| Category | Repos |
|----------|-------|
| **Core graph** | MGraph-AI__Service__Graph |
| **HTML** | MGraph-AI__Service__Html__Graph, MGraph-AI__Service__Semantic_Html |
| **Text/NLP** | MGraph-AI__Service__Semantic_Text |
| **LLMs** | MGraph-AI__Service__LLMs |
| **AWS** | MGraph-AI__Service__AWS, MGraph-AI__Service__Comprehend |
| **Infra** | MGraph-AI__Service__Cache, MGraph-AI__Service__Deploy |
| **GitHub** | MGraph-AI__Service__GitHub, MGraph-AI__Service__GitHub__Digest |
| **Tools** | MGraph-AI__Service__mitmproxy, MGraph-AI__Service__PlantUML |
| **Content** | MGraph-AI__Web-Content-Filtering |
| **UI** | MGraph-AI__UI__Html-Transformation-Workbench |
| **Presentations** | Presentation__BlackHat-EU__Dec-2025 |

All follow consistent patterns: Python 3.12+, Poetry, osbot-utils Type_Safe, FastAPI services, GitHub CI/CD.

---

## Key File Locations

| Purpose | Path (relative to Issues-FS__Dev) |
|---------|----------------------------------|
| Submodule definitions | `.gitmodules` |
| Architecture overview | `modules/Issues-FS__Docs/docs/issues_fs/architecture/v0.4.0__issues-fs__architecture-overview.md` |
| Thinking in Graphs | `modules/Issues-FS__Docs/docs/to_classify/v0_4_0__issues-fs__thinking-in-graphs.md` |
| Lexicon Architecture v2 | `modules/Issues-FS__Docs/docs/to_classify/v0_4_0__issues-fs__lexicon-architecture-v2.md` |
| Role-Based Coordination | `modules/Issues-FS__Docs/docs/to_classify/v0.1.0__issues-fs__role-based-agent-coordination.md` |
| Librarian Role doc | `modules/Issues-FS__Docs/docs/to_classify/6-feb/v0_4_0__issues-fs__librarian-role.md` |
| Executive Briefing | `modules/Issues-FS__Docs/docs/issues_fs/llm-briefs/v0.2.14__briefing__graph-based-issue-tracking.md` |
| Core library source | `modules/Issues-FS/issues_fs/` |
| Service routes | `modules/Issues-FS__Service/issues_fs_service/fast_api/routes/` |
| CLI commands | `modules/Issues-FS__CLI/issues_fs_cli/cli/` |
| DevOps runbooks | `roles/Issues-FS__Dev__Role__DevOps/docs/` |

---

## API Surface (FastAPI Service)

| Endpoint Group | Routes |
|---------------|--------|
| `/api/nodes/*` | CRUD for issues/nodes |
| `/api/nodes/{label}/links` | Relationship management |
| `/api/types/*` | Type configuration |
| `/api/graph/*` | Graph queries, export, subgraph, path finding |
| `/api/comments/*` | Comment management |
| `/phase_1/api/issues/*` | Issues endpoints (phase 1) |
| `/phase_1/api/roots/*` | Root node endpoints (phase 1) |

## CLI Commands

`issues-fs init | create | show | list | update | delete | link | comment | types`

Output formats: table, JSON, markdown, agent-optimized
