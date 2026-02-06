# Issues-FS GitHub Backup: Use Case Specification

**Document:** issues-fs__use-case__github-backup  
**Version:** v1.0  
**Date:** 2026-02-05  
**Status:** Draft  
**Depends On:** issues-fs__use-case-pattern v1.0, issues-fs__thinking-in-graphs v1.0  

---

## Executive Summary

This document specifies the **GitHub Backup** use case — a focused solution for creating point-in-time backups of GitHub issues as Issues-FS graphs. The use case addresses a real gap: most organizations have no backup of their GitHub issues, and if a repository is deleted or issues are lost, that data is unrecoverable.

The package is `issues-fs-github-backup`, installable via PyPI, with a 3-command path to value.

---

## The Problem

### GitHub Issues Are Not Backed Up

When you use GitHub Issues, your issue data lives on GitHub's servers. GitHub provides:
- No built-in backup mechanism for issues
- No point-in-time snapshots
- No way to see what your issues looked like last month
- No recovery if issues are deleted (accidentally or maliciously)

If you delete a repository, your issues are gone. If someone bulk-deletes issues, they're gone. If you need to prove what your issues contained on a specific date for compliance, due diligence, or legal purposes — you can't.

### The Questions Companies Should Ask

- "If we lost all our GitHub issues tomorrow, how would we recover?"
- "Can we prove what our issues contained on January 15th for the audit?"
- "What changed in our issues between the start and end of the acquisition due diligence?"
- "How do we archive issues from a project that's ending while preserving the full history?"

For most companies, the answer to all of these is: "We can't."

### The Solution

`issues-fs-github-backup` provides:

1. **Point-in-time snapshots** — Capture a complete backup of your issues at any moment
2. **Diff between snapshots** — See exactly what changed between two points in time
3. **Restore capability** — Push issues back to GitHub or to a local Issues-FS instance
4. **Scheduled backups** — Automatic daily/weekly/monthly backups
5. **Retention policies** — Manage backup history automatically
6. **Graph-native storage** — Backups are Issues-FS graphs, fully queryable and analyzable

---

## The 3-Command Promise

```bash
# Install
pip install issues-fs-github-backup

# Authenticate
export GITHUB_TOKEN=ghp_your_token_here

# Backup
issues-fs-backup snapshot owasp-sbot/Issues-FS --output ./backups/
```

Output:
```
Snapshot created: snapshot-2026-02-05T14-30-00
Repository: owasp-sbot/Issues-FS
Issues backed up: 47
Comments backed up: 156
Labels backed up: 12

Stored at: ./backups/snapshot-2026-02-05T14-30-00/

Confidence notes:
- 47 issues with full metadata (high confidence)
- 3 issues reference deleted users (marked as 'user:unknown')
- All labels and milestones captured

Run 'issues-fs-backup show snapshot-2026-02-05T14-30-00' for details.
```

That's it. Your issues are backed up.

---

## Command Reference

### `issues-fs-backup auth`

Authenticate with GitHub.

```bash
# Via environment variable (recommended)
export GITHUB_TOKEN=ghp_your_token_here
issues-fs-backup auth --verify

# Via flag (not recommended for scripts)
issues-fs-backup auth --token ghp_your_token_here

# Via interactive prompt
issues-fs-backup auth --interactive
```

The token is stored securely in `~/.issues-fs/github-backup/credentials` with restricted permissions.

**Required scopes:** `repo` (for private repos) or `public_repo` (for public repos only)

---

### `issues-fs-backup snapshot`

Create a point-in-time backup.

```bash
# Basic snapshot
issues-fs-backup snapshot owner/repo

# With output directory
issues-fs-backup snapshot owner/repo --output ./backups/

# Filter by state
issues-fs-backup snapshot owner/repo --state open
issues-fs-backup snapshot owner/repo --state closed
issues-fs-backup snapshot owner/repo --state all  # default

# Filter by labels
issues-fs-backup snapshot owner/repo --labels bug,critical
issues-fs-backup snapshot owner/repo --labels "help wanted"

# Filter by date
issues-fs-backup snapshot owner/repo --since 2026-01-01
issues-fs-backup snapshot owner/repo --until 2026-01-31

# Include additional data
issues-fs-backup snapshot owner/repo --include-comments  # default: true
issues-fs-backup snapshot owner/repo --include-reactions
issues-fs-backup snapshot owner/repo --include-timeline  # full event history

# Multiple repos
issues-fs-backup snapshot owner/repo1 owner/repo2 owner/repo3

# All repos in an org
issues-fs-backup snapshot --org my-organization
```

**What gets captured:**
- Issue number, title, body, state
- Labels, milestones, assignees
- Author, created_at, updated_at, closed_at
- Comments (with author and timestamp)
- Reactions (optional)
- Timeline events (optional): assignments, label changes, references, etc.

**Output structure:**
```
./backups/snapshot-2026-02-05T14-30-00/
├── metadata.json           # Snapshot metadata
├── graph.json              # Issues-FS graph (nodes and edges)
├── issues/                 # Individual issue files (for human readability)
│   ├── issue-001.json
│   ├── issue-002.json
│   └── ...
└── attachments/            # Downloaded attachments (if --include-attachments)
```

---

### `issues-fs-backup list`

List available snapshots.

```bash
# List all snapshots
issues-fs-backup list

# List snapshots for a specific repo
issues-fs-backup list --repo owner/repo

# List with details
issues-fs-backup list --verbose

# Output as JSON
issues-fs-backup list --output json
```

Output:
```
Snapshots in ./backups/

ID                              Repo                    Issues  Date
──────────────────────────────────────────────────────────────────────
snapshot-2026-02-05T14-30-00    owasp-sbot/Issues-FS    47      2026-02-05 14:30
snapshot-2026-02-01T09-00-00    owasp-sbot/Issues-FS    45      2026-02-01 09:00
snapshot-2026-01-15T09-00-00    owasp-sbot/Issues-FS    42      2026-01-15 09:00

3 snapshots, 2.3 MB total
```

---

### `issues-fs-backup show`

Show details of a snapshot.

```bash
# Summary view
issues-fs-backup show snapshot-2026-02-05T14-30-00

# Show specific issue from snapshot
issues-fs-backup show snapshot-2026-02-05T14-30-00 --issue 23

# Output as JSON
issues-fs-backup show snapshot-2026-02-05T14-30-00 --output json

# Show graph structure
issues-fs-backup show snapshot-2026-02-05T14-30-00 --graph
```

Output:
```
Snapshot: snapshot-2026-02-05T14-30-00
Repository: owasp-sbot/Issues-FS
Created: 2026-02-05 14:30:00 UTC

Issues: 47
  Open: 12
  Closed: 35

Labels: 12
  bug (15 issues)
  enhancement (18 issues)
  documentation (8 issues)
  ...

Milestones: 3
  v1.0 (completed, 20 issues)
  v1.1 (open, 15 issues)
  v2.0 (open, 5 issues)

Comments: 156
Reactions: 89

Graph nodes: 47
Graph edges: 234
Anchor links: 47 (all issues linked to Lexicon:anchor__issue)

Storage: 1.2 MB
```

---

### `issues-fs-backup diff`

Compare two snapshots.

```bash
# Basic diff
issues-fs-backup diff snapshot-2026-01-15 snapshot-2026-02-05

# Output as JSON
issues-fs-backup diff snapshot-2026-01-15 snapshot-2026-02-05 --output json

# Output as markdown (for reports)
issues-fs-backup diff snapshot-2026-01-15 snapshot-2026-02-05 --output markdown

# Show only specific change types
issues-fs-backup diff snapshot-2026-01-15 snapshot-2026-02-05 --only added
issues-fs-backup diff snapshot-2026-01-15 snapshot-2026-02-05 --only removed
issues-fs-backup diff snapshot-2026-01-15 snapshot-2026-02-05 --only changed
```

Output:
```
Diff: snapshot-2026-01-15 → snapshot-2026-02-05

Summary:
  Issues in first snapshot:  42
  Issues in second snapshot: 47
  Added:    5
  Removed:  0
  Changed:  8

Added Issues:
  #43  Implement rate limiting          [enhancement]
  #44  Add unit tests for auth module   [testing]
  #45  Update documentation for v1.1    [documentation]
  #46  Fix memory leak in graph loader  [bug, critical]
  #47  Add CLI progress indicators      [enhancement]

Changed Issues:
  #12  Login fails on mobile
       Status: open → closed
       Labels: +fixed, -investigating
       
  #23  Implement WebSocket support
       Status: open → open
       Assignee: @alice → @bob
       Labels: +in-progress
       
  #31  API rate limiting design
       Status: open → closed
       Added 3 comments
       
  ... (5 more)

Removed Issues: (none)
```

---

### `issues-fs-backup restore`

Restore issues from a snapshot.

```bash
# Dry run (show what would be restored)
issues-fs-backup restore snapshot-2026-02-05 --to github:owner/repo --dry-run

# Restore to same repo
issues-fs-backup restore snapshot-2026-02-05 --to github:owner/repo

# Restore to different repo
issues-fs-backup restore snapshot-2026-02-05 --to github:owner/new-repo

# Restore to local Issues-FS instance
issues-fs-backup restore snapshot-2026-02-05 --to local:./my-issues/

# Restore specific issues only
issues-fs-backup restore snapshot-2026-02-05 --to github:owner/repo --issues 23,24,25

# Restore issues matching a filter
issues-fs-backup restore snapshot-2026-02-05 --to github:owner/repo --labels critical
```

**Restore behavior:**
- Creates new issues (does not overwrite existing)
- Preserves original timestamps in issue body as metadata
- Restores labels (creates if not exists)
- Restores milestones (creates if not exists)
- Restores comments with original author attribution in text
- Links to original issue number in body

**Limitations:**
- Cannot restore to exact same issue numbers (GitHub assigns new numbers)
- Cannot restore as original authors (GitHub API limitation)
- Reactions are not restorable via API

---

### `issues-fs-backup schedule`

Set up automatic scheduled backups.

```bash
# Daily backups at midnight
issues-fs-backup schedule owner/repo --cron "0 0 * * *"

# Weekly backups on Sunday at 2am
issues-fs-backup schedule owner/repo --cron "0 2 * * 0"

# With retention policy
issues-fs-backup schedule owner/repo \
    --cron "0 0 * * *" \
    --keep-last 30 \
    --keep-weekly 12 \
    --keep-monthly 12

# With output location
issues-fs-backup schedule owner/repo \
    --cron "0 0 * * *" \
    --output s3://my-bucket/backups/

# List scheduled jobs
issues-fs-backup schedule --list

# Remove a scheduled job
issues-fs-backup schedule --remove owner/repo
```

**Retention policy options:**
- `--keep-last N` — Keep the last N snapshots
- `--keep-daily N` — Keep N daily snapshots
- `--keep-weekly N` — Keep N weekly snapshots (oldest of each week)
- `--keep-monthly N` — Keep N monthly snapshots (oldest of each month)

**Storage backends:**
- Local filesystem (default)
- S3: `--output s3://bucket/path/`
- GCS: `--output gs://bucket/path/`

---

### `issues-fs-backup analyse`

Run analysis on a snapshot using Lexicon tools.

```bash
# Connectivity analysis
issues-fs-backup analyse snapshot-2026-02-05 --connectivity

# Pattern coverage (how well do issues match Lexicon patterns)
issues-fs-backup analyse snapshot-2026-02-05 --patterns

# Gap analysis (what's missing)
issues-fs-backup analyse snapshot-2026-02-05 --gaps

# Compare with live GitHub state
issues-fs-backup analyse snapshot-2026-02-05 --compare-live
```

Output (connectivity):
```
Connectivity Analysis: snapshot-2026-02-05

Total nodes: 47
  Linked to Lexicon:anchor__issue: 47 (100%)
  
Field coverage against anchor__issue:
  title:       47/47 (100%)
  body:        47/47 (100%)
  status:      47/47 (100%)  [mapped from GitHub 'state']
  priority:    0/47  (0%)    [GitHub has no native priority]
  assigned_to: 23/47 (49%)
  
Confidence: high
  All issues have core fields mapped to Lexicon anchor.
  Priority field is not captured (GitHub limitation).
```

---

## Local Vocabulary

### Types

```python
class Snapshot(Type_Safe):
    """A point-in-time backup of GitHub issues."""
    snapshot_id     : Safe_Id
    source_repo     : Safe_Str              # "owner/repo"
    source_type     : Safe_Str = "github"
    created_at      : Timestamp_Now
    issue_count     : Safe_UInt
    comment_count   : Safe_UInt
    label_count     : Safe_UInt
    storage_path    : Safe_Str__File__Path
    storage_size    : Safe_UInt             # bytes
    
class Backup_Job(Type_Safe):
    """A scheduled backup job."""
    job_id          : Safe_Id
    source_repo     : Safe_Str
    schedule        : Safe_Str              # cron expression
    output_path     : Safe_Str
    retention       : Retention_Policy
    last_run        : Timestamp_Now
    last_snapshot   : Safe_Id
    next_run        : Safe_Str              # ISO timestamp
    enabled         : bool = True
    
class Retention_Policy(Type_Safe):
    """Rules for backup retention."""
    keep_last       : Safe_UInt = 10
    keep_daily      : Safe_UInt = 7
    keep_weekly     : Safe_UInt = 4
    keep_monthly    : Safe_UInt = 12
```

### Anchor Links

```python
ANCHOR_LINKS = {
    "Snapshot": {
        "similar_to": ["prov:Entity"],
        "field_mappings": {
            "created_at": "prov:generatedAtTime",
            "source_repo": "prov:wasDerivedFrom",
        }
    },
    "Backup_Job": {
        "similar_to": ["prov:Activity", "Lexicon:anchor__workflow"],
        "field_mappings": {
            "last_run": "prov:endedAtTime",
            "schedule": "workflow:trigger",
        }
    },
    "github_issue": {
        "similar_to": ["Lexicon:anchor__issue"],
        "field_mappings": {
            "title": "issue:title",
            "body": "issue:description",
            "state": "issue:status",
            "labels": "issue:labels",
            "assignees": "issue:assigned_to",
        }
    }
}
```

---

## Technical Design

### Graph Structure

Each snapshot produces an Issues-FS graph with this structure:

```
Snapshot node
    ├── type ──→ "Snapshot"
    ├── source_repo ──→ "owasp-sbot/Issues-FS"
    ├── created_at ──→ "2026-02-05T14:30:00Z"
    ├── contains ──→ Issue-1
    ├── contains ──→ Issue-2
    ├── contains ──→ ...
    └── anchor_link ──→ "prov:Entity"

Issue-1 node
    ├── type ──→ "github_issue"
    ├── number ──→ 1
    ├── title ──→ "Initial setup"
    ├── body ──→ "..."
    ├── state ──→ "closed"
    ├── created_at ──→ "2025-01-15T10:00:00Z"
    ├── author ──→ User-alice
    ├── label ──→ Label-setup
    ├── label ──→ Label-documentation
    ├── milestone ──→ Milestone-v1.0
    ├── has_comment ──→ Comment-1
    ├── has_comment ──→ Comment-2
    └── anchor_link ──→ "Lexicon:anchor__issue"

Comment-1 node
    ├── type ──→ "github_comment"
    ├── body ──→ "Looks good!"
    ├── author ──→ User-bob
    ├── created_at ──→ "2025-01-16T09:00:00Z"
    └── belongs_to ──→ Issue-1

User-alice node
    ├── type ──→ "github_user"
    ├── login ──→ "alice"
    └── url ──→ "https://github.com/alice"

Label-setup node
    ├── type ──→ "github_label"
    ├── name ──→ "setup"
    └── color ──→ "0366d6"

Milestone-v1.0 node
    ├── type ──→ "github_milestone"
    ├── title ──→ "v1.0"
    ├── state ──→ "closed"
    └── due_on ──→ "2025-06-01"
```

### Diff Algorithm

Comparing two snapshots:

1. **Index by issue number** — Build a map of `number → node` for each snapshot
2. **Set operations** — Find added (in B not A), removed (in A not B), common (in both)
3. **Deep compare common** — For each common issue, compare all edges
4. **Use Lexicon analysis** — `analyse_compatibility(node_a, node_b)` gives structural diff
5. **Aggregate changes** — Group by change type for human-readable output

### Storage Backends

```python
class Storage_Backend(Protocol):
    def save(self, snapshot: Snapshot, graph: Graph) -> str: ...
    def load(self, snapshot_id: str) -> tuple[Snapshot, Graph]: ...
    def list(self) -> list[Snapshot]: ...
    def delete(self, snapshot_id: str) -> bool: ...

class Local_Storage(Storage_Backend):
    """Store snapshots on local filesystem."""
    ...

class S3_Storage(Storage_Backend):
    """Store snapshots in AWS S3."""
    ...

class GCS_Storage(Storage_Backend):
    """Store snapshots in Google Cloud Storage."""
    ...
```

### Rate Limiting

GitHub API has rate limits. The backup tool handles this:

```python
# Check rate limit before starting
rate_limit = github.get_rate_limit()
if rate_limit.core.remaining < estimated_requests:
    wait_time = rate_limit.core.reset - datetime.now()
    print(f"Rate limit low. Waiting {wait_time} or use --ignore-rate-limit")
    
# Exponential backoff on 403
for attempt in range(max_retries):
    try:
        response = github.get_issues(...)
    except RateLimitExceededException:
        wait = 2 ** attempt
        print(f"Rate limited. Waiting {wait}s...")
        time.sleep(wait)
```

---

## Configuration

### Config File

`~/.issues-fs/github-backup/config.yaml`:

```yaml
# Default output location
output:
  path: ~/backups/github-issues/
  format: graph+json  # graph, json, both

# Default retention policy
retention:
  keep_last: 10
  keep_weekly: 4
  keep_monthly: 12

# Default snapshot options
snapshot:
  include_comments: true
  include_reactions: false
  include_timeline: false
  include_attachments: false

# Storage backend
storage:
  backend: local  # local, s3, gcs
  s3:
    bucket: my-backup-bucket
    prefix: github-issues/
    region: us-east-1
  gcs:
    bucket: my-backup-bucket
    prefix: github-issues/

# Notification on scheduled backup
notifications:
  on_success: false
  on_failure: true
  email: alerts@example.com
  slack_webhook: https://hooks.slack.com/...
```

---

## Use Case Scenarios

### Scenario 1: Ad-hoc Backup Before Major Change

```bash
# Before deleting old issues or restructuring
issues-fs-backup snapshot owner/repo --output ./pre-cleanup-backup/

# Do the cleanup...

# If something goes wrong
issues-fs-backup restore ./pre-cleanup-backup/snapshot-* --to github:owner/repo
```

### Scenario 2: Compliance/Audit Trail

```bash
# Set up daily backups with long retention
issues-fs-backup schedule owner/repo \
    --cron "0 1 * * *" \
    --keep-daily 90 \
    --keep-monthly 36 \
    --output s3://compliance-backups/

# When auditor asks "what did issue #45 contain on March 15?"
issues-fs-backup list --repo owner/repo --around 2026-03-15
issues-fs-backup show snapshot-2026-03-15 --issue 45
```

### Scenario 3: Due Diligence for Acquisition

```bash
# Snapshot at start of due diligence
issues-fs-backup snapshot target-company/main-product \
    --output ./due-diligence/start/

# Snapshot at end
issues-fs-backup snapshot target-company/main-product \
    --output ./due-diligence/end/

# Generate diff report for lawyers
issues-fs-backup diff ./due-diligence/start/snapshot-* ./due-diligence/end/snapshot-* \
    --output markdown > ./due-diligence/changes-report.md
```

### Scenario 4: Project Archive

```bash
# Project ending, archive everything
issues-fs-backup snapshot owner/completed-project \
    --include-comments \
    --include-reactions \
    --include-timeline \
    --output ./archives/completed-project/

# Verify the archive
issues-fs-backup analyse ./archives/completed-project/snapshot-* --connectivity

# Optional: delete the GitHub repo knowing issues are preserved
```

### Scenario 5: Cross-Repo Analysis

```bash
# Create scratch graph with multiple repos
issues-fs scratch create multi-repo-analysis

issues-fs-backup snapshot org/repo1 --to-scratch multi-repo-analysis
issues-fs-backup snapshot org/repo2 --to-scratch multi-repo-analysis
issues-fs-backup snapshot org/repo3 --to-scratch multi-repo-analysis

# Now analyse across all repos
issues-fs scratch switch multi-repo-analysis
issues-fs list --label critical  # All critical issues across all 3 repos
issues-fs analyse --scope . --patterns  # Pattern coverage across repos
```

---

## Pricing Model (Proposed)

### Free Tier
- Manual snapshots (unlimited)
- Local storage only
- Diff and restore
- Single repo at a time

### Pro ($9/month)
- Scheduled automatic backups
- Up to 10 repos
- S3/GCS storage
- Email notifications

### Team ($29/month)
- Unlimited repos
- Organization-wide backups (`--org`)
- Retention policy management
- Slack notifications
- Priority support

### Enterprise (Contact)
- Self-hosted option
- SSO integration
- Compliance reporting (SOC2, etc.)
- Audit log export
- SLA guarantee

---

## Decisions Log

| # | Decision | Rationale |
|---|----------|-----------|
| G1 | **Store as Issues-FS graph, not raw JSON** | Enables analysis, querying, and compatibility with the broader ecosystem. Raw JSON would be a dead archive. |
| G2 | **Link all issues to Lexicon:anchor__issue** | Provides confidence and enables cross-project analysis. GitHub issues become first-class ecosystem citizens. |
| G3 | **Include human-readable issue files alongside graph** | Graph is for machines; individual JSON files are for humans who want to inspect specific issues. |
| G4 | **Diff uses Lexicon compatibility analysis** | Consistent with ecosystem philosophy. Diff is a graph operation, not a text diff. |
| G5 | **Restore creates new issues, not overwrites** | GitHub doesn't allow overwriting issues. Creating new issues with metadata about originals is the safe approach. |
| G6 | **Support multiple storage backends** | Local is fine for individuals; teams need S3/GCS for durability and sharing. |
| G7 | **Retention policies are built-in** | Backup without retention leads to unbounded storage growth. Make it easy to do the right thing. |

---

## References

- [Issues-FS Use Case Pattern](./v0_4_0__issues-fs__use-case-pattern.md) — Meta-pattern this follows
- [Issues-FS CLI Architecture](./v0_4_0__issues-fs__cli-architecture.md) — CLI design patterns
- [Issues-FS Lexicon Architecture](./v0_4_0__issues-fs__lexicon-architecture-v2.md) — Anchor nodes
- [Thinking in Graphs](./v0_4_0__issues-fs__thinking-in-graphs.md) — Foundational philosophy
- [GitHub REST API](https://docs.github.com/en/rest) — GitHub API reference
- [PyGithub](https://pygithub.readthedocs.io/) — Python GitHub library

---

*Issues-FS GitHub Backup Use Case v1.0*  
*Date: 2026-02-05*
