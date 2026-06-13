# Project: AI Project Management Skill

## What this project is
A Claude Code + Gemini CLI skill plugin that enforces a rigorous 7-stage AI product management methodology. Skills and commands are installed globally and apply to all AI projects under `ciaranq`.

## Current stage
Stage 1: Problem Discovery — 3 open issues at https://github.com/ciaranq/ai-project-management/issues

## Key decisions made
- **Plugin registration**: `ai-pm@local` in `~/.claude/settings.json`
- **Gemini support**: Skills installed to `~/.gemini/skills/`, commands as TOML at `~/.gemini/commands/ai-pm/`
- **Path convention**: `.claude/` for Claude sessions, `.gemini/` for Gemini sessions, `.ai-session/` for shared handoff state
- **Stage gating**: All GitHub Issues for a stage must be closed before advancing — no exceptions
- **Karpathy guidelines**: Embedded in CLAUDE.md template generated at Stage 1 for every project

## Skill structure
```
skills/
├── ai-pm/          → main skill (auto-triggers on PM context)
├── setup/          → /ai-pm:setup command (Stage 0)
├── stage/          → /ai-pm:stage command (stage navigator)
└── collab/         → /ai-pm:collab command (Claude ↔ Gemini handoff)

gemini/commands/ai-pm/
├── setup.toml
├── stage.toml
└── collab.toml
```

## Active collaboration model
Claude (this session) + Gemini CLI (Hermes, AI Pro, Gemini 2.5 Pro). See `AGENTS.md` for model roles.

## Do not
- Merge to main without closed Stage N issues confirming the work
- Edit installed skill files directly — edit in `skills/` and re-copy to `~/.claude/` and `~/.gemini/`
- Put actual values in `.env` — use `.env.example` only
