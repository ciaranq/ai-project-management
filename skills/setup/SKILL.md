---
name: setup
description: Run Stage 0 of the AI PM workflow — scaffolds the complete project foundation including git, GitHub, labels, milestones, dependency files, deploy config, database requirements, and .env.example. Invoke with /ai-pm:setup to start any new AI project, or on an existing project to retroactively add the standard scaffold without overwriting existing files.
argument-hint: [--existing]
allowed-tools: [Bash, Read, Write, Edit]
---

# AI PM: Stage 0 Setup

You are running Stage 0 of the AI PM workflow. Your job is to scaffold the complete project foundation. Work through every section in order. Do not skip sections — if something doesn't apply, document why in the setup issue.

The argument `$ARGUMENTS` may contain `--existing` to signal this is an existing project. If not passed, infer from context: if the project has commits, source files, or existing GitHub issues, treat it as existing.

---

## 1. Determine project mode

```bash
git log --oneline -1 2>/dev/null && echo "HAS_COMMITS" || echo "NEW_PROJECT"
gh issue list --state all --json number 2>/dev/null | python3 -c "import json,sys; d=json.load(sys.stdin); print('HAS_ISSUES' if d else 'NO_ISSUES')"
ls src/ app/ lib/ 2>/dev/null && echo "HAS_SOURCE" || echo "NO_SOURCE"
```

- **New project**: no commits, no source, no issues → build from scratch
- **Existing project**: has any of the above → add scaffold without overwriting, skip steps already done

---

## 2. Git repository

```bash
# Check if git is initialised
git rev-parse --git-dir 2>/dev/null && echo "GIT_EXISTS" || git init
```

If no remote exists, create the GitHub repo:
```bash
gh repo view 2>/dev/null || gh repo create $(basename $(pwd)) \
  --public \
  --description "$(read -p 'Brief description: ' d && echo $d)" \
  --source . --remote origin
```

Note the GitHub repo URL for the setup issue.

---

## 3. Folder structure

Create any missing directories — never overwrite existing ones:
```bash
mkdir -p .claude
mkdir -p .github/workflows
mkdir -p docs/problem-discovery
mkdir -p docs/solution-design
mkdir -p docs/testing
mkdir -p docs/iterations
mkdir -p src
```

If `.claude/CLAUDE.md` does not exist, create it with the template from the main `ai-pm` SKILL.md (the "CLAUDE.md Template" section at the bottom). Fill in `<name>` with the project name, leave other fields blank — they get filled after Stage 1.

---

## 4. Environment detection

Detect the project stack:
```bash
ls package.json pyproject.toml requirements.txt Gemfile go.mod Cargo.toml 2>/dev/null
```

For each detected or chosen stack, create the scaffolding file if it doesn't exist:

**Node / TypeScript** (if `package.json` missing):
```json
{
  "name": "<project-name>",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "node src/index.js",
    "test": "echo \"No tests yet\""
  }
}
```
Also create `tsconfig.json` if TypeScript is intended, and `.nvmrc` with the current Node version (`node -v`).

**Python** (if no `pyproject.toml` / `requirements.txt`):
```toml
[project]
name = "<project-name>"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = []
```
Also create `.python-version` with the current version (`python3 --version`).

**Ruby** (if no `Gemfile`): create a minimal Gemfile with `ruby '<version>'`.

**Go** (if no `go.mod`): `go mod init <module-name>`.

If no stack is detectable and this is a new project, ask: "What language/runtime will this project use?"

---

## 5. Deploy scaffolding

Ask the user: "Where will this be deployed?" Options: Vercel / Docker / GitHub Actions CI only / Railway / Render / AWS Lambda / Unknown (scaffold placeholder).

Create the appropriate file if it doesn't already exist:

**Vercel** → `vercel.json`:
```json
{
  "framework": null,
  "buildCommand": null,
  "outputDirectory": null
}
```

**Docker** → `Dockerfile`:
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "src/index.js"]
```
And `.dockerignore`: `node_modules\n.env\n.git`

**GitHub Actions CI** (always create, regardless of other choices) → `.github/workflows/ci.yml`:
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: echo "Add test command here"
```

**Railway** → `railway.json`: `{"deploy": {"startCommand": "npm start"}}`

**AWS Lambda** → note in CLAUDE.md, prompt user to add SAM/Serverless config later.

**Unknown** → create the GitHub Actions CI stub above and note deploy target TBD in CLAUDE.md.

---

## 6. Database / storage

Ask the user: "What storage does this project need?" Options: PostgreSQL / SQLite / MongoDB / Redis / Vector DB / File storage (S3/GCS) / None.

| Choice | Action |
|--------|--------|
| PostgreSQL | `mkdir -p db/migrations`, create `db/schema.sql` with a comment header |
| SQLite | Create `db/` folder, note in CLAUDE.md |
| MongoDB | Create `db/schema.md` describing collections |
| Redis | Note in CLAUDE.md, add `REDIS_URL=` to `.env.example` |
| Vector DB | Ask which provider (Pinecone / pgvector / Qdrant / Weaviate), note in CLAUDE.md, add API key var to `.env.example` |
| File storage | Add `S3_BUCKET=`, `S3_REGION=`, `AWS_ACCESS_KEY_ID=`, `AWS_SECRET_ACCESS_KEY=` to `.env.example` |
| None | Document "No database required" in setup issue |

---

## 7. .env.example

Create or update `.env.example`. Never create or touch `.env` itself.

Start with this base and add any vars identified in step 6:
```
# AI Provider (add keys for providers used in this project)
ANTHROPIC_API_KEY=
# OPENAI_API_KEY=

# App
NODE_ENV=development
PORT=3000

# Add database, storage, and service vars below as they are identified
```

Add a `.gitignore` entry for `.env` if `.gitignore` exists or create one:
```
.env
.env.local
node_modules/
```

---

## 8. GitHub labels and milestones

Check what already exists before creating:
```bash
gh label list --json name --jq '.[].name'
gh api repos/{owner}/{repo}/milestones --jq '.[].title'
```

Create any missing labels:
```bash
gh label create "stage:0-setup"              --color "6e6e6e" --description "Project foundation" 2>/dev/null || true
gh label create "stage:1-problem-discovery"  --color "0075ca" --description "Phase 1: Problem" 2>/dev/null || true
gh label create "stage:2-ai-feasibility"     --color "0052cc" --description "Phase 1: Feasibility" 2>/dev/null || true
gh label create "stage:3-model-dev"          --color "e4e669" --description "Phase 2: Model" 2>/dev/null || true
gh label create "stage:4-solution-design"    --color "d93f0b" --description "Phase 2: Design" 2>/dev/null || true
gh label create "stage:5-testing-deploy"     --color "0e8a16" --description "Phase 3: Launch" 2>/dev/null || true
gh label create "stage:6-iteration"          --color "5319e7" --description "Phase 4: Improve" 2>/dev/null || true
gh label create "blocker"                    --color "ee0701" --description "Blocking stage advance" 2>/dev/null || true
gh label create "decision"                   --color "fef2c0" --description "Key decision documented" 2>/dev/null || true
```

Create any missing milestones:
```bash
REPO=$(gh repo view --json nameWithOwner --jq '.nameWithOwner')
for STAGE in \
  "Stage 1: Problem Discovery|Phase 1 - Problem Discovery" \
  "Stage 2: AI Feasibility|Phase 1 - Feasibility & Data" \
  "Stage 3: Model Development|Phase 2 - Model Spec" \
  "Stage 4: Solution Design|Phase 2 - Design & Integration" \
  "Stage 5: Testing & Deployment|Phase 3 - Launch" \
  "Stage 6: Iteration|Phase 4 - Improve"; do
  TITLE="${STAGE%%|*}"
  DESC="${STAGE##*|}"
  gh api repos/$REPO/milestones --method POST -f title="$TITLE" -f description="$DESC" 2>/dev/null || true
done
```

---

## 9. Initial commit (new projects only)

If this is a new project with no commits:
```bash
git add .
git commit -m "chore: stage 0 project setup

Scaffold created by ai-pm setup — folder structure, dependency files,
deploy config, .env.example, GitHub labels and milestones.

Ready for Stage 1: Problem Discovery."
git push -u origin main
```

For existing projects, commit only newly created scaffold files:
```bash
git add .claude/ .github/ docs/ .env.example .gitignore
git diff --staged --stat
# Commit only if there are staged changes
git diff --staged --quiet || git commit -m "chore: add ai-pm project scaffold

Retroactively added standard folder structure, .env.example, GitHub
labels and milestones to bring project into ai-pm workflow."
git push
```

---

## 10. Create and close the Stage 0 issue

Build the checklist body from what was actually done, then create and immediately close the issue:

```bash
gh issue create \
  --title "[Stage 0] Project setup complete" \
  --label "stage:0-setup" \
  --body "## Setup Checklist
- [x] Git repo
- [x] GitHub remote: <url>
- [x] Labels created
- [x] Milestones created (Stages 1–6)
- [x] Folder structure
- [x] Stack: <detected stack>
- [x] Deploy target: <choice>
- [x] Database: <choice or 'none'>
- [x] .env.example created

## Decisions
Stack: <value>
Deploy: <value>
Database: <value>"
```

Close it immediately:
```bash
gh issue close <number> --comment "Stage 0 complete. Run /ai-pm:stage to begin Stage 1."
```

---

## 11. Report to user

End with a short summary:
- What was created
- Any questions that need follow-up (e.g., deploy target was set to 'unknown')
- Next step: "Stage 0 complete. Run `/ai-pm:stage next` or tell me to start Stage 1."
