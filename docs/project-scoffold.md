---
name: project-scaffold
description: >
  Executes the full project scaffolding task for this repo, step by step with verification
  after each step. Idempotent — skips anything that already exists, only creates what is missing.
  Use when the user says "scaffold the project", "run project-scaffold", "set up the project
  structure", "create package READMEs", "set up pre-commit hooks", "onboard new developers",
  or wants to fulfill the Jira acceptance criteria for documented folder structure, package
  READMEs, pre-commit enforcement, and 5-minute onboarding. Safe to re-run at any time.
---

# Project Scaffold Skill

## Jira Acceptance Criteria (this skill fulfils)
- Repo has documented folder structure for agents, tools, skills, prompts, tests
- Each package has a README explaining purpose, dependencies, and contribution guidelines
- `.env.example` documented; secrets never committed
- Pre-commit hooks for linting and formatting are enforced
- A new developer can clone, build, and run a local example agent within a few minutes

## Idempotency Rule
**Before creating any file or directory, always check if it already exists.**
- Existing files → print `EXISTS (skipped): <path>` — never overwrite
- Missing files → create, then print `CREATED: <path>`
- This makes the skill safe to re-run on a partially-scaffolded repo

---

## Full Expected Project Structure

Legend: `[E]` = already exists in repo · `[M]` = missing, this skill creates it

```
campaign_analyzer_agent/
├── [E] .env.example                         # Runtime secrets template
├── [E] .gitignore
├── [M] .pre-commit-config.yaml              # ← CREATED by Step 1
├── [E] pyproject.toml
├── [E] requirements.txt
├── [E] README.md                            # ← UPDATED by Step 5
├── [E] CODE_REVIEW.md
│
├── [M] scripts/                             # ← CREATED by Step 2
│   └── [M] run_agent.py
│
├── [E] configs/
│   ├── [E] README.md
│   ├── [E] base.yaml
│   ├── [E] development.yaml
│   ├── [E] logging.yaml
│   ├── [E] production.yaml
│   ├── [E] staging.yaml
│   └── [E] agents/
│       ├── [E] supervisor.yaml
│       ├── [E] campaign_performance.yaml
│       ├── [E] campaign_engagement.yaml
│       ├── [E] campaign_library.yaml
│       └── [E] audience_insights.yaml
│
├── [E] src/
│   ├── [E] __init__.py
│   ├── [E] __version__.py
│   ├── [E] main.py
│   │
│   ├── [E] agents/
│   │   ├── [E] __init__.py
│   │   ├── [E] agent.py
│   │   ├── [M] README.md                   # ← CREATED by Step 3a
│   │   ├── [E] supervisor/
│   │   │   ├── [E] __init__.py
│   │   │   ├── [E] agent.py
│   │   │   ├── [E] callbacks.py
│   │   │   ├── [E] guardrails.py
│   │   │   └── [E] prompts/
│   │   │       ├── [E] system.md
│   │   │       ├── [E] planner.md
│   │   │       ├── [E] delegation_rules.md
│   │   │       └── [E] few_shots.yaml
│   │   └── [E] sub_agents/
│   │       ├── [E] __init__.py
│   │       ├── [E] campaign_performance/
│   │       │   ├── [E] __init__.py
│   │       │   ├── [E] agent.py
│   │       │   ├── [E] schemas.py
│   │       │   └── [E] prompts/
│   │       │       └── [E] system.md
│   │       ├── [E] audience_insights/
│   │       │   ├── [E] __init__.py
│   │       │   ├── [E] agent.py
│   │       │   ├── [E] schemas.py
│   │       │   └── [E] prompts/
│   │       │       └── [E] system.md
│   │       ├── [E] campaign_library/
│   │       │   ├── [E] __init__.py
│   │       │   ├── [E] agent.py
│   │       │   ├── [E] schemas.py
│   │       │   ├── [E] tools.py
│   │       │   └── [E] prompts/
│   │       │       └── [E] system.md
│   │       └── [E] campaign_engagement/
│   │           ├── [E] __init__.py
│   │           ├── [E] agent.py
│   │           ├── [E] schemas.py
│   │           ├── [E] tools.py
│   │           └── [E] prompts/
│   │               └── [E] system.md
│   │
│   ├── [E] core/
│   │   ├── [E] __init__.py
│   │   ├── [E] config.py
│   │   ├── [E] constants.py
│   │   ├── [E] types.py
│   │   ├── [E] errors.py
│   │   ├── [E] exceptions.py
│   │   ├── [E] logging.py
│   │   └── [E] telemetry.py
│   │
│   ├── [E] memory/
│   │   ├── [E] __init__.py
│   │   ├── [E] state_keys.py
│   │   ├── [E] memory_service.py
│   │   ├── [E] session_service.py
│   │   ├── [E] artifact_service.py
│   │   └── [E] stores/
│   │       ├── [E] __init__.py
│   │       ├── [E] redis_store.py
│   │       └── [E] vertex_rag_store.py
│   │
│   ├── [E] tools/
│   │   ├── [E] __init__.py
│   │   ├── [E] _base.py
│   │   ├── [E] registry.py
│   │   ├── [E] agent_tools.json
│   │   ├── [E] bigquery_tool.py
│   │   ├── [E] get_campaign_tool.py
│   │   ├── [E] get_campaign_engagement.py
│   │   ├── [E] planner_tool.py
│   │   └── [M] README.md                   # ← CREATED by Step 3b
│   │
│   ├── [E] integrations/
│   │   ├── [E] __init__.py
│   │   └── [E] bigquery_client.py
│   │
│   ├── [E] snowflake/
│   │   ├── [E] __init__.py
│   │   ├── [E] client.py
│   │   └── [E] decorators.py
│   │
│   ├── [E] skills/
│   │   ├── [E] __init__.py
│   │   ├── [E] toolset.py
│   │   ├── [M] README.md                   # ← CREATED by Step 3c
│   │   └── [E] catalog/
│   │       ├── [E] campaign-performance/
│   │       │   └── [E] SKILL.md
│   │       └── [E] audience-insights/
│   │           └── [E] SKILL.md
│   │
│   └── [E] prompts/
│       ├── [E] __init__.py
│       ├── [E] loader.py
│       ├── [M] README.md                   # ← CREATED by Step 3d
│       ├── [E] personas/
│       │   └── [E] analyst.md
│       └── [E] partials/
│           ├── [E] output_format.md
│           └── [E] safety.md
│
├── [E] docs/
│   ├── [E] index.md
│   ├── [E] agents/
│   │   ├── [E] supervisor.md
│   │   └── [E] sub_agents.md
│   ├── [E] tools/
│   │   └── [E] catalog.md
│   ├── [E] api/
│   │   └── [E] .gitkeep
│   ├── [E] runbooks/
│   │   ├── [E] incident_response.md
│   │   └── [E] on_call.md
│   └── [E] architecture/
│       ├── [E] overview.md
│       ├── [E] agent_topology.md
│       ├── [E] data_flow.md
│       └── [E] adr/
│           ├── [E] 0001-adopt-adk-2.0.0.md
│           ├── [E] 0002-supervisor-pattern.md
│           └── [E] 0003-memory-backend.md
│
└── [E+M] tests/
    ├── [E] __init__.py
    ├── [M] README.md                        # ← CREATED by Step 3e
    ├── [M] unit/                            # ← CREATED by Step 4
    │   ├── [M] __init__.py
    │   ├── [M] test_tools/
    │   │   └── [M] __init__.py
    │   ├── [M] test_core/
    │   │   └── [M] __init__.py
    │   └── [M] test_agents/
    │       └── [M] __init__.py
    ├── [M] integration/                     # ← CREATED by Step 4
    │   └── [M] __init__.py
    └── [M] eval/                            # ← CREATED by Step 4
        └── [M] __init__.py
```

---

## Execution Protocol

Work through each step sequentially. After completing each step, run the listed
**verification commands** and confirm they pass before proceeding.
Report `CREATED` / `EXISTS (skipped)` for every item. Report PASS/FAIL for every check.

---

## Step 0 — Branch Setup

**Action:** Ensure all work happens on a dedicated feature branch, not directly on `develop` or `main`.

```bash
git branch --show-current

# If not already on feature/scaffold-setup:
git checkout develop
git pull origin develop 2>/dev/null || true
git checkout -b feature/scaffold-setup
```

**Verification — Step 0:**
```bash
current=$(git branch --show-current)
[ "$current" = "feature/scaffold-setup" ] && echo "PASS: on feature/scaffold-setup" || echo "FAIL: wrong branch — stop here"
git branch | grep -q "develop" && echo "PASS: develop exists" || echo "WARN: develop missing — run branching-strategy skill first"
```

---

## Step 1 — Pre-commit Configuration

**Action:** Create `.pre-commit-config.yaml` only if it does not already exist.

```bash
if [ -f ".pre-commit-config.yaml" ]; then
  echo "EXISTS (skipped): .pre-commit-config.yaml"
else
  cat > .pre-commit-config.yaml << 'YAML'
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
YAML
  echo "CREATED: .pre-commit-config.yaml"
fi
```

**Verification — Step 1:**
```bash
test -f .pre-commit-config.yaml && echo "PASS: file exists"
python -c "import yaml; yaml.safe_load(open('.pre-commit-config.yaml'))" && echo "PASS: valid YAML"
grep -q "ruff"           .pre-commit-config.yaml && echo "PASS: ruff hook present"
grep -q "black"          .pre-commit-config.yaml && echo "PASS: black hook present"
grep -q "bandit"         .pre-commit-config.yaml && echo "PASS: bandit hook present"
grep -q "detect-secrets" .pre-commit-config.yaml && echo "PASS: detect-secrets hook present"
```

---

## Step 2 — Run Script

**Action:** Create `scripts/` directory and `scripts/run_agent.py` if missing.

```bash
# Directory
if [ ! -d "scripts" ]; then
  mkdir scripts
  echo "CREATED dir: scripts/"
else
  echo "EXISTS (skipped): scripts/"
fi

# Script file
if [ -f "scripts/run_agent.py" ]; then
  echo "EXISTS (skipped): scripts/run_agent.py"
else
  cat > scripts/run_agent.py << 'PY'
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
PY
  echo "CREATED: scripts/run_agent.py"
fi
```

**Verification — Step 2:**
```bash
test -d scripts/                 && echo "PASS: scripts/ dir exists"
test -f scripts/run_agent.py     && echo "PASS: run_agent.py exists"
python -c "import ast; ast.parse(open('scripts/run_agent.py').read())" && echo "PASS: valid Python syntax"
grep -q "env_file" scripts/run_agent.py && echo "PASS: .env guard present"
```

---

## Step 3 — Package READMEs

Create each README only if it does not already exist. Check before writing.

### 3a — `src/agents/README.md`

```bash
if [ -f "src/agents/README.md" ]; then
  echo "EXISTS (skipped): src/agents/README.md"
else
  # Write file with content covering:
  # Purpose, full sub-directory structure tree, key dependencies (google-adk, src/core,
  # src/tools, src/skills, src/memory), how to add a new sub-agent (5-step checklist),
  # contribution guidelines (pure factory functions, prompts in files not strings, tests required)
  echo "CREATED: src/agents/README.md"
fi
```

Content must document:
- The supervisor → sub_agents delegation flow
- Every subdirectory: `supervisor/`, `supervisor/prompts/`, `sub_agents/`, and all four sub-agent packages with their files annotated
- How to add a new sub-agent (create dir → build_<name>_agent factory → SUB_AGENTS tuple → configs/agents/<name>.yaml → delegation_rules.md)

### 3b — `src/tools/README.md`

```bash
if [ -f "src/tools/README.md" ]; then
  echo "EXISTS (skipped): src/tools/README.md"
else
  # Write file covering:
  # Purpose, full file list (_base.py, registry.py, agent_tools.json, all tool files),
  # @tool decorator contract (idempotent, raise ToolError, docstring required),
  # how to add a tool (create file → @tool decorator → register in agent_tools.json → add to TOOLS tuple),
  # contribution guidelines
  echo "CREATED: src/tools/README.md"
fi
```

### 3c — `src/skills/README.md`

```bash
if [ -f "src/skills/README.md" ]; then
  echo "EXISTS (skipped): src/skills/README.md"
else
  # Write file covering:
  # Purpose, file list (toolset.py, catalog/), SKILL.md required sections
  # (Purpose, Inputs, Outputs, Rules, Examples), how build_skill_toolset() works,
  # how to add a skill (create catalog/<name>/SKILL.md → pass name to build_skill_toolset)
  echo "CREATED: src/skills/README.md"
fi
```

Current catalog entries to document:
- `catalog/campaign-performance/SKILL.md` — KPI formulas, pacing, anomaly detection
- `catalog/audience-insights/SKILL.md` — Audience segmentation

### 3d — `src/prompts/README.md`

```bash
if [ -f "src/prompts/README.md" ]; then
  echo "EXISTS (skipped): src/prompts/README.md"
else
  # Write file covering:
  # Purpose, file list (loader.py, personas/analyst.md, partials/output_format.md,
  # partials/safety.md), render_prompt() usage with Jinja2 examples,
  # where agent-specific prompts live (src/agents/<agent>/prompts/), convention that
  # no prompt strings may be hardcoded in Python files
  echo "CREATED: src/prompts/README.md"
fi
```

### 3e — `tests/README.md`

```bash
if [ -f "tests/README.md" ]; then
  echo "EXISTS (skipped): tests/README.md"
else
  # Write file covering:
  # Directory layout (unit/, integration/, eval/), how to run each suite,
  # pytest markers (@pytest.mark.unit, @pytest.mark.integration, @pytest.mark.eval),
  # 80% coverage gate, short example unit test for a tool
  echo "CREATED: tests/README.md"
fi
```

**Verification — Step 3:**
```bash
for f in src/agents/README.md src/tools/README.md src/skills/README.md src/prompts/README.md tests/README.md; do
  if [ -f "$f" ]; then
    lines=$(wc -l < "$f")
    [ "$lines" -ge 20 ] && echo "PASS: $f ($lines lines)" || echo "WARN: $f exists but too short ($lines lines)"
  else
    echo "FAIL: $f missing"
  fi
done
```

---

## Step 4 — Test Directory Structure

**Action:** Create missing test subdirectories and `__init__.py` stubs. Never touch existing files.

```bash
# Helper: create dir + __init__.py only if missing
create_test_pkg() {
  dir=$1
  if [ ! -d "$dir" ]; then
    mkdir -p "$dir"
    echo "CREATED dir: $dir"
  else
    echo "EXISTS (skipped): $dir"
  fi
  init="$dir/__init__.py"
  if [ ! -f "$init" ]; then
    touch "$init"
    echo "CREATED: $init"
  else
    echo "EXISTS (skipped): $init"
  fi
}

create_test_pkg "tests/unit"
create_test_pkg "tests/unit/test_tools"
create_test_pkg "tests/unit/test_core"
create_test_pkg "tests/unit/test_agents"
create_test_pkg "tests/integration"
create_test_pkg "tests/eval"
```

**Verification — Step 4:**
```bash
for d in tests/unit tests/unit/test_tools tests/unit/test_core tests/unit/test_agents tests/integration tests/eval; do
  test -d "$d" && echo "PASS dir: $d" || echo "FAIL dir: $d"
  test -f "$d/__init__.py" && echo "PASS __init__: $d/__init__.py" || echo "FAIL __init__: $d/__init__.py"
done
```

---

## Step 5 — Update Root README and Annotate `.env.example`

### 5a — Root README quickstart

**Check:** Does `README.md` already contain "pre-commit install" and "scripts/run_agent.py"?

```bash
grep -q "pre-commit install"  README.md && echo "EXISTS: quickstart already updated" || echo "NEEDS UPDATE: README.md"
grep -q "scripts/run_agent.py" README.md && echo "EXISTS: run script referenced"    || echo "NEEDS UPDATE: run script ref"
```

If either check fails, update (do not replace) the Quickstart section to include:
1. clone → venv → pip install → cp .env.example → pre-commit install → python scripts/run_agent.py
2. A project structure table covering all top-level directories

### 5b — `.env.example` annotation

**Check:** Do all variables have inline comments?

```bash
python - << 'EOF'
lines = open('.env.example').readlines()
uncommented = [l.strip() for l in lines
               if l.strip() and not l.startswith('#') and '#' not in l]
if uncommented:
    print(f"NEEDS UPDATE: {len(uncommented)} vars lack comments: {uncommented}")
else:
    print("PASS: all variables have inline comments")
EOF
```

If any variables lack comments, add inline `# <description>` to each affected line.
Known missing: `APP_ENV`, `LOG_LEVEL`, `GOOGLE_GENAI_USE_VERTEXAI`, `GOOGLE_API_KEY`,
`ANTHROPIC_API_KEY`, `GCS_ARTIFACT_BUCKET`, `GA4_PROPERTY_ID`, all `SNOWFLAKE__*` vars.

---

## Step 6 — Commit, Push, and Open PR

**Action:** After all verifications pass, present a final status table and suggest (but do not execute) the commit.

### 6a — Final scan

```bash
echo "=== Git status ==="
git status --short

echo "=== Branch ==="
git branch --show-current
```

### 6b — Status table

| Step | Item | Status |
|------|------|--------|
| 0 | Feature branch `feature/scaffold-setup` | |
| 1 | `.pre-commit-config.yaml` | |
| 2 | `scripts/run_agent.py` | |
| 3a | `src/agents/README.md` | |
| 3b | `src/tools/README.md` | |
| 3c | `src/skills/README.md` | |
| 3d | `src/prompts/README.md` | |
| 3e | `tests/README.md` | |
| 4 | `tests/unit/`, `tests/integration/`, `tests/eval/` + `__init__.py` stubs | |
| 5 | Root `README.md` and `.env.example` updated | |

Fill in ✓ CREATED, ⊘ EXISTED (skipped), or ✗ FAILED for each row.

### 6c — Suggested commands (confirm with user before running)

```bash
git add \
  .pre-commit-config.yaml \
  scripts/run_agent.py \
  README.md .env.example \
  src/agents/README.md \
  src/tools/README.md \
  src/skills/README.md \
  src/prompts/README.md \
  tests/README.md \
  tests/unit/ tests/integration/ tests/eval/

git commit -m "feat(scaffold): add pre-commit hooks, package READMEs, test dirs, and 5-min onboarding"
git push -u origin feature/scaffold-setup
```

Open PR: **`feature/scaffold-setup` → `develop`** (not main).

---

## Re-run Behaviour

Running this skill again on a fully-scaffolded repo will print:
```
EXISTS (skipped): .pre-commit-config.yaml
EXISTS (skipped): scripts/run_agent.py
EXISTS (skipped): src/agents/README.md
... (all items)
```
Nothing is overwritten. The skill is fully safe to re-run.
