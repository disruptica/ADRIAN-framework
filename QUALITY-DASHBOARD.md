# Quality Dashboard

Five metrics. That's it. Anything more becomes a status report and stops driving change.

Each metric has:
- **What it measures**
- **How to compute it**
- **Healthy / watch / alert thresholds**
- **What it triggers when it goes wrong**

Reviewed in the Quality Review — the team's one recurring meeting, 30–45 minutes, every day or every other day (see [`ROLES.md`](./ROLES.md#the-team-cadence)). The review must produce a change to something — `CLAUDE.md`, the Request Contract, the gates, or the thresholds themselves — or an explicit decision that nothing needs changing. A week of reviews that produced no change failed.

---

## Why these five (and not the obvious ones)

You'll notice **velocity isn't here**. Neither is **lines of code** or **PRs per week**. That's deliberate.

The 2026 evidence is clear: at 85%+ AI generation, velocity metrics go up regardless of quality. PRs per developer increased 98% with AI; delivery velocity stayed flat or worsened. Tracking velocity at this AI ratio is misleading at best, harmful at worst.

The five below are chosen because they detect the *failure modes specific to AI-first development*:

1. Contract quality → Respawn rate (defect split)
2. Agent context quality → First-time CI pass rate
3. Test theater → Mutation score
4. AI-specific technical debt → 30-day rework rate on AI code
5. Aggregate delivery health → Change failure rate

If you can only track three: pick **defect respawn rate, mutation score, and change failure rate**. Those three catch the most.

---

## Metric 1 — Respawn Rate

### What it measures
How often an approved Request gets killed and respawned. Contracts are locked at approval — mid-flight change means kill-and-respawn, never mutation. The rate itself isn't the signal; **the cause split is**:

- **Killed for priority** — the queue reshuffled and this Request no longer mattered. Neutral to good: ADRIAN rewards change; this is the squash game working.
- **Killed for contract defect** — the contract was wrong, ambiguous, or missing context, discovered in Implement. This is a Phase 1 quality signal: intake missed something or the PM's pressure-test did.

### How to compute
```
respawn_rate         = (requests_killed_this_week / requests_started_this_week) * 100
defect_respawn_rate  = (killed_for_contract_defect / requests_started_this_week) * 100
```

Track both. Only the defect rate has thresholds — priority kills are expected and unbounded by design.

### Thresholds (defect respawn rate)

| Band | Range | Meaning |
|---|---|---|
| Healthy | <10% | Contracts are trustworthy |
| Watch | 10–20% | Intake or PM review needs attention |
| Alert | >20% | Phase 1 is rushed or draft quality is degrading |

### What it triggers

**Watch:** Bring two defect-killed contracts to the Quality Review. Discuss the pattern — was it a question intake should have asked, missing codebase context in Technical Notes, or a PM pressure-test miss?

**Alert:** Hold all non-urgent new Requests for one week. Use the time to:
- Update the intake prompt with the recurring miss pattern
- Tighten the Request Contract template if a section needs more rigor
- Recalibrate the PM's five checks on the specific failure mode

### Defect causes to track separately
- Intake didn't surface a key fact (prompt fix)
- Technical Notes missed relevant code (codebase-analysis fix)
- Acceptance Criteria were incomplete (template fix)
- Architectural impossibility surfaced in Implement (Dev consultation gap in Align)

One cause NOT to count as defect: the user changed their mind after seeing early output. That's a priority kill — reality moved, the system reacted. Reward it.

## Metric 2 — First-Time CI Pass Rate (AI PRs)

### What it measures
Of PRs marked AI-generated, the percentage that pass all CI checks on their first push (before any human intervention or retries).

### How to compute
```
ai_first_time_pass = (ai_prs_passing_ci_on_first_push / total_ai_prs) * 100
```

Restrict to PRs tagged AI-generated. Compute weekly.

### Thresholds

| Band | Range | Meaning |
|---|---|---|
| Healthy | >80% | Agent context is sufficient; rules are working |
| Watch | 60–80% | Rules drift or context gaps appearing |
| Alert | <60% | `CLAUDE.md` is stale or fitness functions are too strict for current code |

### What it triggers

**Watch:** Audit the last 10 first-time-failing AI PRs. Group failures by type:
- Lint / format → `CLAUDE.md` style rules need updating
- Type errors → context loading missing types
- Test failures → contract ambiguity or agent overfitting
- Fitness function failures → architectural rules unclear

**Alert:** Stop adding new `CLAUDE.md` rules until existing ones are working. The agent is either getting too much instruction (contradiction) or too little (gaps). Audit and reset.

### Why this matters specifically for AI work
A low first-time pass rate doesn't just mean wasted CI time — it means the agent is missing context. Each retry that requires human intervention is a productivity tax. At 85%+ AI generation, this metric is the proxy for whether the agentic workflow is healthy.

---

## Metric 3 — Mutation Score on Critical Paths

### What it measures
The percentage of code mutants killed by your test suite on code paths marked critical. Mutation testing deliberately breaks the code in small ways; if your tests still pass, you have test theater.

### How to compute
Per language:
- TS/Node: Stryker — `stryker run`, report shows mutation score per file
- Python: mutmut or cosmic-ray
- Go: gremlins or go-mutesting

Aggregate by critical-path files only. Don't waste cycles on glue code.

### Thresholds

| Band | Range | Meaning |
|---|---|---|
| Healthy | >75% | Tests catch real bugs |
| Watch | 60–75% | Tests have gaps |
| Alert | <60% | Tests are mostly theater on critical paths |

### What it triggers

**Watch:** QA reviews the surviving mutants. Either:
- Add tests to kill them (good)
- Mark them as equivalent mutants if they truly don't change behavior (rare, document the reasoning)

**Alert:** Halt deploys of new features touching critical paths until the score recovers. This is the strongest gate-tightening response in ADRIAN — but it's appropriate because <60% mutation score on critical paths means you can't trust your test suite, and AI agents produce code that passes those tests by construction.

### Why this is the most important metric for AI code
Agents produce tests that are tautological — they assert the implementation works as the implementation says it works. Mutation testing breaks the implementation and checks the test. It's the single best automated detector of "passes tests but feature is wrong."

### What to mark as critical path
- Money movement
- Auth and authz
- Multi-tenant data access
- PHI/PCI/sensitive data handling
- External API integration boundaries
- Anything where a silent failure produces a customer-visible bug

Don't mark everything critical. The tool gets slow and the signal gets diluted.

---

## Metric 4 — 30-Day Rework Rate on AI-Touched Code

### What it measures
Of lines AI generated 30 days ago, the percentage that have been modified, reverted, or deleted since. High rework means AI is producing code that doesn't survive contact with reality.

### How to compute
Requires AI-attribution tagging on PRs. Then:
```
rework_rate = (lines_modified_or_deleted_within_30_days / lines_ai_generated_30_days_ago) * 100
```

Tools that automate this: Exceeds AI, custom git analysis script. A simple script using `git blame` and AI-tagged commit metadata can produce this if you don't want to buy a tool.

### Thresholds

| Band | Range | Meaning |
|---|---|---|
| Healthy | <15% | Normal refactoring |
| Watch | 15–25% | AI output quality slipping or context degrading |
| Alert | >25% | Producing technical debt faster than value |

### What it triggers

**Watch:** Sample 5 high-rework files. Read the original AI PR and the rework PRs. Identify the pattern:
- Was the contract wrong? → Phase 1 issue
- Was the context insufficient? → `CLAUDE.md` issue
- Was the agent improvising? → rules need tightening
- Was the original contract ambiguous in a way that surfaced after deploy? → Request Contract template gap

**Alert:** This is the technical debt leading indicator. At >25% sustained for two weeks, slow AI generation pace by at least 30% and direct freed capacity to:
- Refactor the highest-rework files
- Audit `CLAUDE.md` against the observed failures
- Tighten Phase 1 review

Do NOT respond by adding more human review to PRs — that's solving the wrong end. The fix is upstream context and downstream verification, not midstream review.

### Why 30 days
Shorter windows miss the "passes review but breaks in prod" failures. Longer windows take too long to react. 30 days is the published consensus.

---

## Metric 5 — Change Failure Rate

### What it measures
The percentage of deploys that result in degraded service requiring remediation (rollback, hotfix, incident response).

### How to compute
DORA standard:
```
cfr = (deploys_requiring_remediation_this_period / total_deploys_this_period) * 100
```

Per-service or aggregate. Aggregate is what to watch in the Quality Review.

### Thresholds

| Band | Range | Meaning |
|---|---|---|
| Elite | <5% | DORA elite band |
| High | 5–10% | DORA high band |
| Medium | 10–15% | DORA medium band |
| Low / Alert | >15% | Stop and diagnose |

The 2026 data shows AI-first teams without ADRIAN-style controls land in the 10–15% range. With ADRIAN controls, elite (<5%) is achievable. Aim for <5%, accept <10%, fix anything sustained above.

### What it triggers

**Healthy:** No action. Don't optimize what's working.

**High band:** Bring the period's failures to the Quality Review. Categorize each:
- Contract quality (Phase 1 issue)
- Implementation quality (Phase 2 issue — agent, review, or refactor gap)
- Pipeline gap (Phase 3 — a check should have caught it)
- Verification gap (Phase 3 — didn't catch in canary)

Add one pipeline check or one `CLAUDE.md` rule per category to prevent the next instance.

**Alert (>15%):** Halt non-urgent deploys. Run a focused improvement week. Pick the top failure category and fix it. Repeat next week if needed.

---

## The dashboard format

Keep it simple. Notion page, GitHub README, plain text — doesn't matter. The numbers stay weekly-windowed; refresh the snapshot weekly and the Quality Review reads the current one every time it meets.

```markdown
# Quality Dashboard — Week of <date>

## At a glance
| Metric | This week | Last week | Band |
|---|---|---|---|
| Defect respawn rate | 7% | 12% | Healthy |
| AI PR first-time CI pass | 78% | 81% | Watch |
| Mutation score (critical) | 79% | 77% | Healthy |
| 30-day rework on AI code | 14% | 13% | Healthy |
| Change failure rate | 4% | 6% | Elite |

## Actions from last week's review
- Updated `CLAUDE.md` in payments-service with the audit logger rule (closed)
- Drafted new intake prompt for PHI tagging (in progress)

## This week's action
- AI PR first-time CI pass dropped — audit last 10 failures by Friday
```

Don't add charts unless they tell a story numbers can't. Don't add metrics unless they trigger an action.

---

## When to revise the thresholds

The thresholds in this doc are starting points based on 2026 industry benchmarks. Your team should revise them after two months of data — your baseline and your target may differ from the published ones.

Revise them when:
- A metric is consistently in one band for two months → tighten the watch threshold
- A metric is bouncing between bands every week → loosen, you're seeing noise
- A new failure mode is consistently slipping through → consider adding a sixth metric

But six metrics is the max. Past that, the dashboard becomes a report and stops driving change.

---

## Anti-metrics — what NOT to track

These metrics will tempt you. Resist.

| Metric | Why it's misleading |
|---|---|
| Lines of code | At 85%+ AI generation, LOC inflates regardless of value |
| PRs per developer | Up 98% with AI, decoupled from delivered value |
| Velocity / story points | Subjective at the human level, meaningless at agentic scale |
| Acceptance rate (Copilot-style) | Local productivity proxy, doesn't capture downstream impact |
| Test coverage | High coverage with low mutation score = test theater |
| Time in review | At AI scale, optimizing this leads to weakening review |

These metrics can be useful tools at the individual level (e.g., a Dev tuning their agent loop) but they should not appear on the team dashboard. Team metrics drive team behavior; the wrong team metric produces the wrong team behavior at scale.

---

## What the Quality Review meeting looks like

30–45 minutes. Whole team. Every day or every other day — the cadence follows request throughput, not the calendar. Three blocks: **plan, check, learn**. The dashboard owns the check block.

| Time | What |
|---|---|
| 0–10 min | **Plan.** What enters Phase 1 next, what's blocked, what gets killed or respawned. |
| 10–15 min | **Check.** Read the dashboard. No discussion. |
| 15–25 min | Discuss any metric in watch or alert. One root cause hypothesis per metric. |
| 25–35 min | **Learn.** Retro any Request completed or killed since the last meeting. Decide one change. Owner assigned. |
| 35–45 min | Confirm: what's the change, who owns it, when it lands, how we'll measure if it worked. |

On a 30-minute day, compress proportionally — the Learn block is the one that must not be cut.

A single meeting with nothing to change is fine; say so explicitly and move on. But if a full week of these ends without one change owner-and-deadline, the cadence failed. Restart with a stricter format, or stretch the cadence until each meeting has real material.
