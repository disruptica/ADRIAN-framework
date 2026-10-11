# Roles in ADRIAN

Three human roles. Each one is redefined from its traditional version. Read the redefinition carefully — the names are familiar, the work is not.

---

## Why only three roles

Because the AI does the volume work. Adding roles to oversee AI output just creates a new bottleneck. The three roles map to the three things humans are irreplaceable at:

- **Intent** — what we're building and why (PM)
- **Judgment** — architecture, security, failure modes (Dev)
- **Validation** — does it actually work for the user (QA / Success)

If you find yourself wanting a fourth role, the question to ask first is: *what `CLAUDE.md` rule or pipeline check would eliminate the need for that role?*

---

## Product Manager (PM)

**Old definition:** Writes specs, prioritizes backlog, manages stakeholders.

**ADRIAN definition:** Owns intent and the request contract. Pressure-tests drafted contracts — or drafts them, if the team runs no requirements agent. Owns the customer relationship. Force-multiplier, not bottleneck.

### Per-phase responsibilities

**Phase 1 — Align**
- Review every Request Contract before approval — every one, whether an agent drafted it or you did
- Run the four checks: wrong-feature, one-way door, UX state coverage, domain flag
- Approve or reject. Rejection sends the contract back for redraft or to a follow-up customer conversation
- Approve in the requirements system — the contract is now locked; the PR will link back to it

**Phase 2 — Implement**
- On-call for Dev or agent questions about Intent or Boundaries
- Decide kill-and-respawn when Dev surfaces a contract defect or priorities shift — the contract never mutates mid-flight
- Do not review code. That's not your job.

**Phase 3 — Deliver**
- Confirm Verification metrics moved post-deploy
- Co-own the customer loop close with QA

**Phase 4 — Adapt**
- Bring defect-respawn patterns to the Quality Review
- Update the Request Contract template when patterns repeat

### What the PM does NOT do

- Type specs by hand, where a requirements agent can draft them (review beats transcription)
- Manage developer assignments (the team self-manages)
- Review code (Dev does)
- Run standups (ADRIAN has no standups — the Quality Review is a decision meeting, not status)
- Translate between customer and engineer in real time (the contract and its origin are the bridge)

### When the PM should escalate

- Drafted contracts consistently need >30% revision
- Multiple requests touch the same area in conflicting ways
- A customer asks for something that crosses compliance modes

---

## Developer (Dev)

**Old definition:** Writes code.

**ADRIAN definition:** Steers agents. Owns architecture and security boundaries. Refactors AI output. Owns `CLAUDE.md` and agent context. Quality of code is now a function of quality of context.

### Per-phase responsibilities

**Phase 1 — Align**
- Consulted on Constraints and Decisions
- Veto power on architectural items in the contract
- Flag one-way-door decisions before approval

**Phase 2 — Implement**
- Run the five-step agent loop (load context → decompose → run agents → review four dimensions → refactor)
- Maintain `CLAUDE.md` as agent behavior reveals gaps
- Pair with QA on adversarial test scenarios

**Phase 3 — Deliver**
- On standby for CI failures and rollback
- Sign off on architectural fitness function changes when needed
- Named approver on regulated-mode deploys (see [`FLEX-MODES.md`](./FLEX-MODES.md))

**Phase 4 — Adapt**
- Bring agent-stuck patterns to Quality Review
- Propose `CLAUDE.md` changes
- Audit `CLAUDE.md` adherence (ask the agent to deliberately violate a rule)

### What the Dev does NOT do

- Type most of the code
- Read every line of agent output for style (linter does that)
- Write tests the Acceptance section already implies (those auto-generate)
- Approve their own PR without QA visibility on adversarial tests

### When the Dev should escalate

- An agent is consistently failing on the same kind of task — investigate before adding more rules
- A contract passed Phase 1 but reveals an architectural conflict in Phase 2 (this is a Phase 1 failure, surface it)
- Security boundary violation that the rules didn't catch
- Mutation score dropping on critical paths

---

## QA / Success

The role name is two words on purpose. **QA without Success becomes a gatekeeper. Success without QA becomes a cheerleader.** The role is both.

**Old definition:** Tests features at the end. Files bugs.

**ADRIAN definition:** Adversarial test designer, UX state verifier, production outcome validator, customer voice. Quality engineer plus customer success in one role.

### Per-phase responsibilities

**Phase 1 — Align**
- Review Acceptance for adversarial gaps
- Add negative cases, edge cases, abuse cases to the Acceptance Criteria
- Begin drafting adversarial test scenarios for Phase 2

**Phase 2 — Implement**
- Write adversarial tests in parallel with the agent's work — these are tests the implementer would not naturally think to write
- Tests run in CI alongside Acceptance tests
- Available to Dev for "could we break it this way?" pairing

**Phase 3 — Deliver**
- Manual UX state verification on real devices (loading, empty, error, success for every state in Acceptance)
- Production verification — confirm Verification metrics moved
- Close the customer loop — talk to the customer named in the contract's Origin

**Phase 4 — Adapt**
- Bring "tests passed but feature was wrong" patterns to Quality Review
- Bring UX state gaps to the Request Contract template
- Track customer satisfaction signals over time

### What QA / Success does NOT do

- Wait for the feature to be "done" before engaging — they engage in Phase 1
- Manually test what tests can automate
- Sign off on functional correctness alone — they sign off on customer outcome

### When QA should escalate

- Production verification fails (deployed, but Verification metric didn't move)
- A pattern of "tests pass but customers report bug" — this is a contract quality problem, not a test problem
- UX state coverage is consistently incomplete in contract drafts

---

## Self-managing — what it means here

The team does not need approval from outside the team to:

- Move a request through the four phases
- Kill and respawn a Request (with the cause tagged: priority vs. defect)
- Choose tools, models, or agents
- Refactor AI output
- Roll back a deploy

The team **does** escalate when:

- A request touches another team's domain
- Compliance flag is unclear
- A one-way-door decision wasn't in the contract's Decisions section
- Quality Review numbers trend wrong for two consecutive weeks

**Self-managing ≠ unsupervised.** Supervision is the metrics and the gates, not a manager-in-the-loop.

---

## The team cadence

One meeting. Every day or every other day. 30–45 minutes. Whole team. In that single meeting the team does **planning, the daily check, and the retro**.

Requests move fast enough that a weekly loop is too slow to steer them — at AI speed, the request you would have corrected on Tuesday has already shipped by Friday. So ADRIAN collapses the three Agile ceremonies into one short meeting running on the request cadence instead of a calendar cadence. Same three jobs — **plan, check, learn** — just at the speed the requests actually move.

### The three blocks

| Block | What |
|---|---|
| **Plan** | What enters Phase 1 next, what's blocked, what gets killed or respawned (with the cause tagged) |
| **Check** | The five dashboard numbers. Anything in watch or alert gets one root-cause hypothesis |
| **Learn** | Retro on every Request that completed or was killed since the last meeting — the five retro questions (see [`METHODOLOGY.md`](./METHODOLOGY.md#phase-4--adapt)) |

On a 30-minute day the blocks compress. The Learn block is the one that must not be cut. Full agenda and timings in [`QUALITY-DASHBOARD.md`](./QUALITY-DASHBOARD.md).

Each role brings something specific: the PM brings contract defects and customer signal, the Dev brings `CLAUDE.md` changes and agent-stuck patterns, QA brings adversarial-test and UX-verification findings.

### Honest note on where we actually are

We haven't fully landed this ourselves. Our own team still runs the old shape underneath: Monday plans most of the week, Friday carries the bigger retro, and the meetings in between are shorter. We suspect we'll transition to a true daily cadence soon — the pull toward it is real — but we haven't done it yet, so treat the daily version as the direction, not as battle-tested practice.

### What this is not

It is not a standup. Status lives on the request board, not in a round-robin of what everyone did yesterday. If the meeting turns into status reporting, it has failed — cut it and restart with the three blocks above.

This is a sketch, not a prescription. Adjust to your request cadence.

---

## On hiring for ADRIAN

This isn't strictly a methodology concern, but the role redefinitions change what to hire for.

**PM:** Hire for customer judgment and contract discipline, not for documentation throughput. The bot does the volume.

**Dev:** Hire for systems thinking, security instincts, refactoring taste, and the ability to write good `CLAUDE.md` content. Hire less for raw coding speed. The 10x typist is a 1x agentic developer.

**QA / Success:** Hire for adversarial thinking, customer empathy, and willingness to talk to humans. The "I just write Selenium scripts" QA archetype is wrong for this role.

Single-skill specialists in the old definitions of these roles will struggle in ADRIAN.
