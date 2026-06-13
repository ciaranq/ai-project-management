# AI Project Management Skill

A Claude Code skill that enforces a rigorous 7-stage methodology for building AI products. Grounded in the AI Product Manager framework, with GitHub Issues as the single source of truth for decisions, gating, and progress.

## What it does

The skill guides any AI project — new or existing — through structured phases, ensuring each phase is completed and documented before the next begins. It integrates with GitHub Issues, compound-engineering skills, and native Claude capabilities to make the process practical rather than ceremonial.

## The 7 Stages

| Stage | Name | Phase |
|-------|------|-------|
| 0 | Project Setup | Foundation |
| 1 | Problem Discovery | Phase 1 |
| 2 | AI Feasibility & Data Strategy | Phase 1 |
| 3 | Model Development | Phase 2 |
| 4 | Solution Design & Integration | Phase 2 |
| 5 | Testing, Deployment & Monitoring | Phase 3 |
| 6 | Iteration & Improvement | Phase 4 |

Each stage creates a set of GitHub Issues. The stage is complete when all its issues are closed with documented answers.

## Installation

The skill is installed as a local Claude Code plugin. It is active in all projects when `ai-pm@local` is enabled in `~/.claude/settings.json`.

To install manually:
1. Copy `skills/ai-pm/` to `~/.claude/plugins/cache/local/ai-pm/1.0.0/skills/ai-pm/`
2. Add to `~/.claude/plugins/installed_plugins.json`:
   ```json
   "ai-pm@local": [{
     "scope": "user",
     "installPath": "~/.claude/plugins/cache/local/ai-pm/1.0.0",
     "version": "1.0.0"
   }]
   ```
3. Add to `~/.claude/settings.json` under `enabledPlugins`: `"ai-pm@local": true`
4. Restart Claude Code

## Usage

The skill triggers automatically when Claude detects AI project management work. You can also invoke it explicitly:

- **Start a new project**: "Start a new AI project for [description]"
- **Check stage status**: "What stage is this project at?"
- **Advance a stage**: "Move to the next stage"
- **Onboard existing project**: "Audit this project and bring it into the AI PM workflow"

See [RUNBOOK.md](RUNBOOK.md) for step-by-step usage and [docs/design/methodology.md](docs/design/methodology.md) for design rationale.

## References

- `role-of-PM.jpg` — The AI Product Manager framework this skill is based on
- `workflow.jpg` — The AI workflow architecture, used in Stage 4 production-readiness gates
