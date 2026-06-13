---
name: ai-pm
description: AI Product Manager methodology for structured AI project development across 7 stages. Use this skill whenever: starting a new AI project or feature, asking what stage or phase a project is in, ready to advance to the next stage, checking whether a stage is complete, creating GitHub Issues for project planning, working through AI feasibility decisions, designing a model or solution, planning testing and deployment, or iterating on AI product quality. Invoke proactively for any AI project management work — even if the user doesn't say "PM" or "stages" explicitly. If someone is building something with AI and wants to manage it rigorously, this skill applies.
---

# AI Product Manager Skill

A rigorous 7-stage methodology for building AI products, grounded in the AI PM framework. GitHub Issues are the single source of truth — they capture decisions, document answers, and gate progression between stages.

## Core Rule

**Never advance to the next stage until all GitHub Issues for the current stage are closed.** Closing an issue means the question is answered, the decision is documented, and it's been agreed upon — not just acknowledged.

## The 7 Stages

| Stage | Name | PM Phase | Key Outcome |
|-------|------|----------|-------------|
| 0 | Project Setup | Foundation | Repo, structure, labels, milestones |
| 1 | Problem Discovery | Phase 1 | Problem, user needs, business goals documented |
| 2 | AI Feasibility & Data Strategy | Phase 1 | AI validated as right approach, data plan confirmed |
| 3 | Model Development | Phase 2 | Model spec and success metrics defined |
| 4 | Solution Design & Integration | Phase 2 | UX, integration, feedback loop, uncertainty handling designed |
| 5 | Testing, Deployment & Monitoring | Phase 3 | Launch quality gates met, monitoring live |
| 6 | Iteration & Improvement | Phase 4 | Learnings captured, next iteration planned |

---

## How to Use This Skill

### Mode A — New Project
Follow Stage 0 → Stage 6 in sequence. Create issues at the start of each stage, close them before advancing.

### Mode B — Onboarding an Existing Project
Use this when a project already exists but wasn't started with this methodology. The goal is to identify what was done rigorously, what was skipped, and enter the workflow at the correct stage with all gaps documented.

**Step 1 — Audit**
Scan the project to understand its current state:
```bash
# Read existing docs, README, any specs or decision records
ls docs/ README* *.md
# Review git history for context on what was built and why
git log --oneline -30
# Check if GitHub Issues already exist
gh issue list --state all --json number,title,labels
```

Also read: existing CLAUDE.md, any ADRs (Architecture Decision Records), design docs, or PRDs.

**Step 2 — Stage Mapping**
For each stage (1–6), determine its status based on what you found:

| Status | Meaning | Action |
|--------|---------|--------|
| `complete` | All key questions answered and evidenced | Create the issues pre-closed with answers in comments |
| `partial` | Some questions answered, some not | Create issues, close answered ones, leave unanswered open |
| `missing` | No evidence this stage was done | Create issues open, flag as `blocker` |
| `n/a` | Not applicable to this project type | Create a single issue documenting why, then close it |

**Step 3 — Retroactive Issue Creation**
Create issues for every stage up to the current one. For stages that were effectively completed (the work was done, just not documented), create and immediately close the issue with a comment explaining where the evidence was found:
```bash
gh issue create --title "[Stage 1] Define the problem we're solving" \
  --label "stage:1-problem-discovery" \
  --body "Retroactively documenting..."
gh issue close <number> --comment "Retroactive: Found in README.md - Problem statement: ..."
```

**Step 4 — Determine Entry Stage**
Based on the audit, identify the first stage with open issues. That becomes the active stage. Common patterns:
- Project shipped v1: probably entering Stage 6
- Project in active development: likely Stage 4 or 5
- Project in design/planning: Stage 2 or 3
- Just an idea: Stage 1

**Step 5 — Fill CLAUDE.md**
After the audit, populate `.claude/CLAUDE.md` using the template at the bottom of this skill.

---

### Detecting the Current Stage

When invoked on an existing project:
```bash
# List all stage labels to see what's been created
gh issue list --state all --label "stage:0-setup" --json number,title,state
# Repeat for stages 1-6, or scan all at once:
gh issue list --state all --json number,title,labels,state | python3 -c "
import json,sys
issues = json.load(sys.stdin)
for i in issues:
    labels = [l['name'] for l in i['labels']]
    stage = next((l for l in labels if l.startswith('stage:')), 'none')
    print(f\"{i['state']:6} | {stage:35} | {i['title']}\")
" | sort -k3
```

From the output: the highest stage with all issues **closed** = recently completed. The lowest stage with any **open** issues = current active stage. No issues at all = start at Stage 0.

### Checking Stage Completion

Before advancing to stage N+1:
```bash
gh issue list --label "stage:N-name" --state open
```
If anything is open, surface those issues to the user and help resolve them before proceeding.

### Advancing to the Next Stage

Once all issues in stage N are closed:
1. Summarise what was decided (pull from issue bodies/comments)
2. Confirm with the user: "Stage N is complete. Ready to start Stage N+1?"
3. Create all issues for stage N+1 using the templates in `references/stages.md`

---

## Stage 0: Project Setup

**Goal**: Establish foundations so all future work is consistent and traceable.

### Checklist
- [ ] Git repo initialised (`git init` or confirm exists)
- [ ] GitHub remote exists (`gh repo create` if needed)
- [ ] Label taxonomy created (see below)
- [ ] Milestones created for Stages 1–6
- [ ] Folder structure created
- [ ] Setup issue created and closed

### Folder Structure
```
project/
├── .claude/
│   └── CLAUDE.md           (project brief for Claude — fill in after Stage 1)
├── docs/
│   ├── problem-discovery/  (Stage 1 & 2 outputs)
│   ├── solution-design/    (Stage 3 & 4 outputs)
│   ├── testing/            (Stage 5 outputs)
│   └── iterations/         (Stage 6 outputs, one file per iteration)
├── src/                    (implementation — populated in Stage 4+)
└── README.md
```

### Label Taxonomy
```bash
gh label create "stage:0-setup"              --color "6e6e6e" --description "Project foundation"
gh label create "stage:1-problem-discovery"  --color "0075ca" --description "Phase 1: Problem"
gh label create "stage:2-ai-feasibility"     --color "0052cc" --description "Phase 1: Feasibility"
gh label create "stage:3-model-dev"          --color "e4e669" --description "Phase 2: Model"
gh label create "stage:4-solution-design"    --color "d93f0b" --description "Phase 2: Design"
gh label create "stage:5-testing-deploy"     --color "0e8a16" --description "Phase 3: Launch"
gh label create "stage:6-iteration"          --color "5319e7" --description "Phase 4: Improve"
gh label create "blocker"                    --color "ee0701" --description "Blocking stage advance"
gh label create "decision"                   --color "fef2c0" --description "Key decision documented"
```

### Milestones
```bash
gh api repos/{owner}/{repo}/milestones --method POST -f title="Stage 1: Problem Discovery"     -f description="Phase 1 - Problem Discovery"
gh api repos/{owner}/{repo}/milestones --method POST -f title="Stage 2: AI Feasibility"        -f description="Phase 1 - Feasibility & Data"
gh api repos/{owner}/{repo}/milestones --method POST -f title="Stage 3: Model Development"     -f description="Phase 2 - Model Spec"
gh api repos/{owner}/{repo}/milestones --method POST -f title="Stage 4: Solution Design"       -f description="Phase 2 - Design & Integration"
gh api repos/{owner}/{repo}/milestones --method POST -f title="Stage 5: Testing & Deployment"  -f description="Phase 3 - Launch"
gh api repos/{owner}/{repo}/milestones --method POST -f title="Stage 6: Iteration"             -f description="Phase 4 - Improve"
```

Stage 0 is complete when the setup checklist issue is created and closed.

---

## Stages 1–6: Issue-Driven Progression

Each stage creates a fixed set of GitHub Issues. The issues encode the key questions that must be answered before the project can move forward. When all issues for a stage are closed with documented answers, the stage is complete.

See `references/stages.md` for:
- The exact issues to create for each stage (title, body template, label, milestone)
- The key questions that each issue must answer before it can be closed
- Deliverable files to save in `docs/`

---

## Workflow Architecture Checklist

Before moving from Stage 4 → Stage 5 (entering production), validate against the production-readiness framework:

| Principle | Check |
|-----------|-------|
| **Clear Inputs** | Trigger is defined. Required context/data is specified. |
| **Reliable Context** | Right docs, history, and business rules are available to the model. |
| **Action Boundaries** | Explicit list of tools and systems the AI can touch. |
| **Validation** | Output validation layer defined. Who/what checks before action. |
| **Human Oversight** | Human-in-the-loop approval points identified for high-risk actions. |
| **Continuous Improvement** | Logging, feedback collection, and iteration loop planned. |

Create a Stage 4 issue titled "Production Readiness Checklist" that must have all six rows checked before it can be closed.

---

## GitHub Workflow Patterns

```bash
# Create a stage issue (use issue template from references/stages.md for the body)
gh issue create \
  --title "[Stage N] Issue title" \
  --label "stage:N-name" \
  --milestone "Stage N: Name" \
  --body "$(cat issue-body.md)"

# Close an issue with a decision comment
gh issue close <number> \
  --comment "**Decision**: <what was decided>\n\n**Rationale**: <why>\n\n**Evidence**: <links, data, discussion>"

# Check what's blocking stage advance
gh issue list --label "stage:N-name" --state open --json number,title,assignees

# View all issues across all stages
gh issue list --state all --json number,title,labels,state,milestone
```

---

## Skills & Tools Integration

This skill is designed to work alongside the compound-engineering plugin and native superpowers. Use the right skill for the right stage:

| Stage | Recommended Skills |
|-------|--------------------|
| 0: Setup | `superpowers:writing-plans`, `compound-engineering:ce-commit` |
| 1: Problem Discovery | `compound-engineering:ce-brainstorm`, `compound-engineering:ce-ideate` |
| 2: AI Feasibility | `compound-engineering:ce-brainstorm`, `compound-engineering:ce-web-researcher` |
| 3: Model Development | `compound-engineering:ce-plan`, `claude-api` skill (model selection/pricing) |
| 4: Solution Design | `compound-engineering:ce-plan`, `compound-engineering:ce-doc-review` |
| 5: Testing & Deploy | `compound-engineering:ce-code-review`, `compound-engineering:ce-debug`, `superpowers:verification-before-completion` |
| 6: Iteration | `compound-engineering:ce-ideate`, `compound-engineering:ce-product-pulse` |

For any implementation work within a stage, the standard coding discipline applies:
- Use `superpowers:test-driven-development` for new code
- Use `compound-engineering:ce-commit-push-pr` when ready to commit stage work
- Use `compound-engineering:ce-simplify-code` after implementation

---

## CLAUDE.md Template (fill in after Stage 1)

After Stage 1 is complete, populate `.claude/CLAUDE.md` with the project brief **plus** the behavioral guidelines below. The guidelines come from Karpathy's observations on LLM coding pitfalls and apply to all implementation work in Stages 3–6.

```markdown
# Project: <name>

## Problem Statement
<1-2 sentences from Stage 1 Issue #1.1>

## Business Goals
<from Stage 1 Issue #1.3>

## Current Stage
Stage <N>: <name> — [View open issues](https://github.com/<owner>/<repo>/issues)

## Key Decisions
- [Issue #X] <decision from Stage 2>
- [Issue #Y] <decision from Stage 3>

## Guardrails
- <what the AI must NOT do — from Stage 3 model spec>
- <data/privacy constraints from Stage 2>

---

## Coding Behavior

**Think Before Coding** — State assumptions explicitly. If multiple interpretations exist, present them. If something is unclear, stop and ask.

**Simplicity First** — Minimum code that solves the problem. No features beyond what was asked. No abstractions for single-use code. If it could be 50 lines, don't write 200.

**Surgical Changes** — Touch only what you must. Don't improve adjacent code that isn't broken. Match existing style. Every changed line should trace directly to the request.

**Goal-Driven Execution** — Transform tasks into verifiable goals. For multi-step work, state a brief plan with verify steps before starting.
```

This file is the persistent context keeping Claude aligned across all sessions. Update the "Current Stage" line each time a stage advances.
