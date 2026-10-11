# Adoption Plan

How to roll out ADRIAN on an existing team without breaking what's already working.

**Total time to fully adopt:** 8 weeks for a single team. Add 2 weeks per additional team if rolling out across multiple.

**Pre-condition:** A single team that already ships code together. Don't reorganize and adopt simultaneously — pick one disruption at a time.

---

## The honest version of this plan

You will not get this clean. Real adoption is messy. Use this as a target, not a contract. Expect:

- Week 3 will reveal that your drafted contracts are worse than you thought
- Week 5 will reveal that one Dev hates writing `CLAUDE.md` and you'll need to fix incentives
- Week 6 will reveal a quality issue the dashboard caught that you'd have shipped a month ago
- Week 8 will not be "we're done." It will be "we're in steady state and now we iterate."

The plan is a structure for those discoveries, not a schedule that has to be hit.

---

## Week 0 — Decide and prep

Before week 1, the team lead does four things alone:

1. **Pick one team** to be the pilot. Not the most overloaded. Not the most strategic. Pick the team that ships consistently and has a healthy retrospective culture.
2. **Pick one app** as the pilot codebase. Ideally not the most critical, not the least critical — mid-stakes is best for learning.
3. **Confirm the compliance flag** for that app. If it's regulated, the rollout has more steps (named reviewers, audit trails). If it's unregulated, you can move faster.
4. **Check your requirements system against the six requirements** in [`TOOLING.md`](./TOOLING.md). You probably already own something that qualifies — an issue tracker with a contract template usually does. What you're looking for is the gaps: can it record an approval, can it keep a contract from changing silently afterwards, can it hand a coding agent plain text. Fix those with a template and a convention before week 2, not with a purchase.

Tell the team in week 0 that ADRIAN is coming, share the README, and get questions. Don't try to sell it as "the answer." Sell it as "version 0.1 of an open source framework born at Disruptica that we evolve together."

---

## Week 1 — Foundations

**Goal:** Set up the artifacts. No methodology change yet.

### What to do

- Commit `CLAUDE.md` to the pilot repo, drafted from the relevant template
- Add the Request Contract template to wherever requests get written — an issue template, a spec file, or your requirements agent's prompt
- Set up AI-attribution tagging on PRs (a label is fine; automated detection is better)
- Add mutation testing to CI on the critical paths only (don't try to cover everything yet)
- Set up the architectural fitness function for import boundaries
- Start the Quality Dashboard with three metrics — change failure rate, first-time CI pass, and mutation score. Add the others later.

### What to NOT do

- Do not change the requirements process yet
- Do not change role responsibilities yet
- Do not start using gate checklists yet

### What success looks like at end of week 1

- `CLAUDE.md` exists and is being read by agents
- One mutation test run completed, baseline number recorded
- Dashboard has one week of baseline data

---

## Week 2 — Phase 1 (Align) only

**Goal:** Start using the Request Contract for new work. Don't touch existing work.

### What to do

- Start every new request with the six-section Request Contract
- Your requirements agent drafts and the PM pressure-tests using the four checks. No agent? The PM drafts and Dev or QA pressure-tests instead — never let the author be the only reviewer
- Apply Gate 1 checklist (Request Approval) before any new Request enters implementation
- Existing in-flight work continues unchanged

### What to NOT do

- Don't try to retrofit existing tickets into the new format
- Don't slow Phase 2/3 with new gates yet

### What success looks like at end of week 2

- At least 3 requests have a complete Request Contract
- PM has rejected at least one contract for redraft (if not, the bar is too low)
- Team agrees: "this contract is clearer than what we used to start with"

### Common week 2 failures

- Drafts arrive missing 2–3 sections. Fix the intake prompt or the template; don't backfill manually.
- PM rubber-stamps specs to keep velocity. Watch for this in the retros — if no specs were rejected, the PM either has perfect input or isn't pressure-testing.
- Devs say "this is more process." Show them the defect-respawn data after two weeks — contracts born from a real interview fail far less often than the vague tickets they replace.

---

## Week 3 — Phase 2 (Implement) discipline

**Goal:** Devs adopt the five-step agent loop. `CLAUDE.md` becomes a living document.

### What to do

- Devs run the explicit five-step loop on every new request (load context → decompose → run agents → review four dimensions → refactor)
- Symbol footprint search becomes a hard check on every PR
- When drift shows up in review, the fix goes into `CLAUDE.md` — track this. Count `CLAUDE.md` updates per week as a leading indicator of methodology health.
- Apply Gate 2 checklist (PR Ready) before merging

### What success looks like at end of week 3

- Every new PR has the symbol footprint check done
- `CLAUDE.md` has been updated at least 3 times based on review findings
- At least one PR has been sent back due to symbol footprint miss (if zero, the check isn't being run honestly)

### Common week 3 failures

- Symbol footprint check is skipped under deadline. Make it part of the PR description template so it's visible.
- `CLAUDE.md` updates are not landing. Either the team isn't surfacing patterns in review, or no one feels ownership. Assign rotating ownership of `CLAUDE.md` if needed.

---

## Week 4 — Phase 3 (Deliver) and QA / Success role shift

**Goal:** QA shifts from end-of-flow gatekeeper to parallel adversarial test designer.

### What to do

- QA writes adversarial tests in Phase 2, in parallel with the agent
- QA does manual UX state verification on requests entering Phase 3
- QA closes the customer loop on completed requests
- Apply Gate 3 checklist (Verified in Prod)
- Set up Verification metric tracking — if Verification signals can't be measured, escalate to fix observability gaps

### What success looks like at end of week 4

- QA has caught at least one bug in Phase 2 that would have shipped under the old workflow
- At least one request has had its Verification signals measured post-deploy
- At least one customer loop close has happened (real customer talked to, not just a confirmation message)

### Common week 4 failures

- QA struggles with the parallel work model — old habits are hard to break. Pair QA with Dev on the first 2–3 requests.
- Verification signals aren't measurable for some requests. Don't lower the bar; fix the observability instead. This is the moment to invest in your observability platform if it's weak.

---

## Week 5 — Add Phase 4 rituals

**Goal:** The Quality Review becomes habit — plan, check, and learn in one short meeting.

### What to do

- Start the Quality Review: 30–45 min, whole team, every other day to begin with (see [`ROLES.md`](./ROLES.md#the-team-cadence))
- Run all three blocks from day one — plan, check, learn. The retro block covers every request completed or killed since the last meeting
- Tighten toward daily once the meeting consistently has material; stretch it if it doesn't
- Add the remaining two metrics to the dashboard (respawn rate with defect split, 30-day rework rate)
- The first Monthly Tech Radar happens at end of month 1 (week 4 or 5 depending on calendar)

### What success looks like at end of week 5

- The Quality Review produced at least one concrete change this week (a `CLAUDE.md` update, a Request Contract change, or a gate adjustment)
- Every request that completed or was killed got its five retro questions answered in a Quality Review
- The dashboard has 4 weeks of data — trends are visible

### Common week 5 failures

- The Quality Review becomes a status meeting — a round-robin of yesterday's work. Cancel it for a week and restart with a stricter format. If it doesn't produce change, it failed.
- The retro block gets dropped whenever the meeting runs long. It's the block that must not be cut. Make it a Gate 3 item — "retro questions answered in a Quality Review" is on the checklist.
- The team goes daily before it has daily-sized material, and the meeting empties out. Every other day is the safe starting cadence.

---

## Week 6 — Flex Mode (if applicable)

**Goal:** If you have regulated work, layer in the Flex Mode additions.

### What to do

- If the pilot app has a compliance flag of `SOC2`, `HIPAA`, `PCI`, or `gov`: implement the additions per [`FLEX-MODES.md`](./FLEX-MODES.md)
- Identify named reviewers for each regulated mode
- Add the compliance-specific `CLAUDE.md` sections
- Verify audit trails are operational

### What success looks like at end of week 6

- A regulated-mode request has gone through the full process including named reviewer signoff
- Audit artifacts are retrievable
- If unregulated only: skip to week 7

### Common week 6 failures

- Named reviewers become a bottleneck. Mitigate by cross-training and rotating the role.
- Audit trail gaps surface. Fix immediately — these are the items that bite under audit pressure later.

---

## Week 7 — Tune

**Goal:** Adjust thresholds and rules based on real data.

### What to do

- Review six weeks of dashboard data
- Adjust thresholds where the published defaults don't fit your team
- Review `CLAUDE.md` — is it 100 lines? 500 lines? Either is fine, but the agent must still respect every rule
- Audit `CLAUDE.md`: pick three rules, ask the agent to violate them, fix the ones it violates
- Review the Request Contract — are any sections consistently empty or rubber-stamped? Either tighten the template or simplify

### What success looks like at end of week 7

- Thresholds reflect your team's reality, not the published defaults
- The `CLAUDE.md` audit produced at least one rule update
- Request Contract template has been adjusted at least once

---

## Week 8 — Steady state declared (and what comes next)

**Goal:** The team can run ADRIAN without referring back to this documentation constantly.

### What to do

- Run ADRIAN for the full week without coaching
- Conduct the second Monthly Tech Radar — review the methodology itself, not just the work
- Document the team's local adaptations in an `ADRIAN-LOCAL.md` file in the repo

### What success looks like at end of week 8

- Devs, PM, QA all describe their roles in terms of the four phases naturally
- The dashboard is updated without being asked
- At least one local adaptation has been made (e.g., the team added a sixth metric, or removed a section that didn't fit)

### What "done" means

It doesn't. ADRIAN v0.2 is the starting point. The team owns the methodology now. Quarterly:

- Review the methodology itself in the Monthly Tech Radar
- Promote local adaptations to the canonical doc
- Increment the version

---

## Rolling out to multiple teams

After the pilot succeeds, add teams one at a time. Each new team:

- Gets a "graduate" from the pilot team for the first month as an informal coach
- Reuses the pilot team's `CLAUDE.md` templates as starting points (adjusted for their domain)
- Starts at week 1 — does NOT skip ahead
- Adds 2 weeks to the rollout if their app has a different compliance flag

Resist the temptation to roll out to multiple teams in parallel before the pilot has hit steady state. Parallel rollout multiplies coordination cost and means none of the teams get coaching attention when they hit the predictable Week 3 / Week 5 failures.

---

## Common adoption anti-patterns

These will happen. Watch for them.

### "We'll do it properly later"
A Dev skips the symbol footprint check because they're "almost done." A PM rubber-stamps a contract because the customer is waiting. QA skips the customer loop because "we'll catch it in the next request."

These are the same pattern: trading methodology rigor for short-term speed. Every instance compounds. Surface them in the Quality Review.

### Methodology theater
The team runs Quality Reviews without changing anything. Updates the dashboard without acting on it. Names compliance reviewers who don't actually review.

This is worse than skipping the methodology — it produces evidence of process for audits without producing actual quality. Detect via the "what changed this week?" question. If the answer is nothing, the rituals are theater.

### Tooling over process
Someone proposes a new tool to "solve" a methodology problem. Maybe the defect respawn rate is high — instead of fixing the intake prompt, they propose a new requirements tool.

Tools are usually the wrong answer in ADRIAN. The methodology problems are upstream of tools. Default to fixing the process; only reach for tooling when the same problem persists after two process iterations.

### Compliance flag inflation
Every request gets flagged HIPAA "just in case." This burns the methodology by adding regulated-mode overhead to unregulated work.

The compliance flag should be the strictest mode that actually applies, not the strictest mode you can imagine applying. Push back on inflation in contract review.

### Compliance flag deflation
A request that should be HIPAA gets flagged `none` because no one wants the overhead.

Worse than inflation. Detected only by audit (potentially years later, by then very expensive to fix). Push back hard in contract review on anything that looks like it touches sensitive data with a `none` flag.

---

## What to stop doing

The point of ADRIAN is to do less of some things, not more. Things to stop:

- Stop daily standups. The request board is the status. The daily sync you keep is the Quality Review — a plan/check/learn decision meeting, not a round-robin.
- Stop manual story estimation. Requests aren't estimated; they're sized to one request = one independently shippable change.
- Stop separate sprint planning, standup, and retro meetings. They collapse into one 30–45 minute Quality Review. The unit of planning is a request, not a sprint.
- Stop hand-writing tests that the Acceptance Criteria already imply. Generate them from the contract.
- Stop reviewing code style in PRs. The linter does that. Reviewers focus on the four dimensions.
- Stop writing tickets that are vaguer than the Request Contract. If the ticket can't be a Request, it's not ready.

The methodology should remove work, not add it. If it's net-adding work after 8 weeks, audit what you've kept from the old process that should have been retired.

---

## Rollback plan

If ADRIAN doesn't work for your team after 8 weeks of honest adoption:

1. Do a Monthly Tech Radar specifically on the methodology
2. Identify which phases or rituals are producing value and which aren't
3. Keep what works; revert what doesn't
4. Don't go back to the prior process wholesale — that throws out the learning

The most likely partial-success scenario is: request contract works, gates work, dashboard works, but Phase 4 rituals don't stick. That's fine. Run ADRIAN without the rituals and add them back later.

The least likely partial-success scenario is: the methodology works but your requirements agent doesn't produce good enough contracts. In that case the PM does more drafting and the agent becomes assistive rather than primary. ADRIAN still works; the human cost is higher.
