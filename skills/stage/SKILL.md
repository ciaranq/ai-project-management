---
name: stage
description: Smart stage navigator for the AI PM workflow. Detects the current stage, shows progress, advances the project, or jumps to a specific stage. Invoke with /ai-pm:stage for a status check, /ai-pm:stage next to advance, /ai-pm:stage <N> to target a stage, /ai-pm:stage audit to onboard an existing project into the workflow, or /ai-pm:stage status for a full summary.
argument-hint: [next | status | audit | 1-6]
allowed-tools: [Bash, Read, Write, Edit]
---

# AI PM: Stage Navigator

You are the stage navigator for the AI PM workflow. Parse `$ARGUMENTS` to determine mode, then execute the appropriate section below.

| Arguments | Mode |
|-----------|------|
| _(empty)_ | **Status** — show current position and what's needed |
| `status` | **Status** — same as empty |
| `next` | **Advance** — gate-check then move to next stage |
| `1` through `6` | **Jump** — go to a specific stage |
| `audit` | **Audit** — onboard an existing project |

---

## Step 0: Always run first — read current state

Before doing anything else, collect the full picture:

```bash
# All issues with stage labels
gh issue list --state all --json number,title,labels,state,milestone \
  --jq '.[] | {
    number: .number,
    title: .title,
    state: .state,
    stage: (.labels | map(select(.name | startswith("stage:"))) | .[0].name // "none"),
    milestone: .milestone.title
  }' | python3 -c "
import json, sys
issues = [json.loads(l) for l in sys.stdin]
from collections import defaultdict
stages = defaultdict(lambda: {'open': [], 'closed': []})
for i in issues:
    stages[i['stage']][i['state']].append(i)
for stage in sorted(stages.keys()):
    o = len(stages[stage]['open'])
    c = len(stages[stage]['closed'])
    print(f'{stage}: {c} closed, {o} open')
"
```

From this output, determine:
- **Completed stages**: all issues closed
- **Active stage**: has open issues
- **Not started**: no issues exist yet
- **Current stage number**: the active or next-to-start stage

If there are NO issues at all and Stage 0 hasn't been run, tell the user: "No stages found. Run `/ai-pm:setup` first to initialise the project."

---

## Mode: Status (empty args or `status`)

Show a clear, scannable status table:

```
AI PM Project Status
====================

Stage 0: Setup              ✓ complete
Stage 1: Problem Discovery  ⚡ ACTIVE — 2 open, 1 closed
Stage 2: AI Feasibility     ○ not started
Stage 3: Model Development  ○ not started
Stage 4: Solution Design    ○ not started
Stage 5: Testing & Deploy   ○ not started
Stage 6: Iteration          ○ not started

Current stage: Stage 1 — Problem Discovery
Blocking issues:
  #2  [Stage 1] Define the problem we're solving
  #3  [Stage 1] Document the biggest user problems

To advance: close the blocking issues, then run /ai-pm:stage next
```

Use these symbols:
- `✓ complete` — all issues closed
- `⚡ ACTIVE` — has open issues (this is where work happens)
- `○ not started` — no issues created yet
- `⚠ partial` — has both open and closed issues (use instead of ACTIVE when most are closed)

After the table, list the open issues for the current stage with their numbers and titles so the user can see exactly what's blocking.

---

## Mode: Advance (`next`)

### Gate check

```bash
CURRENT_STAGE_LABEL="stage:N-name"  # replace with actual current stage label
gh issue list --label "$CURRENT_STAGE_LABEL" --state open --json number,title
```

If **any issues are open**, do NOT advance. Show:
```
Cannot advance — Stage N has open issues:
  #X  [Stage N] Issue title
  #Y  [Stage N] Issue title

Close these issues (with decision comments) before advancing.
```

If all issues are closed, proceed.

### Advance to next stage

1. Print a brief summary of what was decided in the completed stage (pull from closed issue titles and any `Decision:` comments)
2. Determine the next stage number
3. Create all issues for the next stage using the templates in `references/stages.md`

When creating issues, use:
```bash
gh issue create \
  --title "[Stage N] <title from template>" \
  --label "stage:N-name" \
  --milestone "Stage N: Name" \
  --body "<body from template>"
```

Create all issues for the new stage before reporting back. Then show the user:
```
✓ Stage N complete — <one-sentence summary of decisions made>

Stage N+1: <Name> is now active.
Created N issues:
  #X  [Stage N+1] First issue
  #Y  [Stage N+1] Second issue

Work through these issues, document decisions in the comments, then run
/ai-pm:stage next when all are closed.
```

---

## Mode: Jump to stage (`1` through `6`)

Parse the argument as an integer N (1–6).

### Jumping forward (N > current stage)

If skipping stages:
```
You're jumping from Stage <current> to Stage <N>, skipping Stage(s) <list>.

Skipped stages will be marked as retroactively incomplete. This is fine
for existing projects being onboarded, but for new projects it means
those questions were never answered.

Continue? (yes/no)
```

Wait for confirmation. If yes:
- For each skipped stage, create a single retroactive issue with the title `[Stage X] SKIPPED — retroactive documentation required` and label `stage:X-name, blocker`
- Then create the full issue set for stage N

### Jumping to current or lower stage

If jumping to an already-active stage: show the status for that stage (same as Status mode but filtered).

If jumping backward: warn:
```
Stage <N> was already completed. Jumping back means reopening decisions.
This happens when business goals change or earlier assumptions were wrong.

The existing Stage N issues will remain closed. New issues will be created
for Stage N to capture what changed. Label them with 'decision' so they
stand out in the history.
```

Then create the new issues for stage N with titles prefixed `[Stage N - Revision]`.

---

## Mode: Audit (`audit`)

Run the existing-project onboarding flow. This is a thorough process — take your time.

### Step 1: Gather evidence

```bash
# Git history context
git log --oneline -20

# Existing docs
find . -name "*.md" -not -path "*/node_modules/*" -not -path "*/.git/*" | head -20
find . -name "*.txt" -name "*spec*" -o -name "*brief*" -o -name "*PRD*" 2>/dev/null

# Existing issues
gh issue list --state all --json number,title,labels,state,body | head -50

# README
cat README.md 2>/dev/null | head -80

# Check for existing .claude/CLAUDE.md
cat .claude/CLAUDE.md 2>/dev/null
```

Read all found documents to understand what has been built and decided.

### Step 2: Map to stages

For each stage 1–6, assess its status based on what you found:

| Status | Criteria |
|--------|----------|
| `complete` | All key questions answered, evidence found |
| `partial` | Some questions answered, some not |
| `missing` | No evidence this stage was done |
| `n/a` | Genuinely doesn't apply (e.g., pure rules engine needs no AI feasibility) |

Present the map to the user for confirmation before creating issues:
```
Stage Audit Results
===================
Stage 1: Problem Discovery  → complete   (found in README.md: "This tool solves...")
Stage 2: AI Feasibility     → partial    (AI choice documented, data strategy missing)
Stage 3: Model Development  → missing    (no model spec found)
Stage 4: Solution Design    → partial    (UI designed, no feedback loop documented)
Stage 5: Testing & Deploy   → complete   (CI pipeline exists, launch notes in CHANGELOG)
Stage 6: Iteration          → active     (v1 shipped, v2 planning underway)

Proposed entry point: Stage 2 (first stage with open gaps)

Does this look right? Any corrections before I create the issues?
```

Wait for user confirmation or corrections.

### Step 3: Create retroactive issues

For each stage up to the entry point:

**`complete` stages** — create all issues pre-closed:
```bash
gh issue create --title "[Stage N] <title>" --label "stage:N-name" --milestone "Stage N: Name" \
  --body "<template body>"
gh issue close <number> --comment "Retroactive: Evidence found in <source>. <brief answer to the question>"
```

**`partial` stages** — create issues, close answered ones, leave unanswered open:
```bash
# For each answered question: create + close with evidence
# For each unanswered question: create and leave open
```

**`missing` stages** — create all issues open, add `blocker` label:
```bash
gh issue create --title "[Stage N] <title>" --label "stage:N-name,blocker" --milestone "Stage N: Name" \
  --body "<template body>

⚠ This stage was not completed before the project launched. These decisions
need to be documented retroactively or addressed in the current iteration."
```

**`n/a` stages** — create a single issue explaining why it's not applicable, then close it.

### Step 4: Report entry point

After all issues are created:
```
Audit complete. Retroactive issues created for Stages 1–5.

Entry point: Stage 2 — AI Feasibility
Open issues requiring attention:
  #X  [Stage 2] Data strategy (never documented)
  #Y  [Stage 4] Feedback collection mechanism (not designed)
  #Z  [Stage 3] Model success metrics (missing)

Run /ai-pm:stage status at any time to see the full picture.
Run /ai-pm:stage next when the current stage's issues are closed.
```

Also check whether Stage 0 setup was done (labels, milestones, folder structure). If not, prompt: "Stage 0 infrastructure is missing. Run `/ai-pm:setup --existing` to add it without overwriting existing files."
