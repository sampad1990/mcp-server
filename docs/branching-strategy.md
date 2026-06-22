---
name: branching-strategy
description: >
  Implements the git branching strategy for this repo, step by step with verification
  after each step. Use when the user says "run branching-strategy", "set up the branching model",
  "tag the baseline", "create develop branch", "archive pre-production code",
  "implement feature/develop/release branching", or wants to fulfil the Jira acceptance
  criteria for pre-production tagging, single source of truth, and the branching convention.
---

# Branching Strategy Skill

## Jira Acceptance Criteria (this skill fulfils)

> **AC1:** Given the team has an existing codebase, when the current code is tagged,
> then it is marked as the pre-production baseline and preserved in an archived branch.
>
> **AC2:** Given the repository is designated as the single source of truth, when development
> begins next sprint, then all new work occurs exclusively in this repository.
>
> **AC3:** Given the team adopts the new branching model, when development starts next sprint,
> then the `feature/`, `develop`, and `release/` branching convention is followed for all work.

---

## Execution Protocol

Work through each step sequentially. After completing each step, run the listed
**verification commands** and confirm they pass before proceeding to the next step.
If verification fails, fix the issue before moving on. Report pass/fail for each gate.

**Important:** Before running any git push commands, confirm explicitly with the user.

---

## Step 1 — Pre-flight Check

**Action:** Confirm the repo is in a clean, pushable state.

```bash
# Check current branch
git branch --show-current

# Check for uncommitted changes
git status --short

# Check remote is configured
git remote -v
```

**Verification — Step 1:**
- If there are uncommitted changes → prompt user to commit or stash before continuing
- If no remote is configured → note this; tag and branch creation will be local only
- If not on `main` → ask user to confirm which branch is the baseline

Report findings. Ask the user to confirm before proceeding to Step 2.

---

## Step 2 — Tag the Pre-Production Baseline (AC1)

**Action:** Create an annotated tag on the current HEAD marking it as the pre-production baseline.

```bash
# Ensure we are on main and up to date
git checkout main
git pull origin main   # skip if no remote

# Create the annotated tag
git tag -a v0.1.0-pre-prod -m "Pre-production baseline — sprint 0 archive"
```

**Verification — Step 2:**
```bash
# Tag exists locally
git tag -l | grep "pre-prod" && echo "PASS: pre-prod tag exists"

# Tag points to the correct commit
git rev-parse v0.1.0-pre-prod
git rev-parse HEAD
echo "(above two SHAs should match)"

# Tag has annotation message
git tag -v v0.1.0-pre-prod 2>/dev/null || git show v0.1.0-pre-prod --format="%H %s" -s
```

After verification passes, ask user: "Tag verified locally. Push tag to remote? (yes/no)"
If yes: `git push origin v0.1.0-pre-prod`

---

## Step 3 — Create the Archive Branch (AC1)

**Action:** Create a permanent archive branch from the tag so the baseline is preserved
as a named branch (tags can be deleted; a protected archive branch cannot).

```bash
# Create archive branch from the tag
git checkout -b archive/pre-prod-baseline v0.1.0-pre-prod

# Return to main
git checkout main
```

**Verification — Step 3:**
```bash
# Archive branch exists locally
git branch | grep "archive/pre-prod-baseline" && echo "PASS: archive branch exists"

# Archive branch points to the same commit as the tag
tag_sha=$(git rev-parse v0.1.0-pre-prod)
branch_sha=$(git rev-parse archive/pre-prod-baseline)
[ "$tag_sha" = "$branch_sha" ] && echo "PASS: branch SHA matches tag SHA" || echo "FAIL: SHA mismatch"

# We are back on main
git branch --show-current
```

After verification passes, ask user: "Archive branch verified locally. Push to remote? (yes/no)"
If yes: `git push origin archive/pre-prod-baseline`

Communicate to the team: **never merge into `archive/*` branches, never delete them.**

---

## Step 4 — Create the `develop` Branch (AC2 + AC3)

**Action:** Create the `develop` integration branch from `main`.
All sprint feature work targets `develop`, never `main` directly.

```bash
git checkout main
git checkout -b develop
```

**Verification — Step 4:**
```bash
# develop branch exists
git branch | grep "develop" && echo "PASS: develop branch exists"

# develop is at same commit as main
main_sha=$(git rev-parse main)
dev_sha=$(git rev-parse develop)
[ "$main_sha" = "$dev_sha" ] && echo "PASS: develop is in sync with main" || echo "FAIL: branches diverged"

# Currently on develop
git branch --show-current
```

After verification passes, ask user: "Push develop to remote? (yes/no)"
If yes: `git push -u origin develop`

---

## Step 5 — Create `CONTRIBUTING.md` (AC3)

**Action:** Create `CONTRIBUTING.md` at the project root documenting the full branching model,
commit format, and PR rules. This is the authoritative reference for the team.

The file must cover:

### 5a. Branch Types Table

| Branch | Pattern | Purpose |
|--------|---------|---------|
| `main` | `main` | Production-ready. Tagged at every release. |
| `develop` | `develop` | Integration branch. All features merge here. |
| `feature/` | `feature/<ticket>-description` | New features and non-emergency fixes. |
| `release/` | `release/<version>` | Release stabilisation. No new features. |
| `hotfix/` | `hotfix/<ticket>-description` | Emergency production fixes. |
| `archive/` | `archive/<label>` | Frozen snapshots. Never merge into these. |

### 5b. Workflow — Starting and Finishing a Feature

```bash
# Start
git checkout develop && git pull origin develop
git checkout -b feature/CAMP-123-add-roi-tool

# Finish — rebase before PR
git fetch origin && git rebase origin/develop
git push -u origin feature/CAMP-123-add-roi-tool
# Open PR → develop
```

### 5c. Workflow — Creating a Release

```bash
git checkout develop
git checkout -b release/1.0.0
# Bump version, update CHANGELOG, fix release bugs only
git push -u origin release/1.0.0
# PR → main AND → develop when ready
git tag -a v1.0.0 -m "Release 1.0.0"
```

### 5d. Merge Rules

- No direct commits to `main` or `develop`
- Every PR requires at least 1 approval and green CI (ruff, black, bandit, tests)
- **Squash-merge** feature branches into develop (clean linear history)
- **Merge commit** (no squash) from release/ and hotfix/ into main (full audit trail)
- Tag `main` after every merge from release/ or hotfix/

### 5e. Commit Message Format

```
<type>(<scope>): <short summary>

Types:  feat | fix | docs | style | refactor | test | chore
Scopes: agents | tools | skills | prompts | core | memory | infra

Examples:
  feat(tools): add bigquery_query with pagination support
  fix(agents): resolve supervisor delegation loop on empty plan
  docs(scaffold): add package READMEs and pre-commit config
```

**Verification — Step 5:**
```bash
# File exists
test -f CONTRIBUTING.md && echo "PASS: CONTRIBUTING.md exists"

# Contains required sections
grep -q "feature/" CONTRIBUTING.md && echo "PASS: feature/ branch documented"
grep -q "develop" CONTRIBUTING.md   && echo "PASS: develop branch documented"
grep -q "release/" CONTRIBUTING.md  && echo "PASS: release/ branch documented"
grep -q "hotfix/" CONTRIBUTING.md   && echo "PASS: hotfix/ branch documented"
grep -q "archive/" CONTRIBUTING.md  && echo "PASS: archive/ branch documented"

wc -l CONTRIBUTING.md
```

---

## Step 6 — Final Summary and Next Steps

**Action:** Run a full state report and present the summary.

```bash
echo "=== Tags ==="
git tag -l

echo "=== Local Branches ==="
git branch -a

echo "=== New Files ==="
git status --short
```

Present this summary table:

| AC | Deliverable | Status |
|----|-------------|--------|
| AC1 | Tag `v0.1.0-pre-prod` | ✓ |
| AC1 | Branch `archive/pre-prod-baseline` | ✓ |
| AC2 | Branch `develop` as integration branch | ✓ |
| AC3 | `CONTRIBUTING.md` with full branching model | ✓ |

**Recommend to the team / repo admin:**
- Set `develop` as the default PR target branch in GitHub / Azure DevOps
- Add branch protection rules for `main`, `develop`, and `archive/*`:
  - Require PR + 1 approval
  - Require CI to pass
  - No force-push
  - No deletion (especially `archive/*`)

**Suggested commit:**
```bash
git add CONTRIBUTING.md
git commit -m "docs(branching): add CONTRIBUTING.md with feature/develop/release/ model"
```
