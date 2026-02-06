# Issues-FS Use Case Pattern: Building Focused Solutions

**Document:** issues-fs__use-case-pattern  
**Version:** v1.0  
**Date:** 2026-02-05  
**Status:** Draft  
**Depends On:** issues-fs__thinking-in-graphs v1.0, issues-fs__architecture-overview v1.0  

---

## Executive Summary

This document defines the **Use Case Pattern** — a meta-pattern for creating focused, minimal, high-value solutions on top of the Issues-FS ecosystem. Each use-case repo is a fractal scope with its own vocabulary, CLI, and clear value proposition. Use cases are designed to be installable, usable, and valuable within minutes — not hours or days.

The naming convention is `Issues-FS__Use_Case__<Name>`, with packages published as `issues-fs-<name>` to PyPI.

---

## Why: The Adoption Problem

### The Gap Between Platform and Value

Issues-FS as a platform is powerful: graphs, fractal scopes, anchor nodes, analysis tools, multiple storage backends. But power doesn't equal adoption. A developer evaluating Issues-FS asks: "What can I do with this in the next 10 minutes that solves a real problem?"

The core library answers: "You can create graphs of issues with typed relationships and..."

That's not a 10-minute answer. That's a "let me read the documentation" answer.

### Use Cases Bridge the Gap

A use case is a **pre-packaged answer** to a specific problem:

- "I want to back up my GitHub issues" → `pip install issues-fs-github-backup`
- "I want to sync Jira and GitHub issues" → `pip install issues-fs-jira-github-sync`
- "I want to visualize my issue dependencies" → `pip install issues-fs-dependency-graph`
- "I want to migrate from Jira to GitHub" → `pip install issues-fs-jira-migration`

Each use case is:
- **Focused** — solves one problem well
- **Minimal** — few dependencies, small surface area
- **Immediate** — value in minutes, not hours
- **Monetizable** — clear value proposition that could justify payment

### Use Cases as Fractal Scopes

In the graph-first model, every use-case repo is a **fractal scope**: it can define its own vocabulary (nodes, edges, types) that extends or specializes the root Lexicon. A backup use case defines `Snapshot`, `Backup_Job`, `Retention_Policy`. These concepts link to Lexicon anchors where relevant but are local to the use case where they're not.

This means use cases are:
- Not just "scripts that use Issues-FS"
- Full participants in the graph ecosystem
- Able to contribute vocabulary back to the Lexicon if concepts prove broadly useful

---

## The Use Case Pattern

### Naming Convention

```
Issues-FS__Use_Case__<Name>
```

Examples:
- `Issues-FS__Use_Case__GitHub_Backup`
- `Issues-FS__Use_Case__Jira_Sync`
- `Issues-FS__Use_Case__Dependency_Graph`
- `Issues-FS__Use_Case__Sprint_Analytics`
- `Issues-FS__Use_Case__Issue_Migration`

The package name drops the `Use_Case` prefix for brevity:
- Package: `issues-fs-github-backup`
- CLI command: `issues-fs-github-backup` or `issues-fs backup` (if installed as plugin)

### Repository Structure

Every use-case repo follows a consistent structure:

```
Issues-FS__Use_Case__<Name>/
│
├── README.md                           → The "3-command promise"
├── setup.py / pyproject.toml           → Package configuration
├── LICENSE
│
├── issues_fs_<name>/                   → Main package
│   │
│   ├── __init__.py
│   ├── cli.py                          → Typer CLI entry point
│   │
│   ├── core/                           → Core functionality
│   │   ├── __init__.py
│   │   ├── <primary_operation>.py      → Main operation (backup, sync, etc.)
│   │   ├── <secondary_operation>.py    → Supporting operations
│   │   └── config.py                   → Configuration handling
│   │
│   ├── vocabulary/                     → Local vocabulary (fractal scope)
│   │   ├── __init__.py
│   │   ├── types.py                    → Local type definitions
│   │   ├── anchors.py                  → Links to Lexicon anchors
│   │   └── vocabulary.json             → Static vocabulary data
│   │
│   └── adapters/                       → External system adapters (if needed)
│       ├── __init__.py
│       ├── github_adapter.py
│       └── jira_adapter.py
│
├── data/                               → Static data files
│   └── default_config.yaml
│
├── docs/                               → Documentation
│   ├── quickstart.md
│   ├── configuration.md
│   └── examples.md
│
└── tests/
    ├── test__core/
    ├── test__cli/
    └── test__vocabulary/
```

### The 3-Command Promise

Every use case README leads with the "3-command promise": what a user can accomplish in 3 commands or fewer.

```markdown
# Issues-FS GitHub Backup

Back up your GitHub issues in 3 commands:

​```bash
pip install issues-fs-github-backup
issues-fs-backup auth --token $GITHUB_TOKEN
issues-fs-backup snapshot owasp-sbot/Issues-FS --output ./backups/
​```

Your issues are now backed up as an Issues-FS graph, queryable and diffable.
```

This is the litmus test for a good use case: if you can't express the core value in 3 commands, the scope is too broad.

### Dependency Principles

Use cases have minimal dependencies:

```
Required:
├── osbot-utils              → Structural patterns
└── issues-fs-lexicon        → Anchor nodes, analysis tools

Optional (depending on use case):
├── issues-fs                → Core library (if operating on local graphs)
├── issues-fs-service-client → API client (if connecting to remote service)
└── <external-sdk>           → GitHub, Jira, etc. (if integrating)
```

A use case should NOT depend on:
- The full `issues-fs-service` (too heavy)
- UI packages
- Unrelated use cases

Each use case is independently installable. Installing `issues-fs-github-backup` doesn't pull in `issues-fs-jira-sync`.

---

## Vocabulary Design

### Local Types

Each use case can define types specific to its domain:

```python
# issues_fs_github_backup/vocabulary/types.py

from issues_fs_lexicon.base_classes import Base__Anchor_Node
from osbot_utils.type_safe          import Type_Safe

class Snapshot(Type_Safe):
    """A point-in-time backup of issues from a source."""
    snapshot_id     : Safe_Id
    source          : Safe_Str              # e.g., "github:owasp-sbot/Issues-FS"
    created_at      : Timestamp_Now
    issue_count     : Safe_UInt
    node_ids        : List[Safe_Id]         # IDs of backed-up nodes
    
class Backup_Job(Type_Safe):
    """A scheduled or triggered backup operation."""
    job_id          : Safe_Id
    source          : Safe_Str
    destination     : Safe_Str__File__Path
    schedule        : Safe_Str              # cron expression or "manual"
    last_run        : Timestamp_Now
    last_snapshot   : Safe_Id               # Reference to most recent Snapshot
    
class Retention_Policy(Type_Safe):
    """Rules for how long to keep backups."""
    keep_last_n     : Safe_UInt             # Keep last N snapshots
    keep_days       : Safe_UInt             # Keep snapshots from last N days
    keep_weekly     : Safe_UInt             # Keep N weekly snapshots
    keep_monthly    : Safe_UInt             # Keep N monthly snapshots
```

These types are **local vocabulary** — they exist in this use case's fractal scope. Other use cases don't see them unless they explicitly import this package.

### Anchor Links

Where local types relate to broader concepts, they link to Lexicon anchors:

```python
# issues_fs_github_backup/vocabulary/anchors.py

from issues_fs_lexicon.anchors.anchors__external import anchor__prov_o

# A Snapshot is a prov:Entity (something that was generated)
# The backup operation is a prov:Activity (something that happened)
# This link enables provenance tracking across the ecosystem

SNAPSHOT_ANCHOR_LINKS = {
    "Snapshot": {
        "similar_to": "prov:Entity",
        "fields_map": {
            "created_at": "prov:generatedAtTime",
            "source": "prov:wasDerivedFrom",
        }
    },
    "Backup_Job": {
        "similar_to": "prov:Activity",
        "fields_map": {
            "last_run": "prov:endedAtTime",
        }
    }
}
```

These links are optional but valuable. They enable:
- Cross-use-case analysis ("show me all prov:Entity nodes across all my projects")
- Interoperability with external systems that understand PROV-O
- Confidence assessment via Lexicon analysis tools

### Vocabulary File

Static vocabulary definitions can live in JSON:

```json
// issues_fs_github_backup/vocabulary/vocabulary.json
{
  "scope": "issues-fs-github-backup",
  "version": "1.0",
  "types": [
    {
      "name": "Snapshot",
      "fields": ["snapshot_id", "source", "created_at", "issue_count", "node_ids"],
      "anchor_links": ["prov:Entity"]
    },
    {
      "name": "Backup_Job",
      "fields": ["job_id", "source", "destination", "schedule", "last_run", "last_snapshot"],
      "anchor_links": ["prov:Activity"]
    },
    {
      "name": "Retention_Policy",
      "fields": ["keep_last_n", "keep_days", "keep_weekly", "keep_monthly"],
      "anchor_links": []
    }
  ],
  "edge_types": [
    {
      "name": "snapshot_of",
      "from": "Snapshot",
      "to": "any",
      "description": "This snapshot contains a backup of the target"
    },
    {
      "name": "produced_by",
      "from": "Snapshot",
      "to": "Backup_Job",
      "description": "This snapshot was produced by this backup job"
    }
  ]
}
```

---

## CLI Design

### Entry Point

Each use case provides a standalone CLI:

```python
# issues_fs_github_backup/cli.py

import typer
from typing import Optional
from pathlib import Path

app = typer.Typer(
    name="issues-fs-github-backup",
    help="Back up GitHub issues as Issues-FS graphs"
)

@app.command()
def auth(token: str = typer.Option(..., envvar="GITHUB_TOKEN")):
    """Authenticate with GitHub."""
    # Store token securely
    ...

@app.command()
def snapshot(
    repo: str = typer.Argument(..., help="Repository in format owner/repo"),
    output: Path = typer.Option("./backup", help="Output directory"),
    labels: Optional[str] = typer.Option(None, help="Filter by labels (comma-separated)"),
):
    """Create a point-in-time backup of issues."""
    ...

@app.command()
def restore(
    snapshot_path: Path = typer.Argument(..., help="Path to snapshot"),
    target: str = typer.Option(..., help="Target: github:owner/repo or local:path"),
    dry_run: bool = typer.Option(False, help="Show what would be restored"),
):
    """Restore issues from a snapshot."""
    ...

@app.command()  
def diff(
    snapshot_a: Path = typer.Argument(..., help="First snapshot"),
    snapshot_b: Path = typer.Argument(..., help="Second snapshot"),
    output: str = typer.Option("table", help="Output format: table, json, markdown"),
):
    """Compare two snapshots and show differences."""
    ...

@app.command()
def schedule(
    repo: str = typer.Argument(...),
    cron: str = typer.Option("0 0 * * *", help="Cron expression"),
    output: Path = typer.Option("./backups"),
    retention: str = typer.Option("keep_last_n:10", help="Retention policy"),
):
    """Schedule automatic backups."""
    ...

if __name__ == "__main__":
    app()
```

### Plugin Integration (Optional)

Use cases can optionally integrate with the main `issues-fs` CLI as plugins:

```python
# In setup.py / pyproject.toml
[project.entry-points."issues_fs.plugins"]
backup = "issues_fs_github_backup.cli:app"
```

This allows:
```bash
issues-fs backup snapshot owasp-sbot/Issues-FS
```

Instead of:
```bash
issues-fs-github-backup snapshot owasp-sbot/Issues-FS
```

Both work; the plugin approach provides a unified CLI namespace.

---

## Design Principles

### 1. Solve One Problem Well

A use case has a single, clear purpose. If you find yourself adding features that don't directly serve the core use case, it's time to create a separate use case.

**Good scope:**
- "Back up GitHub issues"
- "Sync Jira and GitHub issues"
- "Visualize issue dependencies"

**Too broad:**
- "Manage GitHub issues" (what does "manage" mean?)
- "Issue integration platform" (that's the whole ecosystem)

### 2. Value in Minutes, Not Hours

The 3-command promise is real. If a user can't get value within 5-10 minutes, the use case is too complex or poorly documented.

Checklist:
- [ ] Can be installed with `pip install`
- [ ] Requires no configuration to start (sensible defaults)
- [ ] First command produces visible output
- [ ] Core workflow is ≤ 3 commands

### 3. Fail Gracefully with Honest Uncertainty

Use cases inherit the graph-first philosophy of honest uncertainty. When something can't be determined, say so:

```bash
issues-fs-backup snapshot owasp-sbot/Issues-FS

# Output:
# Snapshot created: backup-2026-02-05-001
# Issues backed up: 47
# 
# Confidence notes:
# - 35 issues mapped to Lexicon anchors (high confidence)
# - 12 issues have unknown labels (local-only, not in vocabulary)
# - 2 issues reference deleted users (user nodes marked as 'unknown')
# 
# Run 'issues-fs-backup analyse backup-2026-02-05-001' for details.
```

### 4. Contribute Vocabulary Back

If a use case's local vocabulary proves broadly useful, it should be promoted to the Lexicon. The `Snapshot` concept from the backup use case might become `Lexicon:anchor__snapshot` if other use cases need it.

The path:
1. Define vocabulary locally in use case
2. Use it, refine it, validate it works
3. If multiple use cases need it, propose addition to Lexicon
4. Lexicon maintainers evaluate and potentially adopt
5. Use case updates to link to new Lexicon anchor instead of local definition

### 5. Independent Versioning

Each use case versions independently. `issues-fs-github-backup` v2.0 can coexist with `issues-fs-jira-sync` v1.3. The only coupling is through shared dependencies (osbot-utils, issues-fs-lexicon).

Semantic versioning:
- **Patch** (1.0.x): Bug fixes, no API changes
- **Minor** (1.x.0): New features, backward compatible
- **Major** (x.0.0): Breaking changes to CLI or vocabulary

---

## Candidate Use Cases

Based on the voice memo and ecosystem needs, here are candidate use cases:

### Tier 1: High Value, Clear Scope

| Use Case | Package | Description | 3-Command Promise |
|----------|---------|-------------|-------------------|
| **GitHub Backup** | `issues-fs-github-backup` | Point-in-time backup of GitHub issues | `pip install` → `auth` → `snapshot` |
| **Jira Sync** | `issues-fs-jira-sync` | Bidirectional sync between Jira and Issues-FS | `pip install` → `auth` → `sync` |
| **GitHub Sync** | `issues-fs-github-sync` | Bidirectional sync between GitHub and Issues-FS | `pip install` → `auth` → `sync` |
| **Dependency Graph** | `issues-fs-dependency-graph` | Visualize issue dependencies as a graph | `pip install` → `load` → `visualize` |

### Tier 2: Valuable, Needs Definition

| Use Case | Package | Description |
|----------|---------|-------------|
| **Issue Migration** | `issues-fs-migration` | Migrate issues between platforms (Jira→GitHub, etc.) |
| **Sprint Analytics** | `issues-fs-sprint-analytics` | Velocity, burndown, cycle time analysis |
| **Compliance Audit** | `issues-fs-compliance` | Audit trail, retention, regulatory reporting |
| **Issue Templates** | `issues-fs-templates` | Standardized issue templates with validation |
| **Multi-Repo Dashboard** | `issues-fs-dashboard` | Unified view across multiple repositories |

### Tier 3: Exploratory

| Use Case | Package | Description |
|----------|---------|-------------|
| **AI Issue Triage** | `issues-fs-ai-triage` | ML-based issue classification and routing |
| **Duplicate Detection** | `issues-fs-duplicates` | Find and merge duplicate issues |
| **SLA Monitoring** | `issues-fs-sla` | Track and alert on SLA violations |
| **Issue Forecasting** | `issues-fs-forecast` | Predict issue volume and completion dates |

---

## Example: GitHub Backup Use Case

To illustrate the pattern concretely, here's a more detailed view of the GitHub Backup use case:

### README.md (The Promise)

```markdown
# Issues-FS GitHub Backup

**Never lose your GitHub issues again.**

Back up your GitHub issues as an Issues-FS graph — queryable, diffable, 
and restorable.

## Quick Start

​```bash
pip install issues-fs-github-backup
export GITHUB_TOKEN=your_token_here
issues-fs-backup snapshot owner/repo --output ./backups/
​```

That's it. Your issues are backed up.

## What You Get

- **Point-in-time snapshots**: Every backup is a complete graph of your issues
- **Diff between snapshots**: See what changed between backups
- **Restore capability**: Push issues back to GitHub or another repo
- **Retention policies**: Automatically manage backup history
- **Full graph power**: Query your backups using Issues-FS tools

## Commands

​```bash
issues-fs-backup snapshot owner/repo       # Create a backup
issues-fs-backup diff backup-1 backup-2    # Compare two backups
issues-fs-backup restore backup-1          # Restore from backup
issues-fs-backup schedule owner/repo       # Set up automatic backups
issues-fs-backup list                      # List available backups
​```

## Why This Matters

If you delete a GitHub repo, your issues are gone. If someone maliciously 
or accidentally deletes issues, they're gone. If you need to prove what 
your issues looked like on a specific date for compliance or due diligence, 
you can't — unless you have backups.

This tool solves that.
```

### Core Operation

```python
# issues_fs_github_backup/core/snapshot.py

from issues_fs              import Graph__Repository
from issues_fs_lexicon      import anchor__issue, analyse_connectivity
from github                 import Github
from .vocabulary.types      import Snapshot, Backup_Job

class GitHub_Snapshot:
    
    def __init__(self, github_token: str):
        self.github = Github(github_token)
        
    def create_snapshot(self, repo_name: str, output_path: Path) -> Snapshot:
        """Create a point-in-time backup of a GitHub repo's issues."""
        
        # Create an in-memory Issues-FS graph
        graph = Graph__Repository(storage="memory")
        
        # Fetch issues from GitHub
        repo = self.github.get_repo(repo_name)
        issues = repo.get_issues(state="all")
        
        node_ids = []
        for issue in issues:
            # Create node in graph
            node = graph.create_node()
            node_ids.append(node.id)
            
            # Add edges for issue data
            graph.add_edge(node, "type", "github_issue")
            graph.add_edge(node, "title", issue.title)
            graph.add_edge(node, "body", issue.body)
            graph.add_edge(node, "state", issue.state)
            graph.add_edge(node, "number", issue.number)
            graph.add_edge(node, "created_at", issue.created_at.isoformat())
            graph.add_edge(node, "updated_at", issue.updated_at.isoformat())
            
            # Add labels as edges
            for label in issue.labels:
                graph.add_edge(node, "label", label.name)
            
            # Add assignees as edges
            for assignee in issue.assignees:
                graph.add_edge(node, "assignee", assignee.login)
            
            # Link to Lexicon anchor (partial mapping)
            graph.add_edge(node, "anchor_link", "Lexicon:anchor__issue")
        
        # Persist graph to output path
        graph.save(output_path / f"snapshot-{datetime.now().isoformat()}.json")
        
        # Create and return Snapshot metadata
        snapshot = Snapshot(
            source      = f"github:{repo_name}",
            issue_count = len(node_ids),
            node_ids    = node_ids,
        )
        
        return snapshot
```

### Diff Operation

```python
# issues_fs_github_backup/core/diff.py

from issues_fs_lexicon import analyse_compatibility

class Snapshot_Diff:
    
    def compare(self, snapshot_a: Path, snapshot_b: Path) -> dict:
        """Compare two snapshots and return differences."""
        
        graph_a = Graph__Repository.load(snapshot_a)
        graph_b = Graph__Repository.load(snapshot_b)
        
        # Find nodes by GitHub issue number
        nodes_a = {self._get_issue_number(n): n for n in graph_a.nodes()}
        nodes_b = {self._get_issue_number(n): n for n in graph_b.nodes()}
        
        numbers_a = set(nodes_a.keys())
        numbers_b = set(nodes_b.keys())
        
        added = numbers_b - numbers_a
        removed = numbers_a - numbers_b
        common = numbers_a & numbers_b
        
        # For common issues, check for changes
        changed = []
        for num in common:
            node_a = nodes_a[num]
            node_b = nodes_b[num]
            
            # Use Lexicon compatibility analysis to find differences
            diff = analyse_compatibility(node_a, node_b)
            if diff["divergent_edges_a"] > 0 or diff["divergent_edges_b"] > 0:
                changed.append({
                    "number": num,
                    "changes": diff
                })
        
        return {
            "snapshot_a": str(snapshot_a),
            "snapshot_b": str(snapshot_b),
            "added": list(added),
            "removed": list(removed),
            "changed": changed,
            "summary": {
                "total_a": len(numbers_a),
                "total_b": len(numbers_b),
                "added_count": len(added),
                "removed_count": len(removed),
                "changed_count": len(changed),
            }
        }
```

---

## Monetization Considerations

Use cases are natural monetization points. The pattern supports:

### Free Tier
- Core functionality (snapshot, restore, diff)
- Local storage only
- Manual backups

### Paid Tier
- Scheduled automatic backups
- Cloud storage integration (S3, GCS)
- Retention policy management
- Webhook triggers
- Priority support

### Enterprise Tier
- Multi-repo management
- Compliance reporting
- Audit trails
- SSO integration
- SLA guarantees

The focused nature of use cases makes pricing clear: "GitHub Backup: $X/month for automatic backups of up to N repos."

---

## Creating a New Use Case

### Checklist

- [ ] **Define the 3-command promise** — What's the simplest path to value?
- [ ] **Identify local vocabulary** — What concepts are specific to this use case?
- [ ] **Map to Lexicon anchors** — Which local concepts link to broader vocabulary?
- [ ] **Design the CLI** — Typer commands with clear verbs
- [ ] **Write the README** — Lead with the promise, explain the value
- [ ] **Implement core operations** — Focus on the primary use case first
- [ ] **Add analysis integration** — Use Lexicon tools for confidence/compatibility
- [ ] **Test with real data** — The 3-command promise must work on first try
- [ ] **Document configuration** — What can be customized?
- [ ] **Consider monetization** — What's the free/paid split?

### Template Repository

A template repository `Issues-FS__Use_Case__Template` provides:
- Pre-configured directory structure
- Skeleton CLI with Typer
- Vocabulary template files
- README template with 3-command promise format
- Test scaffolding
- CI/CD configuration for PyPI publishing

---

## Decisions Log

| # | Decision | Rationale |
|---|----------|-----------|
| U1 | **Naming: `Issues-FS__Use_Case__<Name>`** | Clear namespace, consistent with ecosystem conventions. Signals this is a focused solution, not a core component. |
| U2 | **3-command promise as litmus test** | Forces focus. If you can't express value in 3 commands, scope is too broad. |
| U3 | **Independent versioning** | Use cases evolve at different rates. Coupling versions would slow everyone down. |
| U4 | **Local vocabulary with anchor links** | Use cases are fractal scopes. They can define concepts not in Lexicon while still participating in the ecosystem. |
| U5 | **Minimal dependencies** | Each use case should be independently installable without pulling the entire ecosystem. |
| U6 | **Plugin integration optional** | Standalone CLI works everywhere. Plugin integration is nice-to-have for unified namespace. |
| U7 | **Contribute vocabulary back** | Proven local vocabulary should be promoted to Lexicon. This grows the ecosystem organically. |

---

## References

- [Thinking in Graphs: Meaning Through Connectivity](./v0_4_0__issues-fs__thinking-in-graphs.md) — Foundational philosophy
- [Issues-FS Lexicon Architecture](./v0_4_0__issues-fs__lexicon-architecture-v2.md) — Anchor nodes and vocabulary
- [Issues-FS CLI Architecture](./v0_4_0__issues-fs__cli-architecture.md) — CLI design patterns
- [Issues-FS Architecture Overview](./v0_4_0__issues-fs__architecture-overview.md) — Ecosystem architecture
- [Typer](https://typer.tiangolo.com/) — CLI framework

---

*Issues-FS Use Case Pattern v1.0*  
*Date: 2026-02-05*
