---
name: project-scaffold
description: >
  Executes the full project scaffolding task for this repo, step by step with verification
  after each step. Use when the user says "scaffold the project", "run project-scaffold",
  "set up the project structure", "create package READMEs", "set up pre-commit hooks",
  "onboard new developers", or wants to fulfill the Jira acceptance criteria for
  documented folder structure, package READMEs, pre-commit enforcement, and 5-minute onboarding.
---

# Project Scaffold Skill

## Jira Acceptance Criteria (this skill fulfils)
- Repo has documented folder structure for agents, tools, skills, prompts, tests
- Each package has a README explaining purpose, dependencies, and contribution guidelines
- `.env.example` documented; secrets never committed
- Pre-commit hooks for linting and formatting are enforced
- A new developer can clone, build, and run a local example agent within a few minutes

---

## Execution Protocol

Work through each step sequentially. After completing each step, run the listed
**verification commands** and confirm they pass before proceeding to the next step.
If verification fails, fix the issue before moving on. Report pass/fail for each gate.

**Expected branch flow:**
```
main (archived by branching-strategy)
  └── develop
        └── feature/scaffold-setup  ← this skill works here
                                       PR → develop when done
```

---

## Step 0 — Create Feature Branch

**Action:** Before touching any files, ensure all scaffolding work happens on a dedicated
feature branch — never directly on `main` or `develop`.

```bash
# Check the current branch
git branch --show-current

# If not already on develop, switch to it
git checkout develop
git pull origin develop   # sync with remote if it exists

# Create the scaffold feature branch
git checkout -b feature/scaffold-setup
```

**Verification — Step 0:**
```bash
# Confirm we are on the correct branch
current=$(git branch --show-current)
echo "Current branch: $current"
[ "$current" = "feature/scaffold-setup" ] && echo "PASS: on feature/scaffold-setup" || echo "FAIL: wrong branch — do not proceed"

# Confirm develop exists (was created by branching-strategy skill)
git branch | grep -q "develop" && echo "PASS: develop branch exists" || echo "WARN: develop not found — run branching-strategy skill first"
```

If verification fails (not on `feature/scaffold-setup`), stop and fix the branch before continuing.
If `develop` does not exist, ask the user to run the `/branching-strategy` skill first.

---

## Step 1 — Pre-commit Configuration

**Action:** Create `.pre-commit-config.yaml` at the project root.

```yaml
# .pre-commit-config.yaml
# Install once after cloning: pre-commit install
# Manual run: pre-commit run --all-files
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-added-large-files
        args: ['--maxkb=1000']
      - id: detect-private-key
      - id: check-merge-conflict

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.9
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/psf/black
    rev: 24.4.2
    hooks:
      - id: black
        language_version: python3.11

  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.9
    hooks:
      - id: bandit
        args: ["-c", "pyproject.toml"]
        exclude: ^tests/

  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
        exclude: .env.example
```

**Verification — Step 1:**
```bash
# 1a. File exists and is valid YAML
python -c "import yaml; yaml.safe_load(open('.pre-commit-config.yaml'))" && echo "PASS: valid YAML"

# 1b. Contains required hooks
grep -q "ruff" .pre-commit-config.yaml && echo "PASS: ruff present"
grep -q "black" .pre-commit-config.yaml && echo "PASS: black present"
grep -q "bandit" .pre-commit-config.yaml && echo "PASS: bandit present"
grep -q "detect-secrets" .pre-commit-config.yaml && echo "PASS: detect-secrets present"
```

Report results. If all pass, proceed to Step 2.

---

## Step 2 — Package READMEs

**Action:** Create a `README.md` in each of the five packages below.
Each README must cover: Purpose, Structure, Key Dependencies, and Contribution Guidelines.

### 2a — `src/agents/README.md`

Explain: supervisor + sub-agent architecture, how delegation works, how to add a new sub-agent
(create directory → implement builder → register in SUB_AGENTS → add YAML config → update delegation rules).

### 2b — `src/tools/README.md`

Explain: the `@tool` decorator, ToolSpec registry, `agent_tools.json`, the tool contract
(idempotent, raise ToolError, docstring required), and how to add a new tool.

### 2c — `src/skills/README.md`

Explain: SKILL.md format, `build_skill_toolset()`, how skills attach to sub-agents,
required SKILL.md sections (Purpose, Inputs, Outputs, Rules, Examples), and how to add a skill.

### 2d — `src/prompts/README.md`

Explain: `render_prompt()`, Jinja2 usage, where agent-specific prompts live vs shared partials,
and the rule that prompt strings must never be hardcoded in Python files.

### 2e — `tests/README.md`

Explain: directory layout (unit/integration/eval), how to run each suite, pytest markers
(`@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.eval`), the 80% coverage gate,
and a short example unit test for a tool.

**Verification — Step 2:**
```bash
for f in src/agents/README.md src/tools/README.md src/skills/README.md src/prompts/README.md tests/README.md; do
  if [ -f "$f" ]; then
    lines=$(wc -l < "$f")
    echo "PASS: $f exists ($lines lines)"
  else
    echo "FAIL: $f missing"
  fi
done
```

All five must report PASS with at least 20 lines each before proceeding.

---

## Step 3 — Run Script

**Action:** Create `scripts/` directory and `scripts/run_agent.py`.

```python
#!/usr/bin/env python3
"""Launch the Campaign Analyzer Agent via the ADK web server."""

import subprocess
import sys
from pathlib import Path

PROJECT_ROOT = Path(__file__).parent.parent


def main() -> None:
    env_file = PROJECT_ROOT / ".env"
    if not env_file.exists():
        print("ERROR: .env not found. Copy and configure it first:")
        print("  cp .env.example .env")
        sys.exit(1)

    print("Starting Campaign Analyzer Agent at http://localhost:8000 ...")
    subprocess.run(["adk", "web"], cwd=PROJECT_ROOT, check=True)


if __name__ == "__main__":
    main()
```

**Verification — Step 3:**
```bash
# File exists
test -f scripts/run_agent.py && echo "PASS: run_agent.py exists"

# Python syntax is valid
python -c "import ast; ast.parse(open('scripts/run_agent.py').read())" && echo "PASS: valid Python"

# Contains the .env guard
grep -q "env_file" scripts/run_agent.py && echo "PASS: .env guard present"
```

---

## Step 4 — Update Root README

**Action:** Replace the Quickstart section of `README.md` with a **5-step, under-5-minute guide**
that includes: clone, venv, pip install, cp .env.example, pre-commit install, and `python scripts/run_agent.py`.

Preserve all other existing sections (Architecture, Project Layout, Local Skills).

Add a **Project Structure** table that lists every top-level directory with a one-line description.

**Verification — Step 4:**
```bash
# Contains 5-minute onboarding markers
grep -q "pre-commit install" README.md && echo "PASS: pre-commit install mentioned"
grep -q "scripts/run_agent.py" README.md && echo "PASS: run_agent.py referenced"
grep -q "cp .env.example" README.md && echo "PASS: .env setup mentioned"

# Word count sanity (should be substantive)
wc -l README.md
```

---

## Step 5 — Annotate `.env.example`

**Action:** Ensure every variable in `.env.example` has an inline comment explaining:
- What it controls
- Valid values or where to obtain it
- Whether it is required or optional

Variables that currently lack comments: `APP_ENV`, `LOG_LEVEL`, `GOOGLE_GENAI_USE_VERTEXAI`,
`GOOGLE_API_KEY`, `ANTHROPIC_API_KEY`, `GCS_ARTIFACT_BUCKET`, `GA4_PROPERTY_ID`,
and all Snowflake vars.

**Verification — Step 5:**
```bash
# Every non-blank, non-comment line should have a comment on the same line or on the preceding line
python - <<'EOF'
lines = open('.env.example').readlines()
uncommented = [l.strip() for l in lines if l.strip() and not l.startswith('#') and '#' not in l]
if uncommented:
    print(f"WARN: {len(uncommented)} vars may lack inline comments: {uncommented[:3]}")
else:
    print("PASS: all variables have comments")
EOF
```

---

## Step 6 — Commit, Push, and Open PR

**Action:** After all verifications pass:

1. Run `git status` to list changed and new files
2. Confirm we are still on `feature/scaffold-setup`
3. Report a summary table:

| Step | Deliverable | Status |
|------|-------------|--------|
| 0 | Feature branch `feature/scaffold-setup` from `develop` | ✓ |
| 1 | `.pre-commit-config.yaml` | ✓ |
| 2 | 5 × package `README.md` | ✓ |
| 3 | `scripts/run_agent.py` | ✓ |
| 4 | Root `README.md` updated | ✓ |
| 5 | `.env.example` annotated | ✓ |

4. Suggest the commit and PR commands (do NOT execute automatically — confirm with user first):

```bash
# Stage all scaffolding files
git add .pre-commit-config.yaml scripts/run_agent.py README.md .env.example \
        src/agents/README.md src/tools/README.md src/skills/README.md \
        src/prompts/README.md tests/README.md

# Commit on the feature branch
git commit -m "feat(scaffold): add pre-commit hooks, package READMEs, and 5-min onboarding"

# Push feature branch
git push -u origin feature/scaffold-setup
```

5. After the user confirms the push, suggest opening a PR:
   - **From:** `feature/scaffold-setup`
   - **Into:** `develop` (NOT main)
   - **Title:** `feat(scaffold): add pre-commit hooks, package READMEs, and 5-min onboarding`

**Branch state after this skill completes:**
```
main          ← unchanged (existing code preserved)
  archive/pre-prod-baseline  ← frozen snapshot (from branching-strategy)
  develop     ← integration branch (from branching-strategy)
    feature/scaffold-setup  ← PR open → develop
```
