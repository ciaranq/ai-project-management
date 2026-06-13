# AI Project Management — Gemini Context

This project contains the `ai-pm` skill: a 7-stage methodology for building AI products using GitHub Issues for gating and progress tracking.

## Available commands

| Command | What it does |
|---------|-------------|
| `/ai-pm:setup` | Stage 0 — full project scaffold (git, GitHub, labels, deps, deploy, DB, .env.example) |
| `/ai-pm:stage` | Show current stage status |
| `/ai-pm:stage next` | Gate-check then advance to the next stage |
| `/ai-pm:stage status` | Full summary of all 7 stages |
| `/ai-pm:stage audit` | Onboard an existing project into the workflow |
| `/ai-pm:stage <N>` | Jump to a specific stage (1–6) |

## Skills active in this session

The following skills are installed at `~/.gemini/skills/` and activate automatically based on context:

- **ai-pm** — main skill, triggers on AI project management work
- **setup** — Stage 0 scaffolding, invoked via `/ai-pm:setup`
- **stage** — stage navigator, invoked via `/ai-pm:stage`

## The 7 stages

| Stage | Name | Gate |
|-------|------|------|
| 0 | Project Setup | Setup checklist issue closed |
| 1 | Problem Discovery | Problem statement, user problems, business goals issues all closed |
| 2 | AI Feasibility & Data Strategy | Feasibility, solution selection, data strategy issues all closed |
| 3 | Model Development | Model spec, success metrics, architecture issues all closed |
| 4 | Solution Design & Integration | UX, integration, feedback, production-readiness issues all closed |
| 5 | Testing, Deployment & Monitoring | Testing, quality gates, trade-offs, monitoring issues all closed |
| 6 | Iteration & Improvement | Feedback analysis, next scope, improvement plan issues all closed |

**Core rule**: Never advance to the next stage until all GitHub Issues for the current stage are closed with documented decisions in the comments.

## Coding behavior (Stages 3–6)

When writing code during implementation stages:

- **Think before coding**: State assumptions explicitly. Surface ambiguity before implementing.
- **Simplicity first**: Minimum code that solves the problem. No speculative features.
- **Surgical changes**: Touch only what the stage requires. Match existing style.
- **Goal-driven**: Frame tasks as verifiable goals. State a brief plan with verify steps before multi-step work.

## Project files

- `RUNBOOK.md` — step-by-step usage guide
- `skills/ai-pm/SKILL.md` — main skill source
- `skills/setup/SKILL.md` — setup command source
- `skills/stage/SKILL.md` — stage command source
- `docs/design/methodology.md` — design rationale
