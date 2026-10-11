# Contributing to ADRIAN

ADRIAN is v0.2 — actively evolving. Contributions are welcome, and the contribution process uses ADRIAN's own methodology (the project eats its own cooking).

---

## Who maintains this

- **Author / Maintainer:** Jay (Jairo) Anaya
- **Origin & steward:** [Disruptica](https://disruptica.com) — ADRIAN was born at Disruptica and is the framework used internally on its products
- **Reference implementation:** [BugLady](https://buglady.ai) — where ADRIAN was proven, not a dependency of it

---

## What kinds of contributions are welcome

| Type | Examples | How to propose |
|---|---|---|
| Bug reports | A gate item that's unclear, a contradiction between docs, a broken link | GitHub issue |
| Clarifications | A section that needs an example, a term that needs defining | PR with the clarification |
| Local adaptations | "Here's how we made ADRIAN work for a 5-person team" | Discussion thread, then maybe a doc |
| New compliance mode | FedRAMP, GDPR-specific, ISO 27001 | Issue first to discuss scope, then PR |
| New language template | Rust, Java, Ruby `CLAUDE.md` template | PR directly |
| Methodology changes | A new phase, a new role, a new gate | Issue first — these are larger discussions |
| Threshold updates | "Our team's mutation score baseline was different" | Comment on `QUALITY-DASHBOARD.md` rather than changing canonical values |
| Tool-shaped assumptions | A doc that only works if you own a particular product — see the six requirements in [`TOOLING.md`](./TOOLING.md) | GitHub issue; these are treated as bugs |
| New tooling adapter | "Here's how ADRIAN maps onto Kiro / Spec Kit / Linear" | PR against [`TOOLING.md`](./TOOLING.md) |

---

## What's NOT a welcome contribution

- **Tool recommendations** in the canonical docs. ADRIAN is tool-agnostic. Tool recommendations belong in local team docs, not here.
- **Vendor promotion.** If your contribution is "use product X for step Y," it will be declined regardless of how good product X is.
- **Methodology renames.** ADRIAN is the name. If you want to fork and rename, do so under a different repo (and the MIT license permits this).
- **Removing the honest-framing language.** "ADRIAN is not an industry standard" stays in the docs. It's the disclosure that lets readers calibrate trust.

---

## How to propose a change — using ADRIAN's own process

For non-trivial changes (more than a typo or broken link), follow the Request model:

### 1. Open an issue first (Phase 1: Align)

Use the issue template. Include:

- **Summary** — what should change for users of ADRIAN, one sentence or short paragraph
- **Description** — the fuller narrative and context
- **User Impact** — who benefits and how much
- **Business Context** — why this matters to the framework now
- **Constraints** — anything that limits the change (backward compatibility, doc-only vs. structural)
- **Decisions** — what prior choices in ADRIAN does this respect or revisit
- **Acceptance** — how we'll know the change works
- **Verification** — how we'll know the change worked once merged

Yes, this is the Request Contract. ADRIAN changes go through ADRIAN.

### 2. Wait for issue review (Phase 1 gate)

The maintainer responds with one of:

- **Approved** — open a PR
- **Needs revision** — comments on the proposal
- **Declined** — with reasoning
- **Punted to discussion** — bigger conversation needed

Don't open a PR before the issue is approved. PRs without an approved issue will be asked to back up.

### 3. Open a PR (Phase 2: Implement)

The PR description must:

- Link the approved issue
- Summarize what changed
- Note any deviations from what was approved in the issue
- Include "AI-generated %" if AI helped draft the PR (be honest — there's no penalty)

### 4. Review (Phase 2 → Phase 3 gate)

The maintainer reviews on four dimensions:

- **Methodology fit** — does this respect ADRIAN's principles?
- **Honesty** — does this overstate what's been validated in practice?
- **Backward compatibility** — does this break existing users?
- **Documentation quality** — is it clear, terse, and load-bearing?

### 5. Merge and version bump (Phase 3: Deliver)

Minor changes go into a patch version. Substantive changes go into a minor version. Phase or role changes go into a major version.

### 6. Retrospective (Phase 4: Adapt)

Quarterly, the maintainer publishes a short retrospective: what changed, what worked, what didn't, what's next.

---

## Forking ADRIAN

If you want to take ADRIAN in a direction the maintainer won't, fork it. The MIT license allows it.

When you fork:

- **Rename.** Don't call your fork ADRIAN. Pick a new name. This avoids confusion and respects the work.
- **Keep the references.** `REFERENCES.md` is honest about what ADRIAN borrows. Your fork should be honest about what it borrows from ADRIAN.
- **Don't claim Disruptica origin.** ADRIAN was born at Disruptica; your fork is yours. Credit the source, don't impersonate it.

A graceful fork credits the source. An ungraceful fork erases it. Be graceful.

---

## Code of conduct

Treat contributors as engineers, not adversaries. Direct feedback is welcome; condescension and personal attacks are not.

If you wouldn't say it to a colleague in a code review at a job you wanted to keep, don't say it here.

---

## Getting in touch

- **Issues and PRs:** primary channel
- **Discussions:** for "should we…" questions before opening an issue
- **Email:** for sponsorship, partnerships, or anything that doesn't belong in public — Jay (Jairo) Anaya, via [Disruptica](https://disruptica.com)
- **BugLady-specific questions:** [buglady.ai](https://buglady.ai), not this repo
