# ADRIAN Methodology — Full Reference

**The ADRIAN loop: Align → Implement → Deliver → Adapt**

This is the canonical reference. All other documents in the set support or template this one.

---

## Premise

ADRIAN is built on three premises. If any of them is wrong in your context, ADRIAN is wrong for you.

1. **AI produces 85%+ of the code.** Humans steer, review key dimensions, and refactor — they do not type most of the code.
2. **A requirements bot (BugLady) drafts specs from in-app conversations.** Humans approve and pressure-test those specs.
3. **The team is self-managing.** Supervision is metrics and gates, not approval chains.

If you're at 60% AI generation, ADRIAN is overkill. If you don't have a requirements bot, the methodology still works but the PM does more drafting in Phase 1. If you have approval chains, the gate structure here will fight your org.

---

## The Request

The unit of work is a **Request**, not a story or a ticket.

A Request is:

- Something a user or the business actually asked for — born from a BugLady interview
- Small enough to deliver in **hours to days**, never weeks
- Specified completely (Request Contract) before implementation begins
- Prioritized by **User Impact × Business Context** — priority drives the queue, and the queue drives strategy
- Verifiable in production via a named metric or observation
- Killable: if priorities shift or the contract proves wrong mid-flight, the Request is killed and respawned, never mutated

If a change can't be sized to days, it isn't one Request — split it before approval.

### The squash doctrine

Software development used to be chess: plan far ahead, commit, execute over months. AI-first development is squash: the ball (user need) moves daily, and winning is reacting with quality at speed. ADRIAN's mechanics exist to make reacting cheap:

- **Reprioritizing the queue is free** — nothing in-flight mutates, so reshuffling what's next costs nothing
- **Killing an in-flight Request is cheap** — the unit is small, the contract and interview survive into the respawn
- **ADRIAN rewards change.** A killed-for-priority Request is the system working, not failing. What gets watched is killed-for-contract-defect — that's a Phase 1 quality signal (see the Respawn Rate metric)

---

## PHASE 1 — ALIGN

**Goal:** Produce an approved Request Contract that the rest of the process can trust.

**Roles:** BugLady drafts. PM owns. Dev and QA are consulted.

**Time share:** 15–30% of request time. This is the lever that buys back the 23.5% incident rate that 85%+ AI teams currently absorb. Underspending here is the most common ADRIAN failure mode.

### What BugLady produces

BugLady interviews the user (or PM, or business stakeholder) in a chat inside the application — asking questions the way a product manager would, while analyzing the codebase, business goals, and compliance posture. The interview ends in the **Request Contract**: six core sections always (Summary, Description, User Impact, Business Context, Acceptance Criteria, Verification), plus conditional sections by request type (bug-shaped: Steps to Reproduce, Expected/Actual Behavior, Environment; feature-shaped: Boundaries, Constraints, Decisions, Technical Notes), **with the interview itself attached**. Full template in [`REQUEST-CONTRACT.md`](./REQUEST-CONTRACT.md).

The contract lives in BugLady, not git. The interview is context for the dev team and verbatim input for the coding agents.

### What the PM does

The PM does not write the contract. The PM **pressure-tests and prioritizes** it. Five specific checks:

| Check | Question | Failure action |
|---|---|---|
| Wrong-request | Read the Summary out loud. Is this what the user actually asked for in the interview? | Reject. Re-interview or correct. |
| Priority | User Impact and Business Context scored 1–5 with rationale? Product sets queue position | Score it. Priority drives the queue; the queue drives strategy. |
| One-way door | Are any items in Decisions expensive to undo? | Flag for Dev consultation before approval. |
| UX state coverage | For every screen touched: loading, empty, error, success states in Acceptance Criteria? | Add them. Most UX bugs are missing states, not bad design. |
| Domain flag | What compliance mode applies? (none / SOC2 / HIPAA / PCI / gov) | Set the flag. Drives [`FLEX-MODES.md`](./FLEX-MODES.md). |

### What Dev and QA do in Phase 1

Dev: Consulted on Constraints and Decisions. Says yes or no to architectural items. Does not edit the contract.

QA: Reviews Acceptance for adversarial gaps (negative cases, edge cases, abuse cases). Adds scenarios. Begins drafting adversarial tests for Phase 2.

### Phase 1 gate: Request Approval

PM approves in BugLady. The Request Contract is now **locked** — it lives in the requirements tool (not git; the interview may contain customer PII that doesn't belong in a repository). The PR created in Phase 2 links back to the contract, and coding agents receive the contract plus the interview as context.

Locked means no mid-flight mutation. If the contract proves wrong or priorities change, the Request is **killed and respawned** with a corrected contract. This is what kills drift: 85%+ AI teams that mutate requirements mid-flight produce code that diverges from intent within two sprints.

---

## PHASE 2 — IMPLEMENT

**Goal:** AI produces code that satisfies the Request Contract. Humans steer and review the dimensions humans are better at.

**Roles:** Dev steers. AI agents build. PM is on-call for clarifications. QA writes adversarial tests in parallel.

**Time share:** 40–50% of request time.

### Dev's five-step loop

1. **Load context.** Repo conventions, the Request Contract (with its interview), the architectural decision record, the relevant `CLAUDE.md`. Context engineering is the senior skill — agent output is a function of context quality.
2. **Decompose into tasks.** Rule: a task is too big if the agent can't produce a passing test for it end-to-end.
3. **Run the agent loop.** Plan → generate → test → review → iterate. Run multiple agents in parallel for independent tasks.
4. **Review the four dimensions humans are better at:**
   - **Architectural fit** — Does this respect Decisions in the contract and conventions in `CLAUDE.md`?
   - **Security boundaries** — Auth, authz, data egress, secret handling, PII/PHI flow
   - **Failure modes** — Blast radius if a dependency is down. Idempotency. Retry behavior.
   - **Symbol footprint** — Search the codebase for every symbol the agent touched. If a callsite exists that the agent never opened, the task is not done. This is the highest-yield AI code review technique.
5. **Refactor aggressively.** AI output on day one is acceptable. AI output untouched at six months is the technical debt iceberg. Treat agent output as a first draft.

### What Dev does NOT do

- Read every line for style — the linter does that
- Hand-write tests the contract already implies — those auto-generate from Acceptance scenarios
- Approve their own PR without QA visibility

### Agent rules — encoded in `CLAUDE.md`

Every repo has a `CLAUDE.md` at the root. It encodes:

- Language idioms (TS strict mode, Python type hints required, Go error-wrapping conventions)
- Test framework and how to run tests
- "Never do" list specific to the stack
- File and folder conventions
- Secret handling rules

Templates for TS/Node, Python, Go are in [`CLAUDE-MD-TEMPLATES.md`](./CLAUDE-MD-TEMPLATES.md).

**Architectural drift fix: when drift shows up in review, the fix goes in `CLAUDE.md`, not in the reviewer's head.** This is the compounding loop that makes ADRIAN work over time.

### QA's parallel work

While the agent is building, QA is writing the tests that try to break it:

- Edge cases the contract missed
- Adversarial input (oversized, malformed, malicious)
- Abuse cases (authenticated user trying to access another tenant's data)
- Failure injection (what if this dependency times out mid-request)

These tests run in CI alongside the Acceptance tests.

### Phase 2 gate: PR Ready

All of:

- Every Acceptance scenario has an automated test that passes
- The PR description includes the agent's plan + diff summary
- Symbol footprint search is clean (no untouched callsites)
- Dev has signed off on the four review dimensions

---

## PHASE 3 — DELIVER

**Goal:** The change reaches production and we verify it actually worked.

**Roles:** CI/CD does most of the work. QA does the high-judgment parts. Dev is on standby for rollback.

**Time share:** 15–25% of request time.

### What the pipeline does automatically

Block merge if any of these fail. No human review required.

| Check | What it catches |
|---|---|
| Type check, lint, format | Style / type errors |
| Unit + integration tests from Acceptance | Functional regressions |
| Mutation testing on critical paths | "Tests pass but feature is wrong" — tests overfit to implementation |
| SAST + secret scan | Security baseline |
| SBOM / dependency check | Supply chain |
| Schema / contract diff | Breaking API changes |
| Architectural fitness functions | Import boundary violations, layering rules |
| AI-attribution tag | Marks PRs for 30-day quality tracking |

### Mutation testing — the lever for "wrong feature"

Mutation testing runs your test suite against deliberately-broken versions of the code. If a broken version still passes, your tests are theater.

For AI-generated code this is essential: agents over-fit tests to the implementation, producing tautological suites.

Tools by language:

- TS/Node: Stryker
- Python: mutmut, cosmic-ray
- Go: gremlins, go-mutesting

Target: 75%+ mutation score on critical paths. Don't bother running it on glue code.

### QA's deliver-phase work

Four things humans are uniquely good at:

1. **Adversarial test design** — already done in Phase 2, runs automatically in CI
2. **UX state verification** — manually walk every state in Acceptance on a real device. Cannot be cheaply automated. Should not be.
3. **Production verification** — after deploy, confirm the metrics in Verification actually moved. If the contract said "reduce support tickets" and tickets didn't move, the feature didn't work, regardless of green tests.
4. **Customer loop close** — talk to the customer who triggered the BugLady chat. This is the human connection the role exists for.

### Deploy strategy by domain

| Domain | Strategy |
|---|---|
| Internal tools, consumer experiments | Direct to prod, feature-flagged |
| B2B SaaS | Canary 5% → 50% → 100%, auto-rollback on error budget burn |
| Fintech, healthcare, gov | Canary + named human approver on prod promotion + signed deploy record |

### Phase 3 gate: Verified in Prod

Not "deployed." Verified. Defined as:

- Deploy completed without rollback
- Verification section metrics moved in the expected direction (or are confirmed unchanged where that was the goal)
- No SLO burn from the deploy
- Customer loop closed where applicable

---

## PHASE 4 — ADAPT

**Goal:** Lessons learned compound into two learners: the **team** (practices, contract template, gates) and the **agents** (`CLAUDE.md` rules, BugLady interview prompts, agent context).

**Roles:** Whole team. Every day or every other day. 30–45 minutes.

**Time share:** 5–10% of Request time, averaged.

This is where most teams fail. They ship, declare victory, move on. The compounding gains in agentic development come from this phase — and the agent half of it is the part humans forget. Every lesson that only lands in a human's head is lost to the next agent run; every lesson encoded in `CLAUDE.md` or a BugLady prompt improves every future Request automatically. Skip this phase and you become a faster way to produce average code.

### Two rituals

**1. Quality Review (30–45 min, sync, whole team, every day or every other day)**

One meeting doing three jobs — **plan, check, learn**. Traditional Agile splits these into planning, standup, and retro on a weekly loop; at AI request speed that loop is too slow to steer anything, so ADRIAN merges them and runs them on the request cadence. Cadence and format in [`ROLES.md`](./ROLES.md#the-team-cadence); timings in [`QUALITY-DASHBOARD.md`](./QUALITY-DASHBOARD.md).

*Plan* — what enters Phase 1 next, what's blocked, what gets killed or respawned and why.

*Check* — five numbers. Details in [`QUALITY-DASHBOARD.md`](./QUALITY-DASHBOARD.md):

- Respawn rate (split by cause: priority vs. contract defect)
- First-time CI pass rate for AI PRs
- Mutation score on critical paths
- 30-day rework rate on AI-touched code
- Change failure rate

*Learn* — retro on every Request that completed or was killed since the last meeting. Five questions:

- Did the contract hold, or was the Request killed/respawned? Why — priority shift (neutral) or contract defect (Phase 1 signal)?
- Did the agent get stuck? Where? Why?
- What's now in `CLAUDE.md` that wasn't before this Request?
- Does BugLady's interview prompt need updating (a question it should have asked)?
- Tag the Request: `clean` / `churn` / `incident`

Every meeting ends with a change to something — `CLAUDE.md`, the BugLady interview prompt, the Request Contract template, the gates, or the dashboard thresholds — or an explicit decision that nothing needs changing. One quiet meeting is fine. **A week of Quality Reviews that produced no change is a failed week.**

**2. Monthly Tech Radar (45 min, team)**

What did we learn? What goes into the team's adopt / trial / hold / drop list? When did the agent surprise us — good or bad? What should we change in the framework itself?

### What gets captured per PR

Tag every PR with:

- AI-generation percentage (rough estimate is fine)
- Request Contract it implements (BugLady link)
- Mutation score on touched files
- Domain (compliance flag from the contract)
- Reviewer

Without this attribution, the 30-day rework rate metric is meaningless.

---

## How the four phases relate to existing frameworks

If your team is coming from Scrum, Kanban, or Shape Up:

| ADRIAN | Scrum equivalent | Kanban equivalent | Shape Up equivalent |
|---|---|---|---|
| Request | Story (sort of) | Card | Pitch (smaller) |
| Align (Phase 1) | Refinement / Planning | Card definition | Shaping |
| Implement (Phase 2) | Sprint execution | In-progress | Building |
| Deliver (Phase 3) | Sprint review / Done | Done | Cool-down validation |
| Adapt (Phase 4) | Retro | Service review | Cool-down |
| Quality Review | Planning + standup + retro, merged | Replenishment + cadence review | Betting table (lighter) |

ADRIAN does not have sprints. Phases run per-request, not per-team-cadence. Multiple requests are in flight at once at different phases. The one recurring meeting — the Quality Review — replaces Scrum's planning, standup, and retro with a single 30–45 minute block every day or every other day.

---

## What makes ADRIAN fail

Honest invalidators. If you see any of these, stop and fix before continuing.

1. **BugLady's contract quality plateaus low.** The Request Contract is load-bearing. If BugLady output is <80% as good as your best human PM on equivalent work, fix BugLady before scaling the methodology.
2. **`CLAUDE.md` is treated as documentation instead of executable rules.** Audit by asking the agent to deliberately violate a rule. If it does, the rules aren't strong enough.
3. **Regulated work gets the "named reviewer" requirement dropped under deadline pressure.** Don't. The audit trail is non-negotiable. See [`FLEX-MODES.md`](./FLEX-MODES.md).
4. **The Quality Review becomes status.** It exists to plan, check, and change something. If it degrades into a round-robin of yesterday's work, kill it and replace it.
5. **Mutation testing gets disabled because it's slow.** Move it to nightly + critical-path-only in CI rather than disabling. The signal is too valuable.

---

## Versioning

This is ADRIAN v0.1.

Major versions change the phase structure or the role definitions.
Minor versions change templates, thresholds, or gate checklists.

Every team running ADRIAN should version-pin and review the upstream every quarter.
