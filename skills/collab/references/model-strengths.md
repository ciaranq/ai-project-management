# Model Strengths: Claude vs Gemini

Use this guide when deciding which model should handle a task. The goal is not loyalty to a model — it's getting the best outcome fastest.

---

## Claude (claude-sonnet-4-6, Claude Code in VSCode)

### Strengths
- **Multi-step reasoning with dependencies** — each step can change what the next step should be; Claude tracks this well
- **Ambiguity resolution** — when requirements are unclear, Claude surfaces the right clarifying questions and makes defensible judgment calls
- **Architecture and design decisions** — trade-off analysis, API design, choosing abstractions, deciding what NOT to build
- **Code review and security analysis** — catches subtle bugs, logic errors, auth edge cases, injection risks
- **Writing** — documentation, plans, specs, decision records, commit messages, issue bodies
- **Debugging non-obvious issues** — when the cause isn't clear, Claude reasons through the problem systematically
- **ai-pm stage gating** — deciding whether a stage is truly done requires judgment; Claude owns this
- **Critique** — when something needs to be challenged or improved, not just executed

### When NOT to use Claude for this
- Reading more than ~200k tokens of code in one session (context limits)
- Mechanical changes across 50+ files (slow and error-prone compared to Gemini's context)
- Tasks where the answer is already known and just needs to be applied at scale

### Typical Claude tasks in the ai-pm workflow
- Writing the problem statement (Stage 1 issues)
- AI feasibility analysis and decision (Stage 2)
- Designing the model spec and success metrics (Stage 3)
- Solution design and architecture decisions (Stage 4)
- Code review before stage advancement
- Closing issues with documented decisions
- Reviewing Gemini's output before committing

---

## Gemini (Gemini 2.5 Pro via Hermes / AI Pro, Gemini CLI in VSCode)

### Strengths
- **Massive context window (1M tokens)** — can hold an entire large codebase in one session; no chunking needed
- **Whole-codebase analysis** — "find all places where X pattern is used", "what calls this function", "what's the data flow from A to B"
- **Systematic mechanical refactoring** — given a clear spec, apply the same transformation to 100 files accurately
- **Test generation at scale** — read the whole module, generate comprehensive tests for every public function
- **Large document processing** — summarise 500-page docs, extract structured data from large log files
- **Well-defined tasks with large scope** — when the WHAT is clear and the HOW just needs to be applied widely

### When NOT to use Gemini for this
- Tasks requiring judgment about what SHOULD be done (not just what CAN be done)
- Architectural decisions with meaningful trade-offs
- Security-sensitive review (Claude's reasoning chain is better for this)
- Anything where the spec is still unclear

### Typical Gemini tasks in the ai-pm workflow
- Executing mechanical file changes from a Claude-designed spec
- Generating test suites for a complete module
- Analysing the full codebase to answer a specific question
- Bulk documentation generation from existing code
- Large-scale search ("find every API endpoint that doesn't validate input")

---

## Decision guide

| Task type | Use |
|-----------|-----|
| "What should we build?" | Claude |
| "Build what we decided" (small-medium) | Claude |
| "Build what we decided" (large, many files) | Gemini |
| "Is this the right approach?" | Claude |
| "Apply this approach across the codebase" | Gemini |
| "Review this code for correctness" | Claude |
| "Find all usages of X in the codebase" | Gemini |
| "Write the spec for Y" | Claude |
| "Implement the spec for Y" | Depends on size |
| "Generate tests for module Z" | Gemini |
| "Review the tests for quality" | Claude |
| "Debug this weird error" | Claude |
| "Summarise this large log file" | Gemini |
| "Decide if stage N is complete" | Claude |
| "Retroactive audit of existing codebase" | Gemini (discovery) → Claude (judgment) |

---

## Collaboration patterns

### Pattern 1: Design → Execute
1. **Claude**: design the solution, write the spec/plan, create the GitHub Issue
2. **Gemini**: execute the mechanical implementation from the spec
3. **Claude**: review the output, close the issue, advance the stage

Best for: large refactors, bulk test generation, systematic changes with clear specs.

### Pattern 2: Explore → Decide
1. **Gemini**: explore the full codebase, produce a map/analysis of the relevant code
2. **Claude**: read Gemini's analysis, make the architectural decision
3. **Claude or Gemini**: implement, depending on size

Best for: understanding a large unfamiliar codebase, impact analysis before a change.

### Pattern 3: Parallel specialisation
1. **Claude**: works on design docs, issue writing, review
2. **Gemini**: works on implementation and test generation in parallel

Best for: when the design and implementation can proceed independently.

### Pattern 4: Review cycle
1. Any model: produces output (code, docs, design)
2. Other model: reviews with fresh eyes
3. First model: addresses feedback

Best for: catching blind spots. Each model will miss different things.

---

## What each model reads on startup

**Claude** reads `.claude/CLAUDE.md` — keep this updated with current stage, key decisions, and any handoff summary.

**Gemini** reads `GEMINI.md` at project root — keep this updated the same way.

Both models read `.ai-session/handoff.md` when the `/ai-pm:collab pickup` command is run.
