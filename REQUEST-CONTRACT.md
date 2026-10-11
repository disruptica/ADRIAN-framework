# The Request Contract

The Request Contract is the artifact intake produces and the PM approves. It is the single most important artifact in ADRIAN. Everything downstream — code, tests, deploys, verification — derives from it.

**Where it lives:** in your requirements system, locked at approval and addressable by a stable link. Whether that system is inside or outside git is your choice, bounded by one rule: **customer PII never enters the repository.** A contract carrying a customer interview belongs outside git, with the PR linking back to it; a contract written internally, with no customer-identifying content, can live in the repo and diff alongside the code. Both topologies, and what ADRIAN requires of any requirements system, are in [`TOOLING.md`](./TOOLING.md).

Coding agents receive the contract and its origin as context at implementation time.

---

## How a Request Contract is born

1. A user, the PM, or the business raises a need — ideally in a conversation, in the application, where a requirements agent can interview them directly
2. Intake asks what a product manager would ask — the problem, the impact, the context — while analyzing the codebase, business goals, and compliance posture. A requirements agent does this in a chat; without one, the PM does it in a call or a thread
3. It ends in a structured Request Contract, **with the origin attached** so the dev team can see the raw context and the coding agent gets it verbatim
4. The PM pressure-tests, sets the priority, and approves

The origin is not decoration. Ambiguity that survives the structured sections is often resolved by reading what the user actually said.

---

## The sections — core and conditional

The contract is **flexible by request type**. Six sections are always required; the rest attach based on whether the Request is a bug, a feature, or a change.

### Core — required on every Request

| Section | Purpose |
|---|---|
| **Summary** | One sentence or short paragraph of what the user wants, in their words. This is the intent — if the team can't agree "yes, that's it," everything downstream is wrong |
| **Description** | The fuller narrative: what's happening, what should happen, relevant context from the origin |
| **User Impact** | Who is affected, how many, how badly. One of the two priority inputs |
| **Business Context** | Why this matters to the business: revenue, churn risk, compliance deadline, strategic bet. The other priority input |
| **Acceptance Criteria** | Scenarios that must pass — positive, negative, edge, UX states. Becomes the automated test suite |
| **Verification** | How we'll know it worked in production: metrics, observations, customer confirmation |

**Priority = User Impact × Business Context.** The PM scores both (a simple 1–5 each is enough); the product drives queue position. The queue reshuffles whenever reality changes — that's the squash game, and it's free because nothing in-flight mutates.

### Conditional — bug-shaped Requests

| Section | Purpose |
|---|---|
| **Steps to Reproduce** | Exact sequence that triggers the problem |
| **Expected Behavior** | What should have happened |
| **Actual Behavior** | What happened instead |
| **Environment** | Browser/OS/app version/tenant/config — whatever narrows the search |

### Conditional — feature-shaped and architectural Requests

| Section | Purpose |
|---|---|
| **Boundaries** | In scope / out of scope / deferred — stops scope creep and agent improvisation |
| **Constraints** | Compliance flag (none/SOC2/HIPAA/PCI/gov), perf budgets, data classification, architectural rules |
| **Decisions** | Prior choices this Request must respect (from ADRs, `CLAUDE.md`, adjacent Requests) — the drift killer |
| **Technical Notes** | Codebase analysis: suspected files, related modules, prior similar changes. Produced by the requirements agent where one runs, by Dev where none does. Advisory, not binding |

### Always attached

| Item | Purpose |
|---|---|
| **Origin** | What triggered this Request, in its rawest available form. Context for the dev team; verbatim input for the coding agent |

**Origin is required; its form depends on your intake.** Where intake is conversational, origin is the full interview transcript — that's the highest-value version, and the one coding agents benefit from most. Where it isn't, origin names the actual trigger: the support ticket, the incident, the customer email, the analytics query, the sales call note. What ADRIAN forbids is a contract with no traceable source, because that's a contract someone invented.

If the origin contains customer-identifying content and your contract lives in git, reference the origin rather than pasting it.

**The compliance flag rule still applies:** any Request touching PII, PHI, card data, or a regulated domain must carry Constraints with the flag set, regardless of request type. A bug fix in a HIPAA app is still HIPAA work.

---

## Worked example — bug-shaped Request

```markdown
# Request: Refund button spins forever on Safari

**Type:** Bug | **Priority:** UI 4 × BC 3 = 12 | **Compliance:** PCI

## Summary
CS agents on Safari can't complete refunds — the button spins forever
and they have to switch browsers mid-call with the customer.

## Description
Since Tuesday's deploy, clicking "Issue refund" on the order detail page
shows the loading spinner indefinitely on Safari 17.x. The refund
actually completes server-side, but the UI never confirms — agents retry
and create duplicate refund attempts (idempotency catches them, but the
audit log fills with rejected duplicates).

## Steps to Reproduce
1. Log in as CS agent on Safari 17.x
2. Open any refundable order
3. Click "Issue refund", confirm in modal

## Expected Behavior
Spinner resolves within ~2s, success toast, refund appears in history.

## Actual Behavior
Spinner never resolves. Refund IS created server-side. No toast.

## Environment
Safari 17.2–17.4 (macOS + iOS). Chrome/Firefox/Edge unaffected.
Prod tenant, all plans.

## User Impact
~40 CS agents; ~15% use Safari. Duplicate-attempt noise in the PCI
audit log. Customer-facing delay on live calls. Score: 4/5.

## Business Context
Refund flow is in PCI scope; audit log noise complicates the quarterly
review due in 3 weeks. Score: 3/5.

## Acceptance Criteria
Given a CS agent on Safari 17.x
When they complete a refund
Then the spinner resolves and the success toast shows within 3s
And no duplicate refund attempt is logged

Given the refund API responds slower than 10s
Then the UI shows the timeout error state with retry (all browsers)

## Verification
- Zero duplicate-attempt audit entries within 48h of deploy
- Manual check on real Safari (macOS + iOS) by QA
- Confirm with the CS team lead who reported it

## Technical Notes (codebase analysis)
The refund confirmation listens on an EventSource; Safari's EventSource
handling differs after the response-stream change in Tuesday's deploy
(PR #482). Suspect `refund-status-stream.ts`. Related: order-status uses
the same pattern and may be silently affected.

## Origin
[Attached — 14-message interview with the CS team lead]
```

A feature-shaped Request uses the same core six plus Boundaries, Constraints, Decisions, and Technical Notes instead of the reproduce/expected/actual/environment quartet.

---

## The kill-and-respawn doctrine

Requests are sized for **hours to days**, not weeks. That sizing is what makes ADRIAN's speed safe:

- **The contract is locked at approval.** No mid-flight mutation — mutation is how drift happens.
- **If priorities shift or the contract turns out wrong mid-flight: kill the Request and respawn a new one.** The new contract inherits the origin and whatever survived. Killing is cheap because the unit is small.
- **Killing is not failure.** ADRIAN rewards change; the respawn IS the system working. What gets tracked is *why*: killed-for-priority is neutral (the squash game), killed-for-contract-defect is a Phase 1 signal (see the Respawn Rate metric in [`QUALITY-DASHBOARD.md`](./QUALITY-DASHBOARD.md)).
- **If a Request can't be sized to days, it isn't one Request.** Intake splits it before approval — the agent proposes the split, the PM decides.

---

## How to prompt a requirements agent

If a requirements agent conducts your intake, this is the behavior to prompt it for. If the PM conducts intake instead, read it as the interview checklist — the content is the same, only the interviewer changes.

```
You are a product manager interviewing a user inside their application.
Ask questions the way a good PM would: what happened, what did you expect,
who is affected, why does it matter now. Analyze the codebase, business
goals, and compliance posture as you go.

The interview ends in a Request Contract. Always include: Summary (one
sentence or short paragraph of what the user wants, in their words),
Description, User Impact, Business Context, Acceptance Criteria, and
Verification. Attach the full interview.

If the request is bug-shaped, add: Steps to Reproduce, Expected Behavior,
Actual Behavior, Environment.
If it is feature-shaped or architectural, add: Boundaries, Constraints,
Decisions, and your Technical Notes from codebase analysis.
If it touches PII, PHI, card data, or a regulated domain, Constraints with
the compliance flag is mandatory regardless of type.

Acceptance Criteria must include positive, negative, and edge scenarios,
and UX states (loading/empty/error/success) for anything touching a UI.
Score User Impact and Business Context 1–5 each with one line of rationale;
the PM confirms or overrides.

If the request is too large to deliver in days, propose splitting it into
multiple Requests. If you cannot fill a required section from the
conversation, ask one more question rather than guessing.
```

---

## Contract quality red flags — PM checklist

Send the contract back for redraft (or re-interview) if any of these are true:

- [ ] Summary doesn't sound like something the user actually said or wants
- [ ] User Impact or Business Context is unscored or scored without rationale
- [ ] Acceptance Criteria has no negative or edge scenarios
- [ ] Acceptance Criteria has no UX states and the Request touches a UI
- [ ] Verification has no measurable production signal
- [ ] Bug-shaped but missing the reproduce/expected/actual/environment quartet
- [ ] Feature-shaped but Boundaries has no "out of scope" items
- [ ] Touches sensitive data but Constraints/compliance flag is missing
- [ ] Too big to deliver in days and not split
- [ ] The origin is missing, truncated, or untraceable
