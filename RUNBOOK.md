# AI PM Skill — Runbook

Step-by-step guide for using the `ai-pm` skill in Claude Code.

---

## Starting a New Project

### 1. Trigger the skill

Tell Claude: _"Start a new AI project — [brief description]"_

The skill will detect no existing stages and begin Stage 0.

### 2. Stage 0: Project Setup

Claude will:
- Confirm or create a git repo
- Create or connect to a GitHub remote (`gh repo create`)
- Create the label taxonomy (9 labels, `stage:0-setup` through `stage:6-iteration` plus `blocker` and `decision`)
- Create milestones for Stages 1–6
- Create the folder structure (`docs/`, `.claude/`, `src/`)
- Create and close a setup checklist issue

When Stage 0 is done, you'll have a clean, consistent project scaffold. Verify with:
```bash
gh label list
gh api repos/{owner}/{repo}/milestones --jq '.[].title'
```

### 3. Stages 1–6: Issue-Driven Progression

Each stage follows the same rhythm:

1. Claude creates the issues for this stage
2. You work through the questions in each issue (Claude helps with research, brainstorming, documentation)
3. When an issue's questions are answered, close it with a comment documenting the decision
4. Once all issues are closed, Claude confirms completion and asks if you're ready to advance

**Key commands during a stage:**
```bash
# See what's open and blocking
gh issue list --label "stage:N-name" --state open

# Close an issue with a decision record
gh issue close <number> --comment "Decision: ..."

# See the full picture
gh issue list --state all
```

### 4. Advancing Stages

Tell Claude: _"Stage N is complete, let's move to Stage N+1"_

Claude will:
- Verify all issues for Stage N are closed (blocks if not)
- Summarise what was decided
- Create the issues for Stage N+1

---

## Onboarding an Existing Project

Use this when a project already exists but wasn't started with this methodology.

### 1. Trigger the audit

Tell Claude: _"Audit this project and bring it into the AI PM workflow"_

Claude will scan:
- Existing docs, README, specs
- Git history (what was built and why)
- Any existing GitHub Issues

### 2. Review the stage map

Claude will present a table showing each stage's status:
- `complete` — all key questions answered and evidenced
- `partial` — some answered, some not
- `missing` — no evidence this stage was done
- `n/a` — not applicable

### 3. Retroactive issues

For stages marked `complete` or `partial`, Claude creates and closes issues (pre-filled with answers from existing docs). For `missing` stages, issues are created open as blockers.

This gives you a truthful picture of what was actually done rigorously vs. assumed.

### 4. Enter at the right stage

Once retroactive issues are settled, Claude identifies the first stage with open issues. That's where active work begins.

---

## Stage-by-Stage Reference

### Stage 1: Problem Discovery

**Questions to answer:**
- What problem are we solving to create impact?
- What are the biggest user problems? (with evidence)
- What are the business goals and success metrics?

**Deliverable:** `docs/problem-discovery/stage-1-summary.md`

**Tools to use:** `compound-engineering:ce-brainstorm`, `compound-engineering:ce-ideate`

---

### Stage 2: AI Feasibility & Data Strategy

**Questions to answer:**
- Should we use AI? Is it better than alternatives?
- Which AI approach is best?
- Do we have the required data? Is it complete and unbiased?

**Deliverable:** `docs/problem-discovery/stage-2-feasibility.md`

**Tools to use:** `compound-engineering:ce-brainstorm`, `compound-engineering:ce-web-researcher`

---

### Stage 3: Model Development

**Questions to answer:**
- What does the model do? (inputs, outputs, constraints)
- What should it NOT do? (guardrails)
- How do we measure success? (metrics, thresholds, monitoring)

**Deliverable:** `docs/solution-design/stage-3-model-spec.md`

**Tools to use:** `compound-engineering:ce-plan`, `claude-api` skill for model selection

---

### Stage 4: Solution Design & Integration

**Questions to answer:**
- How do users experience the solution?
- How does it integrate with existing systems?
- How do we collect feedback and handle wrong answers?

**Production-readiness gate (from workflow.jpg):**
Before closing Stage 4, all six must be checked: Clear Inputs, Reliable Context, Action Boundaries, Validation, Human Oversight, Continuous Improvement.

**Deliverable:** `docs/solution-design/stage-4-design.md`

**Tools to use:** `compound-engineering:ce-plan`, `compound-engineering:ce-doc-review`

---

### Stage 5: Testing, Deployment & Monitoring

**Questions to answer:**
- Does it meet the quality threshold from Stage 3?
- How do we prioritise quality vs. latency vs. cost?
- What's the rollback plan if responses aren't good enough?

**Deliverable:** `docs/testing/stage-5-launch.md`

**Tools to use:** `compound-engineering:ce-code-review`, `compound-engineering:ce-debug`, `superpowers:verification-before-completion`

---

### Stage 6: Iteration & Improvement

**Questions to answer:**
- What did users actually experience? (feedback analysis)
- What's the highest-leverage improvement?
- Build new capabilities or improve existing?

**Deliverable:** `docs/iterations/iteration-N.md`

Stage 6 loops back into Stage 3, 4, or 5 for each iteration. Each iteration creates new numbered issues (`[Stage 6 - Iter N]`).

**Tools to use:** `compound-engineering:ce-ideate`, `compound-engineering:ce-product-pulse`

---

## Coding Discipline (Stages 3–6)

When Claude writes code during implementation stages, the following applies:

**Think Before Coding** — Assumptions stated explicitly. Multiple interpretations surfaced, not silently chosen. Unclear things named and questioned before implementation.

**Simplicity First** — Minimum code that solves the problem. No speculative features. No abstractions for single-use code. If it could be 50 lines, it won't be 200.

**Surgical Changes** — Only what the stage requires is touched. Adjacent code is not "improved". Style matches what exists.

**Goal-Driven Execution** — Tasks framed as verifiable goals. Multi-step work gets a brief plan with verify steps before starting.

---

## Common Patterns

### "We skipped design — the engineer just built it"
Run an onboarding audit. Stages 1–4 will likely show as `partial` or `missing`. Create retroactive issues for what was implicitly decided, then open issues for what wasn't. Now you have a documented baseline to iterate from.

### "We want to add a new capability to an existing AI feature"
Enter at Stage 3 (Model Development) with an iteration-scoped issue set. You don't need to redo Stages 1–2 unless the problem or feasibility has changed.

### "The model quality is degrading"
Enter at Stage 5 (Testing & Monitoring). Create a new issue `[Stage 5 - Regression] ...` and work through it before touching Stage 3.

### "Business goals changed"
Re-enter at Stage 1 with updated issues. Mark the old Stage 1 issues as superseded in comments. This creates an audit trail showing why direction changed.
