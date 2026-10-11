# Phase Gate Checklists

Each gate is a hard transition between phases. **A request cannot move forward unless every item is true.** The gates are the supervision in a self-managing team — if you weaken them, you remove the safety rail.

Print these. Use them. They're terse on purpose.

---

## Gate 1 → Request Approval (Align → Implement)

**Owner:** PM. Approves in BugLady.

**Cannot proceed if any answer is "no":**

- [ ] Intent is one sentence in customer language
- [ ] Boundaries lists in scope, out of scope, deferred
- [ ] Compliance flag is set (one of: none, SOC2, HIPAA, PCI, gov)
- [ ] Data classification covers all user-facing data in the request
- [ ] Decisions section references prior ADRs / specs where relevant
- [ ] Acceptance has positive, negative, edge cases
- [ ] If the request touches a UI: Acceptance has loading, empty, error, success UX states
- [ ] If multi-tenant, money, PII, PHI, or credentials are touched: Acceptance has abuse/security scenarios
- [ ] Verification names at least one measurable production signal
- [ ] Rollback strategy is documented if compliance flag ≠ none
- [ ] Dev consulted on architectural items in Decisions
- [ ] QA consulted on adversarial gaps in Acceptance

**On gate pass:** The Request Contract is locked in BugLady (it never goes to git — the interview may contain customer PII). The Request enters the priority queue. Mid-flight change means kill-and-respawn, never mutation.

---

## Gate 2 → PR Ready (Implement → Deliver)

**Owner:** Dev. Signs in the PR description.

**Cannot proceed if any answer is "no":**

- [ ] Every Acceptance scenario has an automated test
- [ ] All automated tests pass
- [ ] Mutation score on critical paths ≥75% (or critical paths explicitly marked as N/A)
- [ ] PR description includes the agent's plan and a diff summary
- [ ] Symbol footprint search is clean — for every symbol the agent touched, all callsites were either opened or confirmed unaffected
- [ ] `CLAUDE.md` rules respected (linter passes, fitness functions pass)
- [ ] Dev signoff on: architectural fit, security boundaries, failure modes, symbol footprint
- [ ] QA's adversarial tests included and passing
- [ ] The contract was not mutated mid-flight (if it proved wrong, the Request was killed and respawned)
- [ ] AI-attribution tag set on the PR (rough % is fine)

**On gate pass:** PR enters automated pipeline.

---

## Gate 3 → Verified in Prod (Deliver → Adapt)

**Owner:** QA / Success. Signs in the retro block of the next Quality Review.

**Cannot proceed if any answer is "no":**

- [ ] Deploy completed without rollback
- [ ] All pipeline checks passed (types, lint, tests, mutation, SAST, secrets, SBOM, contract diff, fitness functions)
- [ ] Manual UX state walkthrough completed on real devices
- [ ] Verification production signals checked and moved in expected direction (or confirmed unchanged where that was the goal)
- [ ] No SLO burn from this deploy
- [ ] Customer loop closed: original BugLady-chat customer confirmed the flow works for their case
- [ ] If compliance flag was PCI/HIPAA/gov: named human approver signed the prod promotion
- [ ] If compliance flag was SOC2: change attribution recorded for audit log

**On gate pass:** Request is "done." Goes into the retro block of the next Quality Review (Phase 4).

---

## Compliance overlay on the gates

Some gate items are conditional on the compliance flag. The full mapping is in [`FLEX-MODES.md`](./FLEX-MODES.md). Quick reference:

| Compliance flag | Additional gate items |
|---|---|
| `none` | None beyond the base checklists |
| `SOC2` | Gate 2: change attribution logged. Gate 3: deploy attribution recorded. |
| `HIPAA` | Gate 1: PHI fields explicitly tagged. Gate 2: named reviewer on PR. Gate 3: BAA-compliant infrastructure verified. |
| `PCI` | Gate 1: scope confirmation. Gate 2: named reviewer on PR + secrets review. Gate 3: named approver on prod. |
| `gov` | Gate 1: data residency confirmed. Gate 2: named reviewer. Gate 3: named approver + signed deploy record. |

---

## When a gate fails

The request does not "fail." It moves back to the prior phase with a documented reason.

**Gate 1 failure (Request Approval):**
- Send the contract back to BugLady with the PM's notes (or re-interview)
- If BugLady keeps failing in the same way, that's a Phase 4 input — adjust BugLady's prompt or the Request Contract template

**Gate 2 failure (PR Ready):**
- Most failures here are missing tests or symbol-footprint gaps — these are quick fixes
- If the failure is "agent cannot satisfy the contract," that is a Phase 1 failure surfaced late — kill and respawn with a corrected contract, do not paper over it
- If mutation score is low, the agent wrote tautological tests — pair with QA on adversarial cases

**Gate 3 failure (Verified in Prod):**
- If Verification signals didn't move: the feature is technically done but didn't achieve the Intent. This is the most important failure to surface. Do NOT close the request. Treat it as a new Phase 1 input.
- If a regulated-mode signoff is missing: the deploy should not have happened. Roll back, fix the process, re-deploy with proper signoff. This is a process violation worth a Quality Review item.

---

## Why these gates exist

Each gate exists because of a specific failure mode that AI-first teams currently absorb:

| Gate | Failure mode it prevents |
|---|---|
| Request Approval | Building the wrong request; contract drift; missing compliance handling |
| PR Ready | Architectural drift; security gaps; tautological tests; "agent missed a callsite" bugs |
| Verified in Prod | "Deployed but didn't actually solve the customer's problem"; audit-trail gaps in regulated work |

Weakening a gate trades a known failure mode for short-term speed. Sometimes that trade is worth it (early-stage, internal tools, low-risk experiments). It is never worth it for regulated work.

---

## Gates are not approval chains

The team is self-managing. Gates are owned by team members, not external approvers.

- The PM signs Gate 1 — but the PM is on the team, not above it.
- The Dev signs Gate 2 — for their own PRs.
- The QA signs Gate 3 — for their own verification.

The "named human approver" requirement in regulated modes is the one exception, and it can usually be filled by another team member with the appropriate role designation (e.g., a Dev on a different request signs as the named PCI reviewer).

If your gates require external approval, you don't have a self-managing team. You have a bottleneck.
