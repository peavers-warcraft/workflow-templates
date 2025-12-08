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
    uses: peavers-warcraft/workflow-templates/.github/workflows/packaging.yml@main
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
    uses: peavers-warcraft/workflow-templates/.github/workflows/packaging.yml@main
    with:
      use_self_hosted: false
    secrets: inherit
```

## Notes

- The `github.repository` context variable is used automatically to determine the current repo
- No need to hardcode repository names in caller workflows
