# Tooling — what ADRIAN requires, and what satisfies it

ADRIAN is tool-independent. It needs a **requirements system**, and it works better with a **requirements agent**, but it does not care which ones you use.

This document defines the interface. If your tooling meets the six requirements below, you can run ADRIAN on it. Everything else in this repo is written against that interface, not against any product.

---

## Two terms used throughout the docs

| Term | What it is | Required? |
|---|---|---|
| **Requirements agent** | The AI that conducts intake — interviewing a user, analyzing the codebase, drafting the Request Contract | **No.** Without one, the PM conducts the interview and drafts the contract. ADRIAN still works; Phase 1 just costs more human time |
| **Requirements system** | Where the approved Request Contract lives, locks, and is addressed from | **Yes.** Something must hold the contract and make post-approval change impossible or visible |

Often they're the same product. They don't have to be. A team can run a conversational agent for intake and keep approved contracts in an issue tracker, or have no agent at all and keep PM-drafted contracts in a spec directory.

---

## The six requirements

Any requirements system running ADRIAN must:

1. **Produce the six core contract sections** — Summary, Description, User Impact, Business Context, Acceptance Criteria, Verification — plus the conditional sections for the request type. Template in [`REQUEST-CONTRACT.md`](./REQUEST-CONTRACT.md).
2. **Record an approval event** — who approved it, when. The Phase 1 gate in [`PHASE-GATES.md`](./PHASE-GATES.md) is unenforceable without one.
3. **Lock the contract at approval, or make post-approval change visible.** Kill-and-respawn is ADRIAN's drift defense, and it only works if mutation is either impossible or obvious. A tool where anyone can silently edit an approved contract fails this requirement — if that's your tool, enforce the lock by convention and watch the respawn rate closely.
4. **Give the contract a stable address** — a link or ID the PR can reference for as long as the code lives.
5. **Carry origin traceability** — a record of what triggered the Request: the interview, ticket, incident, customer email, or analytics query. See the Origin section in [`REQUEST-CONTRACT.md`](./REQUEST-CONTRACT.md).
6. **Export the contract and its origin as plain text** — coding agents consume both as context at implementation time. A tool that can only render content in its own UI fails this requirement.

One constraint binds regardless of tool: **customer PII never enters the repository.**

---

## Where the contract lives — two valid topologies

Earlier drafts of ADRIAN required the contract to live outside git. That was a property of the authors' tooling, not of the methodology. Both of these conform:

**Contract outside git.** Intake is conversational with real customers, so the origin carries names, screenshots, and account details. The contract lives in the requirements system; git gets the PR, which links back to it. This is the shape of in-app intake tools.

**Contract in git.** Intake is internal — a PM, a dev, or an agent writing a spec with no customer-identifying content in it. The contract lives in the repo, versioned alongside the code it specifies. This is the shape of spec-driven development toolchains, and it has a real advantage: the contract diffs with the code, so drift is visible in review.

The rule is about the PII, not the location. If your origin contains customer-identifying content and your contract lives in git, **reference the origin rather than embedding it** — a link into the system that holds the transcript, not a pasted transcript.

---

## Mapping real setups onto ADRIAN

The 2025–2026 tool landscape splits into three groups. None of them is a perfect fit out of the box; the last column is what ADRIAN asks you to add.

| Setup | Intake style | Contract lives | What you need to add |
|---|---|---|---|
| **In-app requirements agent** (BugLady — ADRIAN's reference implementation) | Chat interview with the actual end user, inside the product, with codebase and compliance analysis | Requirements system, outside git | Nothing structural. This is the shape ADRIAN was designed against |
| **Spec-driven development toolchains** ([GitHub Spec Kit](https://github.com/github/spec-kit), AWS Kiro, Tessl, OpenSpec, LeanSpec, BMAD-METHOD) | A human or agent authors the spec; no end-user interview | In the repo | **Verification** (these tools specify intent and tasks, rarely production signals), **User Impact × Business Context** scoring for queue position, and an Origin pointing at whatever triggered the spec |
| **Conversational spec generators** (ReqSpell, Gluecharm, PRDKit, MySpec, Dovetail) | Chat or interview transcript → structured spec document | Tool, doc store, or repo | A section mapping onto ADRIAN's six core sections, an approval/lock step, and a stable link for the PR |
| **Issue tracker + human PM** (Linear, Jira, GitHub Projects) | PM interviews the customer directly | Issue tracker | The whole Request Contract template as an issue template, plus the discipline to write up the interview as Origin. No agent means Phase 1 is slower, not impossible |

Two gaps show up repeatedly in the first three groups, so check for them specifically:

- **Verification is usually missing.** Most spec tooling stops at "what to build" and "how to build it." ADRIAN's Phase 3 gate is *verified in prod*, which needs a named metric or observation decided before implementation. If your tool has no field for it, add one.
- **Prioritization is usually missing or velocity-shaped.** ADRIAN prioritizes on User Impact × Business Context, scored by a human. Tools that order work by estimated effort or dependency graph don't supply this.

---

## What ADRIAN does not require

So that the tool conversation stays small:

- A specific coding agent. Claude Code, Cursor, Codex, Aider, and anything else that reads a rules file works. See [`CLAUDE-MD-TEMPLATES.md`](./CLAUDE-MD-TEMPLATES.md).
- A specific issue tracker, or any issue tracker, if the requirements system already satisfies the six requirements.
- A specific CI platform. The Phase 2 and 3 gates in [`PHASE-GATES.md`](./PHASE-GATES.md) describe checks, not vendors.
- An integration between any of them. The PR links to the contract; that's the only coupling ADRIAN needs.

If you find yourself adding tooling to satisfy ADRIAN, re-read the six requirements. The answer is usually a template and a convention, not a purchase.

---

## About the reference implementation

ADRIAN was developed at [Disruptica](https://disruptica.com) alongside [BugLady](https://buglady.ai), an in-app requirements agent, and every practice in this repo was exercised with it first. That's a disclosure, not a recommendation: the methodology is published separately *because* the two should be separable, and a framework that only works on its authors' tool isn't a framework.

Where the docs describe behavior that only BugLady currently provides, it's marked as such. If you run ADRIAN on different tooling and hit a requirement that turns out to be tool-shaped rather than methodology-shaped, that's a bug in these documents — see [`CONTRIBUTING.md`](./CONTRIBUTING.md).
