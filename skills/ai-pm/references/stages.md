# Stage Issue Templates

For each stage, create these issues using `gh issue create`. Each issue body includes a checklist of questions that must be answered before the issue can be closed. Document answers directly in the issue as comments.

---

## Stage 1: Problem Discovery

**Milestone**: "Stage 1: Problem Discovery"  
**Label**: `stage:1-problem-discovery`

### Issue 1.1 — Problem Statement
```
Title: [Stage 1] Define the problem we're solving

## What problem are we solving to create impact?

Answer these before closing:
- [ ] One-sentence problem statement written
- [ ] Who experiences this problem? (user segment)
- [ ] How often does this problem occur?
- [ ] What happens today without a solution? (current workaround/cost)
- [ ] Why is NOW the right time to solve this?

## Decision to document
Agreed problem statement: _________
```

### Issue 1.2 — User Problems
```
Title: [Stage 1] Document the biggest user problems

## What are the biggest user problems?

Answer these before closing:
- [ ] Top 3 user pain points listed with evidence (research, quotes, data)
- [ ] Pain points ranked by frequency and severity
- [ ] User journey mapped: where does the pain hit hardest?
- [ ] Validated with at least one real user (or proxy evidence cited)

## Decision to document
Primary user pain point we're targeting: _________
```

### Issue 1.3 — Business Goals
```
Title: [Stage 1] Define business goals

## What are the business goals?

Answer these before closing:
- [ ] Primary business goal stated (revenue / cost / retention / other)
- [ ] How does solving this problem achieve the business goal?
- [ ] Success metric defined (what number moves, by how much, by when?)
- [ ] Stakeholders identified and aligned
- [ ] Constraints noted (budget, timeline, regulatory)

## Decision to document
Business goal and success metric: _________
```

**Deliverable**: Save `docs/problem-discovery/stage-1-summary.md` with all three issue answers compiled.

---

## Stage 2: AI Feasibility & Data Strategy

**Milestone**: "Stage 2: AI Feasibility"  
**Label**: `stage:2-ai-feasibility`

### Issue 2.1 — AI Feasibility
```
Title: [Stage 2] Validate that AI is the right approach

## Should we use AI? Is AI the best way to solve this problem?

Answer these before closing:
- [ ] Alternatives to AI considered (rules engine, search, manual process)
- [ ] Why AI is better than alternatives (or: decision NOT to use AI, with rationale)
- [ ] What specific AI capability solves the problem? (classification / generation / retrieval / reasoning / other)
- [ ] Risk assessment: what happens when the AI is wrong?
- [ ] Reversibility: can users correct AI mistakes?

## Decision to document
Use AI: Yes/No. Type of AI: _______. Justification: _________
```

### Issue 2.2 — Solution Selection
```
Title: [Stage 2] Choose the best solution approach

## What is the best solution?

Answer these before closing:
- [ ] 2–3 solution options explored
- [ ] Options compared on: accuracy, cost, time-to-build, maintainability
- [ ] Chosen solution described in one paragraph
- [ ] Build vs. buy decision made (existing model/API vs. custom)
- [ ] Dependencies and integrations identified

## Decision to document
Chosen solution: _________. Key trade-off accepted: _________
```

### Issue 2.3 — Data Strategy
```
Title: [Stage 2] Define data availability and strategy

## Do we have the data we need? What is our data strategy?

Answer these before closing:
- [ ] Required data sources listed
- [ ] Data availability confirmed (or gap plan documented)
- [ ] Data quality assessed: is it complete and unbiased?
- [ ] Quality vs. quantity trade-off articulated
- [ ] Data access/permissions confirmed
- [ ] PII/privacy considerations noted
- [ ] How data will be collected, stored, and used

## Decision to document
Data strategy: _________. Known data gaps and mitigation: _________
```

**Deliverable**: Save `docs/problem-discovery/stage-2-feasibility.md` with all decisions.

---

## Stage 3: Model Development

**Milestone**: "Stage 3: Model Development"  
**Label**: `stage:3-model-dev`

### Issue 3.1 — Model Specification
```
Title: [Stage 3] Specify what the model does

## How do we translate the solution into a model? What should the model do?

Answer these before closing:
- [ ] Model inputs defined (what goes in?)
- [ ] Model outputs defined (what comes out? format? constraints?)
- [ ] Model behaviour described for happy path
- [ ] Edge cases and failure modes listed
- [ ] System prompt / instructions drafted (if using LLM)
- [ ] Tools/functions the model can call (if agentic)
- [ ] What the model must NOT do (guardrails)

## Decision to document
Model spec: inputs=_____, outputs=_____, key guardrails=_____
```

### Issue 3.2 — Success Metrics
```
Title: [Stage 3] Define how we measure model success

## How do we measure the model's success?

Answer these before closing:
- [ ] Primary metric defined (accuracy / task completion rate / latency / user satisfaction / other)
- [ ] Baseline established (what does "good enough" look like?)
- [ ] Evaluation method described (human eval / automated / A-B test)
- [ ] Threshold for launch readiness set (minimum acceptable score)
- [ ] Ongoing monitoring metric defined (post-launch)
- [ ] Failure rate threshold: at what error rate do we pause/rollback?

## Decision to document
Launch threshold: _________. Monitoring metric: _________. Rollback trigger: _________
```

### Issue 3.3 — Model Architecture
```
Title: [Stage 3] Choose model architecture and provider

## Which model/architecture and why?

Answer these before closing:
- [ ] Model options evaluated (provider, size, fine-tuned vs. base)
- [ ] Cost per request estimated
- [ ] Latency requirements met
- [ ] Context window requirements assessed
- [ ] Fine-tuning vs. prompting vs. RAG decision made
- [ ] Prototype/spike done to validate feasibility

## Decision to document
Model choice: _________. Cost estimate: _________. Why over alternatives: _________
```

**Deliverable**: Save `docs/solution-design/stage-3-model-spec.md`.

---

## Stage 4: Solution Design & Integration

**Milestone**: "Stage 4: Solution Design"  
**Label**: `stage:4-solution-design`

### Issue 4.1 — User Experience Design
```
Title: [Stage 4] Design the user experience

## How do users experience the solution? What should the UX look like?

Answer these before closing:
- [ ] User journey through the AI feature mapped
- [ ] UI/interaction design described or mocked
- [ ] Loading states and async behaviour handled
- [ ] How uncertainty/confidence is communicated to users
- [ ] Tone and style of AI responses defined
- [ ] Accessibility considerations noted

## Decision to document
UX approach: _________. How we handle uncertainty: _________
```

### Issue 4.2 — Integration & System Design
```
Title: [Stage 4] Design integrations and system architecture

## How does the AI integrate with existing systems?

Answer these before closing:
- [ ] Data flow diagram drawn (inputs → AI → outputs → downstream)
- [ ] All system touchpoints listed (APIs, databases, services)
- [ ] Authentication and authorisation designed
- [ ] Rate limiting and cost controls designed
- [ ] Caching strategy (if any)
- [ ] Fallback behaviour when AI is unavailable

## Decision to document
Integration architecture summary: _________. Fallback plan: _________
```

### Issue 4.3 — Feedback Collection
```
Title: [Stage 4] Design the feedback loop

## How do we collect feedback? What can users do when they get wrong answers?

Answer these before closing:
- [ ] Feedback mechanism designed (thumbs up/down, rating, free text, implicit signals)
- [ ] Correction flow designed: how users fix wrong AI outputs
- [ ] Feedback data stored and accessible for analysis
- [ ] Escalation path for harmful/seriously wrong responses
- [ ] Feedback-to-improvement loop described

## Decision to document
Feedback mechanism: _________. Correction flow: _________
```

### Issue 4.4 — Production Readiness Checklist
```
Title: [Stage 4] Production readiness sign-off

## Is this ready for production?

Check every item before closing:
- [ ] **Clear Inputs**: Trigger is defined. Required context/data is specified.
- [ ] **Reliable Context**: Right docs, history, and business rules available to model.
- [ ] **Action Boundaries**: Explicit list of tools and systems the AI can touch.
- [ ] **Validation**: Output validation layer defined. Who/what checks before action.
- [ ] **Human Oversight**: Human-in-the-loop approval points identified for high-risk actions.
- [ ] **Continuous Improvement**: Logging, feedback collection, and iteration loop planned.

## Decision to document
Outstanding risks before launch: _________. Accepted risks: _________
```

**Deliverable**: Save `docs/solution-design/stage-4-design.md`.

---

## Stage 5: Testing, Deployment & Monitoring

**Milestone**: "Stage 5: Testing & Deployment"  
**Label**: `stage:5-testing-deploy`

### Issue 5.1 — Testing Plan
```
Title: [Stage 5] Define and execute testing plan

## How do we launch with high quality? How do we test at scale?

Answer these before closing:
- [ ] Unit/integration test coverage for AI pipeline components
- [ ] Evaluation dataset created (representative test cases)
- [ ] Accuracy against evaluation dataset measured and meets launch threshold
- [ ] Edge cases tested (empty inputs, adversarial inputs, max-length inputs)
- [ ] Latency tested under expected load
- [ ] Cost per request validated against budget

## Decision to document
Test results: accuracy=___%, latency=___ms, cost/req=$_____. Launch: Go/No-Go
```

### Issue 5.2 — Response Quality Gates
```
Title: [Stage 5] Ensure responses are accurate and adhere to guidelines

## How do we ensure responses are accurate and follow guidelines?

Answer these before closing:
- [ ] Content policy / guidelines documented
- [ ] Automated guardrail checks in place (moderation, format validation)
- [ ] Sample of outputs reviewed by humans
- [ ] Problematic output categories identified and handled
- [ ] Response quality scoring defined

## Decision to document
Quality gate criteria: _________. Known failure modes and mitigations: _________
```

### Issue 5.3 — Quality vs. Latency vs. Cost Trade-off
```
Title: [Stage 5] Document quality/latency/cost trade-off decisions

## How do we prioritise quality vs. latency vs. cost?

Answer these before closing:
- [ ] Current metrics: quality=___, latency=___ms, cost/req=$___
- [ ] Acceptable ranges for each defined
- [ ] Optimisation levers identified (model size, caching, streaming, etc.)
- [ ] Agreed trade-off position documented
- [ ] Re-evaluation trigger defined (what metric change prompts revisiting)

## Decision to document
Accepted trade-off: _________. Optimisation backlog items: _________
```

### Issue 5.4 — Monitoring & Rollback Plan
```
Title: [Stage 5] Set up monitoring and define rollback criteria

## What do we do if responses are not good enough?

Answer these before closing:
- [ ] Monitoring dashboards set up (error rate, latency, cost, quality signals)
- [ ] Alerting thresholds configured
- [ ] Rollback procedure documented (how to revert, who decides)
- [ ] Rollback trigger defined (specific metric thresholds)
- [ ] On-call / escalation path defined
- [ ] Post-launch review scheduled (1-week, 1-month)

## Decision to document
Rollback trigger: _________. On-call owner: _________. Review dates: _________
```

**Deliverable**: Save `docs/testing/stage-5-launch.md` with test results and launch decision.

---

## Stage 6: Iteration & Improvement

**Milestone**: "Stage 6: Iteration"  
**Label**: `stage:6-iteration`

Stage 6 is an ongoing cycle. Each iteration creates a new set of issues. Name them with an iteration number: `[Stage 6 - Iter 1]`, `[Stage 6 - Iter 2]`, etc.

### Issue 6.1 — Feedback Analysis
```
Title: [Stage 6 - Iter N] Analyse user feedback

## How do we learn from user feedback?

Answer these before closing:
- [ ] Feedback data from Stage 5 collection mechanism reviewed
- [ ] Top pain points and complaints identified
- [ ] Positive signals (what's working) identified
- [ ] Volume and trends quantified
- [ ] Specific failure cases documented with examples

## Decision to document
Top insight from feedback: _________. Highest-impact improvement to make: _________
```

### Issue 6.2 — Next Iteration Scope
```
Title: [Stage 6 - Iter N] Define scope for next iteration

## What do we do next? Do we build new capabilities or improve existing ones?

Answer these before closing:
- [ ] Options generated: new features vs. quality improvements vs. performance
- [ ] Each option scored: impact, effort, risk
- [ ] Chosen scope defined (no more than 1-2 focused changes per iteration)
- [ ] Entry point confirmed: does this re-enter at Stage 3, 4, or 5?
- [ ] Success criteria for this iteration set

## Decision to document
Iteration N scope: _________. Enters at Stage ___. Success criteria: _________
```

### Issue 6.3 — Improvement Plan: Good to Great
```
Title: [Stage 6 - Iter N] Plan how to go from "good" to "great"

## How do we go from "good" to "great"?

Answer these before closing:
- [ ] Current state: where is quality/performance/adoption today?
- [ ] "Great" state defined: what specific metrics define greatness?
- [ ] Gap analysis done
- [ ] Highest-leverage investments identified
- [ ] Timeline and milestones set

## Decision to document
Definition of "great" for this product: _________. Primary lever to pull: _________
```

**Deliverable**: Save `docs/iterations/iteration-N.md` with feedback analysis, scope decision, and plan.

After closing all Stage 6 issues, begin the next iteration by re-entering the relevant stage (3, 4, or 5) with new issues scoped to the iteration's focus.
