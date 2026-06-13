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

### Claude Code

The skill installs as a local plugin and is active in all projects.

```bash
# 1. Copy skill files
mkdir -p ~/.claude/plugins/cache/local/ai-pm/1.0.0/skills
cp -r skills/ai-pm skills/setup skills/stage ~/.claude/plugins/cache/local/ai-pm/1.0.0/skills/
cp skills/ai-pm/plugin.json ~/.claude/plugins/cache/local/ai-pm/1.0.0/.claude-plugin/plugin.json

# 2. Register the plugin (add to ~/.claude/plugins/installed_plugins.json under "plugins")
# "ai-pm@local": [{"scope":"user","installPath":"~/.claude/plugins/cache/local/ai-pm/1.0.0","version":"1.0.0"}]

# 3. Enable in ~/.claude/settings.json under "enabledPlugins"
# "ai-pm@local": true

# 4. Restart Claude Code
```

### Gemini CLI (Hermes / AI Pro)

The same SKILL.md format works natively in Gemini CLI. Skills are installed to `~/.gemini/skills/` and commands as TOML files in `~/.gemini/commands/`.

```bash
# Install skills (with .claude/ → .gemini/ path rewrite)
mkdir -p ~/.gemini/skills/ai-pm/references ~/.gemini/skills/setup ~/.gemini/skills/stage
mkdir -p ~/.gemini/commands/ai-pm

sed 's/\.claude\//\.gemini\//g; s/CLAUDE\.md/GEMINI.md/g' skills/ai-pm/SKILL.md > ~/.gemini/skills/ai-pm/SKILL.md
sed 's/\.claude\//\.gemini\//g; s/CLAUDE\.md/GEMINI.md/g' skills/ai-pm/references/stages.md > ~/.gemini/skills/ai-pm/references/stages.md
sed 's/\.claude\//\.gemini\//g; s/CLAUDE\.md/GEMINI.md/g' skills/setup/SKILL.md > ~/.gemini/skills/setup/SKILL.md
sed 's/\.claude\//\.gemini\//g; s/CLAUDE\.md/GEMINI.md/g' skills/stage/SKILL.md > ~/.gemini/skills/stage/SKILL.md

# Install TOML commands
cp gemini/commands/ai-pm/setup.toml ~/.gemini/commands/ai-pm/setup.toml
cp gemini/commands/ai-pm/stage.toml ~/.gemini/commands/ai-pm/stage.toml
```

Gemini activates skills automatically via description matching — no plugin registration needed. Commands are available as `/ai-pm:setup` and `/ai-pm:stage` immediately after installation.

## Commands

| Command | What it does |
|---------|-------------|
| `/ai-pm:setup` | Stage 0 — scaffolds git, GitHub, labels, milestones, dependencies, deploy config, DB, `.env.example` |
| `/ai-pm:stage` | Show current stage status and what's blocking progress |
| `/ai-pm:stage next` | Gate-check current stage then advance — creates all issues for the next stage |
| `/ai-pm:stage status` | Full summary table of all 7 stages |
| `/ai-pm:stage audit` | Onboard an existing project — scans the codebase, maps to stages, creates retroactive issues |
| `/ai-pm:stage <N>` | Jump to a specific stage (1–6), with gate checks and confirmation |

The `ai-pm` skill also triggers automatically when Claude detects AI project management context — you don't always need to use the commands explicitly.

See [RUNBOOK.md](RUNBOOK.md) for step-by-step usage and [docs/design/methodology.md](docs/design/methodology.md) for design rationale.

## References

- `role-of-PM.jpg` — The AI Product Manager framework this skill is based on
- `workflow.jpg` — The AI workflow architecture, used in Stage 4 production-readiness gates
