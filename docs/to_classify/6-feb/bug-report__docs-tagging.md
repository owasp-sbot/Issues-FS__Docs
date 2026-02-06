# Bug Report: Issues-FS__Docs is Not Creating New Tags on Dev Branch Commits

**Date:** 2026-02-06
**Reporter:** Librarian Agent (automated investigation)
**Repo:** `owasp-sbot/Issues-FS__Docs`
**Severity:** Medium -- CI pipeline is non-functional, blocking automated versioning

---

## 1. Bug Description

The `Issues-FS__Docs` repository has 10 commits on the `dev` branch but only one tag (`v0.1.0`), which was created on the initial commit on `main`. The CI pipeline (`ci-pipeline__dev.yml`) is configured with `should_increment_tag: true` and `release_type: 'minor'`, but no new tags are being created on any subsequent commits to `dev`.

The `increment-tag` job in the base `ci-pipeline.yml` depends on the `run-tests` job succeeding first (`needs: [run-tests]`). If tests fail, the tag increment step is never reached.

---

## 2. Investigation Findings

### 2.1 CI Workflow Configuration (CORRECT)

The three workflow files are correctly configured and match the working DevOps repo pattern:

- **`ci-pipeline__dev.yml`** -- Triggers on push to `dev`, calls base pipeline with `should_increment_tag: true`, `release_type: 'minor'`
- **`ci-pipeline__main.yml`** -- Triggers on push to `main`, calls base pipeline with `should_increment_tag: true`, `release_type: 'major'`, `should_publish_pypi: true`
- **`ci-pipeline.yml`** (base) -- Defines `run-tests` -> `increment-tag` -> `publish-to-pypi` job chain. Sets `PACKAGE_NAME: 'issues_fs_docs'`

The `increment-tag` job has `needs: [run-tests]`, meaning it will only run if the `run-tests` job succeeds.

### 2.2 Missing Files (ROOT CAUSE)

Compared against the DevOps runbook (`runbook__repo-minimums.md`) and the working `Issues-FS__Dev__Role__DevOps` repo, the following required files are **missing** from `Issues-FS__Docs`:

| File | Issues-FS__Docs | DevOps (working) |
|------|----------------|-------------------|
| `pyproject.toml` | MISSING | Present |
| `requirements-test.txt` | MISSING | Present |
| `scripts/gh-release-to-main.sh` | MISSING | Present |

### 2.3 Files That DO Exist (CORRECT)

These files are present and correctly structured:

- `.github/workflows/ci-pipeline.yml` -- correct
- `.github/workflows/ci-pipeline__dev.yml` -- correct
- `.github/workflows/ci-pipeline__main.yml` -- correct
- `issues_fs_docs/__init__.py` -- correct (sets `package_name` and `path`)
- `issues_fs_docs/version` -- contains `v0.1.0`
- `issues_fs_docs/utils/__init__.py` -- present
- `issues_fs_docs/utils/Version.py` -- correct (imports `issues_fs_docs`, reads version file)
- `tests/unit/utils/test_Version.py` -- correct (tests Version class)

### 2.4 Git History Analysis

```
0aaa97d (HEAD -> dev, origin/dev) added agent review docs
8c26deb add 3 new docs to_classify
3a0280f refactor out doc
95d50f4 added 5 docs
1445ec9 refactored in multiple docs files
2433a2d added type safe briefs
b1853df Merge commit into dev
e742e76 added CI pipeline and versioning .github actions   <-- CI added here
70916cf added two foundational docs
ff4f208 started adding docs about this project
4a06feb (tag: v0.1.0, main) Initial commit               <-- only tag
```

The CI pipeline was added at commit `e742e76`. There have been 6 commits after that on `dev`, and zero new tags were created by any of them.

### 2.5 Remote Tags

Only one tag exists on the remote:
```
24683b6 refs/tags/v0.1.0  (points to initial commit 4a06feb)
```

---

## 3. Root Cause Analysis

**Primary cause: Missing `pyproject.toml` and `requirements-test.txt`**

The CI pipeline's `run-tests` job uses the shared action `owasp-sbot/OSBot-GitHub-Actions/.github/actions/pytest__run-tests@dev`. This action needs to:

1. Install Python dependencies (likely reads from `pyproject.toml` or `requirements-test.txt`)
2. Run `pytest tests/unit`

Without `pyproject.toml`, the action cannot install the package (`issues_fs_docs`) as a Python package, so `import issues_fs_docs` in `test_Version.py` will fail with a `ModuleNotFoundError`.

Without `requirements-test.txt`, the action cannot install test dependencies (`pytest`, `pytest-cov`, `osbot-utils`).

**Chain of failure:**

```
run-tests FAILS (cannot install deps / import errors)
    -> increment-tag SKIPPED (needs: [run-tests] not satisfied)
        -> No new tag created
```

This is consistent with the observed behavior: commits go through to `dev` (git push works fine), but the CI pipeline's test job fails, which blocks the downstream tag increment job from ever executing.

**Secondary issue: No `scripts/gh-release-to-main.sh`**

This is not directly related to the tagging bug but is another missing piece per the runbook. It would be needed for the release-to-main workflow.

---

## 4. Possible Fixes (Ranked by Likelihood of Impact)

### Fix 1 (CRITICAL): Add `pyproject.toml`

Create `pyproject.toml` in the repo root:

```toml
[tool.poetry]
name        = "issues_fs_docs"
version     = "v0.1.0"
description = "Issues-FS__Docs"
authors     = ["Dinis Cruz <dinis.cruz@owasp.org>"]
license     = "Apache 2.0"
readme      = "README.md"
homepage    = "https://github.com/owasp-sbot/Issues-FS__Docs"
repository  = "https://github.com/owasp-sbot/Issues-FS__Docs"

[tool.poetry.dependencies]
python            = "^3.12"
osbot-utils       = "*"

[build-system]
requires = ["poetry-core>=1.9.1"]
build-backend = "poetry.core.masonry.api"
```

### Fix 2 (CRITICAL): Add `requirements-test.txt`

Create `requirements-test.txt` in the repo root:

```
osbot-utils

# for testing
pytest
pytest-cov
```

### Fix 3 (RECOMMENDED): Add `scripts/gh-release-to-main.sh`

Copy from the DevOps role repo to complete the standard repo layout.

### Fix 4 (VERIFY): Confirm CI runs after fixes

After adding the missing files, push to `dev` and verify:
- `run-tests` job passes
- `increment-tag` job runs and creates a new tag (should be `v0.2.0` given `release_type: 'minor'`)

---

## 5. Comparison with Working Repos

### Issues-FS__Dev__Role__DevOps (WORKING -- has tags up to v0.4.2)

| Aspect | DevOps (working) | Issues-FS__Docs (broken) |
|--------|------------------|--------------------------|
| `pyproject.toml` | Present | **MISSING** |
| `requirements-test.txt` | Present (`osbot-utils`, `pytest`, `pytest-cov`) | **MISSING** |
| `scripts/gh-release-to-main.sh` | Present | **MISSING** |
| CI workflow files | 3 files (base, dev, main) | 3 files (base, dev, main) -- identical structure |
| Package `__init__.py` | Present | Present |
| `version` file | Present (`v0.1.0`) | Present (`v0.1.0`) |
| `utils/Version.py` | Present | Present |
| `tests/unit/utils/test_Version.py` | Present | Present |
| `PACKAGE_NAME` in CI | `issues_fs_dev_role_devops` | `issues_fs_docs` |
| Tags on remote | Multiple (up to v0.4.2+) | Only `v0.1.0` |

### Key Difference

The DevOps repo has all the packaging infrastructure (`pyproject.toml`, `requirements-test.txt`) needed for the `pytest__run-tests` GitHub Action to install dependencies and run tests successfully. Issues-FS__Docs is missing these files, causing test failures that block the tag increment step.

The CI workflow YAML files themselves are identical in structure between the two repos. The problem is not in the CI configuration but in the **missing supporting files** that the CI depends on.

---

## 6. Summary

The bug is caused by two missing files (`pyproject.toml` and `requirements-test.txt`) that prevent the CI test job from succeeding. Since the tag increment job depends on tests passing, no new tags are ever created. Adding these two files should immediately fix the issue.
