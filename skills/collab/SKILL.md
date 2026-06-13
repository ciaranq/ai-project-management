---
name: collab
description: Multi-model collaboration skill for Claude and Gemini working together on the same project in VSCode. Use when handing off work to the other model, picking up work the other model left, deciding which model should handle a task, or checking the current collaboration state. Invoke with /ai-pm:collab handoff, /ai-pm:collab pickup, /ai-pm:collab divide, or /ai-pm:collab status. Trigger automatically whenever the user mentions switching to Gemini, picking up from Claude, or asks who should handle a task.
argument-hint: [handoff | pickup | divide <task> | status]
allowed-tools: [Bash, Read, Write, Edit]
---

# AI Collaboration: Claude ↔ Gemini

This skill manages handoffs between Claude (Claude Code in VSCode) and Gemini (Gemini CLI / Hermes). Both run on the same machine, share the same filesystem and git repo, and work sequentially on the same project.

The single source of shared state is `.ai-session/handoff.md`. Both models always write to it when handing off, always read it when picking up.

---

## Step 0: Always — identify current model

First, determine which model is running this skill. Check which context file was loaded:
```bash
# Claude Code sessions have this env var or IDE context
echo ${CLAUDE_CODE_VERSION:-"not-claude"} 
# Or check for the IDE integration
```

In practice: if you are Claude reading this, you are **Claude**. The other model is **Gemini**. 
If you are Gemini reading this, you are **Gemini**. The other model is **Claude**.

Use this throughout — "I am handing to Gemini" or "I am handing to Claude" — never say "the other model".

---

## Mode: `handoff`

You are done with a chunk of work and want to hand off to the other model. Create a rich, specific handoff document so nothing is lost in translation.

### Step 1 — Collect current state
```bash
# What changed
git diff --stat HEAD~3..HEAD 2>/dev/null || git diff --stat
git log --oneline -5

# What's currently open / in progress
gh issue list --state open --json number,title,labels,assignees --jq '.[] | "#\(.number) \(.title)"'

# Any uncommitted work
git status --short
git stash list
```

### Step 2 — Write the handoff document

Create or overwrite `.ai-session/handoff.md`:

```markdown
# Handoff — [current model] → [receiving model]
**Date**: [today's date]
**Branch**: [current branch]
**Project stage**: Stage [N] — [stage name]

## What was accomplished this session
<!-- Be specific: what decisions were made, what was built, what was resolved -->
- [item] (commit: [sha])
- [item] (issue: #N closed)

## Where work was left
<!-- Be precise — file name, function name, line number, test name -->
**Currently in progress**: [exact description]
**File**: [path]
**State**: [what's done in this file, what's not]

## What to do next
<!-- Ordered, specific, with success criteria -->
1. [ ] [task] — done when [criterion]
2. [ ] [task] — done when [criterion]
3. [ ] [task] — done when [criterion]

## Open questions
<!-- Things that need a decision or more information -->
- [question] — [context needed to answer it]

## Things I discovered (read these first)
<!-- Gotchas, constraints, surprising behaviour found during the session -->
- [discovery]: [implication]

## Key files to read
<!-- The minimum set to understand the current state -->
- [path]: [why it matters]

## For [receiving model specifically]
<!-- Tailored instructions based on what the receiving model is good at -->
[model-specific guidance]
```

Fill every section. Vague handoffs cause re-work. If "where work was left" is imprecise, the next model will spend the first 10 minutes figuring out where to start.

### Step 3 — Commit the handoff
```bash
mkdir -p .ai-session
git add .ai-session/handoff.md
git diff --staged --quiet || git commit -m "session: handoff to [receiving model] — [one-line summary of state]"
git push
```

### Step 4 — Update the receiving model's context file

**If handing to Gemini**, append to `GEMINI.md` (at project root):
```markdown
## Current handoff (updated [date])
See `.ai-session/handoff.md` for full context.
Next task: [one-liner]
Branch: [branch]
```

**If handing to Claude**, append to `.claude/CLAUDE.md`:
```markdown
## Current handoff (updated [date])
See `.ai-session/handoff.md` for full context.
Next task: [one-liner]
Branch: [branch]
```

### Step 5 — Report to user
Tell the user:
- What was committed
- Exactly what to tell the other model to start: e.g., "Tell Gemini: `/ai-pm:collab pickup`"
- Any flags or gotchas to verbally mention before switching

---

## Mode: `pickup`

You are starting work and need to understand what the other model left you. Read the handoff and brief yourself efficiently.

### Step 1 — Read all context sources
```bash
# Primary: the handoff document
cat .ai-session/handoff.md 2>/dev/null || echo "No handoff found — reading git log instead"

# Recent commits (last 10)
git log --oneline -10

# Current branch and status
git status --short
git branch --show-current

# Open issues for current stage
gh issue list --state open --json number,title,labels \
  --jq '.[] | "#\(.number) \(.title) [\(.labels | map(.name) | join(", "))]"'
```

### Step 2 — Reconstruct understanding

From what you read, form a clear picture of:
- What stage the project is in and what gate it's working toward
- What was just completed (so you don't redo it)
- Where exactly the current work is (file, function, test)
- What the very next action is

If `.ai-session/handoff.md` doesn't exist: read the last 10 commits, the current open issues, and the context file (`GEMINI.md` or `.claude/CLAUDE.md`). Construct a best-effort understanding.

### Step 3 — Brief the user and confirm

Present a concise summary:
```
Picking up from [other model].

Last session: [one-sentence summary of what was done]
In progress: [exactly what was being worked on]
Next action: [the specific next thing to do]

Open questions from the handoff:
  - [question]

Ready to continue? Any corrections to my understanding?
```

Wait for confirmation or corrections before starting work.

### Step 4 — Update handoff on pickup
Mark the handoff as active to prevent confusion:
```bash
# Add a pickup timestamp to the handoff
echo "\n---\n**Picked up by [current model]** at [date]" >> .ai-session/handoff.md
```

---

## Mode: `divide <task>`

Given a task description, recommend how to split it between Claude and Gemini. Parse the task from `$ARGUMENTS` after "divide".

Consult `references/model-strengths.md` for the full guide. Summary:

**Route to Claude when the task needs:**
- Judgment about ambiguous requirements or architectural trade-offs
- Multi-step reasoning where each step depends on the last
- Security review, code review, or quality critique
- Writing documentation, plans, specs, or decision records
- Debugging where the cause is non-obvious
- Any ai-pm stage gating decision

**Route to Gemini when the task needs:**
- Reading or refactoring across a large number of files (Gemini has a 1M token context window)
- Generating comprehensive test suites for existing code
- Summarising or extracting from large documents or log files
- Mechanical, well-defined tasks that just need to touch many files
- Searching the entire codebase for patterns or usages

**For tasks that span both:**
Split into phases. Example: "Refactor auth module":
- Claude: design the new interface and write the migration plan (judgment + architecture)
- Gemini: execute the mechanical file changes using the plan (large context + systematic)
- Claude: review the diff and close the stage issue (judgment + quality gate)

Present the recommendation as:
```
Task: [task description]

Recommended split:
  Claude:  [specific sub-tasks]
  Gemini:  [specific sub-tasks]
  Order:   [who goes first and why]

Rationale: [one sentence explaining the key deciding factor]
```

---

## Mode: `status`

Show the current collaboration state at a glance.

```bash
# Handoff state
echo "=== Handoff ===" && cat .ai-session/handoff.md 2>/dev/null | head -20 || echo "No active handoff"

# Recent session history
echo "=== Last 5 commits ===" && git log --oneline -5

# Current open work
echo "=== Open issues ===" && gh issue list --state open --json number,title,labels \
  --jq '.[] | "#\(.number) \(.title)"' | head -10

# Branch
echo "=== Branch ===" && git branch --show-current
```

Present as a clean summary:
```
Project: [name]  |  Stage: N — [stage name]  |  Branch: [branch]

Last handoff: [date] — [from model] handed to [to model]
Summary: [one-liner from handoff]

Currently open: [N] issues in Stage [N]
Last commit: [sha] [message]

No active handoff — [current model] is the active model.
```

---

## Shared conventions (both models must follow)

**`.ai-session/` folder** — committed to git, never gitignored. Both models write here.

**Commit message prefix** — use `session:` for handoff commits so they're easy to find:
```
session: handoff to Gemini — auth refactor 60% done, needs test coverage
session: pickup by Claude — continuing from Gemini's refactor
```

**Branch ownership** — use descriptive branch names, not model names. Work passes between models on the same branch. Don't create a new branch just because you're a different model.

**Issue assignment** — don't change issue assignments during a session. The stage gate owns the issue, not the model working on it.

**Never re-read what's already in the handoff** — the handoff is authoritative. If it says "issue #5 is closed", trust it. Don't re-check every claim — use the time to do the work.
