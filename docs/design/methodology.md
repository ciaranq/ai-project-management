# Methodology Design Notes

Design rationale for the AI PM skill's 7-stage approach.

## Why stages, not phases?

The PM framework uses 4 phases. The skill uses 7 stages because some phases contain decisions that are causally distinct — making a wrong call in AI Feasibility (Stage 2) doesn't just waste Phase 1 time, it wastes Phase 2, 3, and 4 as well. Splitting Phase 1 into two stages (Problem Discovery + Feasibility) forces the feasibility question to be answered *after* the problem is understood, not before.

Similarly, Phase 2 (Solution Design) splits into Model Development and Solution Design because model selection drives constraints on UX, integration, and cost — you can't design the solution meaningfully until the model is specified.

## Why GitHub Issues as the gate?

Issues are:
- **Permanent** — decisions don't disappear when chat contexts are cleared
- **Traceable** — you can always see why a decision was made via issue comments
- **Actionable** — you can assign, label, and link issues to code changes
- **Integrated** — PRs can reference issues, milestones show stage progress

The alternative (docs/decision records) works but doesn't enforce progression — a doc can be left half-finished and ignored. A closed issue is unambiguous.

## Why the Karpathy behavioral guidelines?

The coding guidelines embedded in CLAUDE.md come from Karpathy's observations on common LLM coding pitfalls. They're included because:

1. AI projects tend to accrete speculative features during Stages 3–4 ("while we're at it...")
2. Implementation stages (3–5) are where the most code is written, and where drift from the documented design is most likely
3. The guidelines are cheap to follow and expensive to violate — overcomplicated code written in Stage 4 becomes tech debt in Stage 6

The four principles map directly to PM discipline:
- **Think Before Coding** = don't implement before Stage 1 & 2 are closed
- **Simplicity First** = scoped to the current stage, not the eventual roadmap
- **Surgical Changes** = don't re-solve problems that prior stages already closed
- **Goal-Driven Execution** = each stage has explicit verifiable outcomes (closed issues)

## Why the workflow.jpg production-readiness gate at Stage 4?

The six principles (Clear Inputs, Reliable Context, Action Boundaries, Validation, Human Oversight, Continuous Improvement) from the AI workflow architecture represent the minimum bar for any AI system operating in production. Gating Stage 4 → Stage 5 on this checklist prevents launching without explicit answers to "what happens when this goes wrong."

Practically: most AI projects that fail in production fail because one of these six was skipped — usually Action Boundaries ("the AI touched something it shouldn't") or Human Oversight ("there was no approval step for high-stakes actions").

## Existing project onboarding

The audit + retroactive issue approach is designed to avoid two failure modes:

1. **"We're already past this"** — teams skip methodology because they feel it doesn't apply to mid-flight projects. Retroactive issues let you apply it to any project at any point.

2. **"Everything is fine"** — without explicit documentation, it's easy to assume decisions were made when they weren't. Creating retroactive issues forces honesty: if you can't write the answer in an issue comment, the decision probably wasn't made.

The `partial` and `missing` status categories are deliberately unsettling. A `missing` Stage 2 on a shipped product is a flag that the AI feasibility was never validated — the team should know that.

## Stage 6 as a loop, not an endpoint

Stage 6 doesn't close the project. It creates a feedback loop back into Stages 3–5. Each iteration is:
- Scoped (one or two focused changes, not a roadmap)
- Documented (new issues under `[Stage 6 - Iter N]`)
- Evaluated (new success criteria in the issues)

This mirrors how good AI products are actually built: the first version is rarely the right one, but the methodology for improving it should be as rigorous as the methodology for building it.
