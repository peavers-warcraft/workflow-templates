# Workflow Templates

Reusable GitHub Actions workflows for WoW addon projects.

## Available Workflows

### 1. `packaging.yml` - Package and Release to CurseForge

Packages an addon using BigWigsMods/packager and releases to CurseForge.

**Usage:**
```yaml
name: Package and Release
on:
  workflow_dispatch:

permissions:
  contents: write
  actions: write

jobs:
  package:
    uses: peavers-warcraft/workflow-templates/.github/workflows/package-and-release.yml@main
    secrets: inherit
```

**Required Secrets:**
- `CF_API_KEY` - CurseForge API key
- `PERSONAL_ACCESS_TOKEN` - GitHub PAT with repo access

---

### 2. `release-on-push.yml` - Version Bump on Push

Automatically increments version, updates TOC file, creates tag, and triggers packaging when code is pushed to master.

**Usage:**
```yaml
name: Update Version and Tag
on:
  push:
    branches:
      - master

permissions:
  contents: write
  actions: write

jobs:
  release:
    uses: peavers-warcraft/workflow-templates/.github/workflows/release-on-push.yml@main
    with:
      toc_file: MyAddon.toc
      addon_name: MyAddon
    secrets: inherit
```

**Inputs:**
| Input | Required | Description |
|-------|----------|-------------|
| `toc_file` | Yes | The TOC file to update (e.g., `MyAddon.toc`) |
| `addon_name` | Yes | The addon name for git tags (e.g., `MyAddon`) |

---

### 3. `release-on-schedule.yml` - Version Bump on Schedule

Same as `release-on-push.yml` but designed for scheduled/data-driven releases.

**Usage:**
```yaml
name: Update Version and Tag
on:
  schedule:
    - cron: '0 */6 * * *'  # Every 6 hours
  workflow_dispatch:

permissions:
  contents: write
  actions: write

jobs:
  release:
    uses: peavers-warcraft/workflow-templates/.github/workflows/release-on-schedule.yml@main
    with:
      toc_file: MyDataAddon.toc
      addon_name: MyDataAddon
    secrets: inherit
```

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `toc_file` | Yes | - | The TOC file to update |
| `addon_name` | Yes | - | The addon name for git tags |
| `packaging_workflow` | No | `packaging.yml` | Workflow filename to trigger |

---

### 4. `cleanup-releases.yml` - Delete Old Releases

Keeps only the most recent release and deletes all older ones.

**Usage:**
```yaml
name: Cleanup Old Releases
on:
  workflow_dispatch:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

permissions:
  contents: write

jobs:
  cleanup:
    uses: peavers-warcraft/workflow-templates/.github/workflows/cleanup-releases.yml@main
    secrets: inherit
```

---

### 5. `auto-merge-pr.yml` - Auto-merge Trusted PRs

Automatically merges PRs from a trusted user. Includes fork protection to prevent malicious code execution on self-hosted runners.

**Usage:**
```yaml
name: Auto-merge Updates
on:
  pull_request:
    types: [opened, synchronize]
    branches:
      - master

permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    uses: peavers-warcraft/workflow-templates/.github/workflows/auto-merge-pr.yml@main
    with:
      allowed_user: peavers
    secrets: inherit
```

**Inputs:**
| Input | Required | Description |
|-------|----------|-------------|
| `allowed_user` | Yes | GitHub username allowed to auto-merge |

**Security Features:**
- Only runs for PRs from the same repository (not forks)
- Only runs for PRs from the specified allowed user
- Uses exponential backoff for merge retries

---

### 6. `validate-issues.yml` - Issue Template Validation

Validates that new issues follow the required templates and auto-closes issues that don't provide required information.

**Usage:**
```yaml
name: Validate Issues
on:
  issues:
    types: [opened]

permissions:
  issues: write

jobs:
  validate:
    uses: peavers-warcraft/workflow-templates/.github/workflows/validate-issues.yml@main
    secrets: inherit
```

**Validation Rules:**
- Detects if issue uses Bug Report or Feature Request template (by title prefix)
- Bug reports must include: WoW version, addon version, bug description, steps to reproduce, expected behavior, Lua error text
- Feature requests must include: problem/use case, proposed solution, priority
- Issues not using a template are auto-closed with a friendly message

**Inputs:**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `use_self_hosted` | No | `true` | Use self-hosted runner |

---

### 7. `sync-issue-templates.yml` - Sync Templates to All Repos

Syncs issue templates from workflow-templates to all addon repositories via pull requests.

**Triggers:**
- Manual dispatch (with optional dry-run mode)
- Automatic on push to `.github/ISSUE_TEMPLATE/**`

**Target Repositories:**
- PeaversTalents
- PeaversCommons
- PeaversDynamicStats
- PeaversRealValue
- PeaversRemembersYou
- PeaversAlwaysSquare
- PeaversTalentsData
- PeaversCurrencyData

**Inputs (workflow_dispatch):**
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `dry_run` | No | `false` | Preview changes without creating PRs |
| `target_repo` | No | `` | Sync to specific repo only (empty = all repos) |

**Required Secrets:**
- `PERSONAL_ACCESS_TOKEN` - GitHub PAT with `repo` scope for cross-repo access

---

## Issue Templates

This repository contains the canonical issue templates used by all addon repositories:

### Bug Report (`bug_report.yml`)
Structured form requiring:
- WoW Version (dropdown: Retail, Classic Era, Cataclysm Classic, etc.)
- Addon Version
- Bug Description
- Steps to Reproduce
- Expected Behavior
- Lua Error Text (or "No error displayed")
- Other Addons (optional)
- Screenshots (optional)

### Feature Request (`feature_request.yml`)
Structured form requiring:
- Problem or Use Case
- Proposed Solution
- Alternatives Considered (optional)
- Priority (dropdown)
- Mockups/Examples (optional)

### Configuration (`config.yml`)
- Disables blank issues
- Provides link to Discord for general questions and support

---

## Required Repository Secrets

All calling repositories should have these secrets configured:

| Secret | Description |
|--------|-------------|
| `CF_API_KEY` | CurseForge API key for addon uploads |
| `PERSONAL_ACCESS_TOKEN` | GitHub PAT with `repo` and `workflow` scopes |

## Runner Configuration

All workflows support both self-hosted and GitHub-hosted runners via the `use_self_hosted` input:

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `use_self_hosted` | No | `true` | Use self-hosted runner (`true`) or GitHub-hosted ubuntu-latest (`false`) |

**Example using GitHub-hosted runners:**
```yaml
jobs:
  package:
    uses: peavers-warcraft/workflow-templates/.github/workflows/package-and-release.yml@main
    with:
      use_self_hosted: false
    secrets: inherit
```

## Notes

- The `github.repository` context variable is used automatically to determine the current repo
- No need to hardcode repository names in caller workflows
