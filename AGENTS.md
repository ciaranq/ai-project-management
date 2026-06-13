# AGENTS.md — AI Agent Instructions for ciaranq Projects

This file is read by any AI model (Claude, Gemini, Copilot, etc.) working on repositories under the `ciaranq` GitHub account. It establishes shared conventions so models can work together without stepping on each other.

---

## The methodology

All projects use the **ai-pm 7-stage workflow**. Before doing anything, check what stage the project is in:

```bash
gh issue list --state all --json number,title,labels,state \
  --jq '.[] | "\(.state) \(.labels | map(.name) | join(",")) \(.title)"' | grep "stage:" | sort
```

**Never advance a stage until all its GitHub Issues are closed.** Issues encode the decisions that gate progression. Closing an issue without documenting the decision in a comment doesn't count.

| Stage | Name | What unlocks the next stage |
|-------|------|-----------------------------|
| 0 | Setup | Setup checklist issue closed |
| 1 | Problem Discovery | Problem, user needs, business goals all closed |
| 2 | AI Feasibility & Data | Feasibility, solution, data strategy all closed |
| 3 | Model Development | Model spec, success metrics, architecture all closed |
| 4 | Solution Design | UX, integration, feedback, prod-readiness all closed |
| 5 | Testing & Deployment | Testing, quality gates, monitoring all closed |
| 6 | Iteration | Feedback analysis, next scope, improvement plan all closed |

---

## Before starting any work

1. **Read the handoff file** (if it exists): `.ai-session/handoff.md`
2. **Read the model-specific context**: `GEMINI.md` (Gemini), `.claude/CLAUDE.md` (Claude), `AGENTS.md` (all)
3. **Check open issues**: `gh issue list --state open`
4. **Check current branch**: `git branch --show-current`
5. **Don't re-do work** already reflected in closed issues or recent commits

If no handoff file exists and no issues exist, run `/ai-pm:setup` to initialise the project into the workflow.

---

## Shared conventions all models must follow

### Commits
- Use `feat:`, `fix:`, `chore:`, `docs:` prefixes
- Use `session:` prefix for handoff commits: `session: handoff to Gemini — auth WIP`
- Reference issue numbers: `fix: validate email input (closes #12)`

### Branches
- Don't create model-named branches (`claude/...`, `gemini/...`) — work passes between models on the same branch
- Branch names describe the work, not who's doing it

### Issues
- Don't close issues without a decision comment
- Don't create new issues for work already tracked
- Label every issue with its stage label before closing

### Handoff state
- `.ai-session/handoff.md` is the shared state file — always committed to git
- Never gitignore `.ai-session/`
- Update it before switching models; read it before starting

### Code quality (Stages 3–6)
- **Think before coding**: state assumptions, surface ambiguity before implementing
- **Simplicity first**: minimum code that solves the problem; no speculative features
- **Surgical changes**: only touch what the current stage requires
- **Goal-driven**: frame tasks as verifiable goals with a brief plan before multi-step work

---

## Model roles on ciaranq projects

### Claude (Claude Code, VSCode extension, Claude Pro)
Best for: architecture decisions, code review, security analysis, writing specs/docs/plans, debugging non-obvious issues, ai-pm stage gating.

### Gemini (Gemini CLI / Hermes, AI Pro, Gemini 2.5 Pro)
Best for: large codebase analysis (1M token context), systematic refactoring across many files, test generation at scale, bulk documentation, well-defined tasks needing large context.

### Collaboration command
When switching models, use `/ai-pm:collab handoff` (creates structured `.ai-session/handoff.md`) and `/ai-pm:collab pickup` (reads handoff and briefs the receiving model).

---

## Installing the ai-pm skill

The skill lives at https://github.com/ciaranq/ai-project-management

**Claude Code:**
```bash
mkdir -p ~/.claude/plugins/cache/local/ai-pm/1.0.0/skills
cp -r skills/ai-pm skills/setup skills/stage skills/collab ~/.claude/plugins/cache/local/ai-pm/1.0.0/skills/
# Register in ~/.claude/plugins/installed_plugins.json and ~/.claude/settings.json
# See README.md for full instructions
```

**Gemini CLI (Hermes):**
```bash
# Skills
for skill in ai-pm setup stage collab; do
  mkdir -p ~/.gemini/skills/$skill/references 2>/dev/null
  for f in $(find skills/$skill -name "*.md"); do
    dest=~/.gemini/skills/$skill/${f#skills/$skill/}
    mkdir -p $(dirname $dest)
    sed 's/\.claude\//\.gemini\//g; s/CLAUDE\.md/GEMINI.md/g' $f > $dest
  done
done

# Commands
mkdir -p ~/.gemini/commands/ai-pm
cp gemini/commands/ai-pm/*.toml ~/.gemini/commands/ai-pm/

# Agent identity
mkdir -p ~/.gemini/agents
cp gemini/agents/hermes.md ~/.gemini/agents/hermes.md

# Global context (brief for every Gemini session)
cp GEMINI.md ~/.gemini/GEMINI.md

# Set model in settings (preserves existing settings)
python3 -c "
import json, os
p = os.path.expanduser('~/.gemini/settings.json')
s = json.load(open(p)) if os.path.exists(p) else {}
s['model'] = 'gemini-2.5-pro'
s['contextFileName'] = 'GEMINI.md'
json.dump(s, open(p, 'w'), indent=2)
print('Settings updated')
"
```

---

## Project-specific context

Each project has its own `AGENTS.md` (this file, copied or linked), plus:
- `GEMINI.md` — Gemini-specific session context
- `.claude/CLAUDE.md` — Claude-specific session context
- `.ai-session/handoff.md` — current handoff state (if a session was paused)
- `RUNBOOK.md` — full usage guide for the ai-pm workflow (in this repo)

When working on a project that doesn't yet have these files, start with `/ai-pm:setup` to scaffold everything.
