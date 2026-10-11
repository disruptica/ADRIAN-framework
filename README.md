# ADRIAN — AI-Driven, Request-Intelligent, Agentic eNgineering

[![CI](https://github.com/disruptica/adrian-framework/actions/workflows/ci.yml/badge.svg)](https://github.com/disruptica/adrian-framework/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Version](https://img.shields.io/badge/version-0.2.0-blue.svg)](./CITATION.cff)

**An open source framework born at [Disruptica](https://disruptica.com).**

ADRIAN is a software development framework for small (3 to 10 people), self-managing teams that produce most of their code with AI agents (Claude Code, Cursor, Codex, Aider, etc.).

The framework is **tool-independent**: it needs a requirements system that satisfies six specific things, and it does not care which one. We run it on an in-app requirements agent ([BugLady](https://buglady.ai)), but ADRIAN works just as well on GitHub Projects, Linear, Jira, or a spec-driven toolchain like Spec Kit or Kiro. See [`TOOLING.md`](./TOOLING.md).

> **Born at:** [Disruptica](https://disruptica.com) — where we run ADRIAN daily on our own products
> **Author:** Jay (Jairo) Anaya
> **Reference implementation:** [BugLady](https://buglady.ai) — the in-app PM agent that interviews users and produces ADRIAN's Request Contracts. Disruptica's internal deployment is codenamed **Ramjet** — same tool, dogfooded on our own codebases before anything ships to buglady.ai. It is where ADRIAN was proven, not a dependency of it.

---

## The philosophy: chess vs squash

Software development used to be chess: study the board, plan ten moves ahead, commit to a strategy, execute it over months. Roadmaps, quarters, epics.

AI-first development is **squash**: move fast, react to where the ball actually is, reposition constantly. The ball is user need — it moves every day. Winning isn't predicting; it's *reacting with quality at speed* without loosing the north.

ADRIAN is built for squash:

- **The unit of work is a Request** — something a user or the business asked for, small enough to deliver in hours to days
- **Priority drives strategy**, not the other way around — the queue reshuffles as often as reality changes
- **ADRIAN rewards change.** Killing an in-flight Request because priorities shifted is not failure — it's the system working. Requests are small enough that killing one is cheap; kill and respawn beats mutating mid-flight
- **Quality lives in the contract and the pipeline**, not in slow ceremony — gates are automated and upstream, so speed doesn't cost correctness

---

## Honest framing — read this first

ADRIAN is a framework we built for ourselves first and open-sourced second. It is not (yet) a widely-adopted industry standard. There is no widely-adopted standard for AI-first agentic teams — the field is roughly 18 months old as of  October 2026.

What ADRIAN **is**:

- A synthesis of converging 2025–2026 practices: request-driven contracts, agent context engineering, AI-attributed quality metrics, mutation testing, architectural fitness functions
- Battle-tested at Disruptica on a team structure of **1 PM + 1 or more Devs + 1 QA/Success**, with BugLady (internally: Ramjet) as the requirements agent — though no document in this set requires it
- Designed to flex across unregulated and regulated domains via a compliance mode switch
- Version 0.2 — revised quarterly based on the metrics it produces

What ADRIAN **is not**:

- A published industry framework like Scrum or XP
- A replacement for engineering judgment
- A guarantee of quality — it is a structure that *enables* quality if practiced

References this draws from are listed in [`REFERENCES.md`](./REFERENCES.md).

---

## The core idea

> **The requirements are doing less work than they need to, and review is doing more work than it can sustain.**

When AI produces 95%+ of code, traditional review cannot keep up. The quality gates move **upstream into the Request Contract** and **downstream into automated verification**. Humans spend attention only on intent, judgment, and accountability.

### Where this came from

Working with Claude Code and Codex gave us extreme velocity on the coding side and extreme pressure on the quality side. The pressure landed on requirements: agents will build whatever a vague ticket implies, fast, and at volume. So we did two things — wrote down the methodology that became ADRIAN, and built an in-app requirements agent to fix intake (**BugLady**, internally **Ramjet**). The two grew up together, which is exactly why the methodology is published on its own: a framework that only works on its authors' tool isn't a framework.

### How a Request is born

1. Someone with a need — a user, the PM, the business — gets interviewed about it. Best case that's a chat inside the application, where a **requirements agent** runs the interview; without one, the PM runs it
2. Intake analyzes the conversation, the codebase, business goals, and compliance posture
3. It ends in a **Request Contract** — structured sections plus its **origin** (the interview transcript, or whatever else triggered the request) as context for both the dev team and the coding agents
4. The PM pressure-tests, sets priority (User Impact × Business Context), and approves. The contract locks
5. The Request enters the queue, gets implemented by agents under Dev steering, ships through automated gates, and is verified in staging and production

**Customer PII never enters the repository.** Where the contract itself lives — in a requirements system outside git, or in the repo alongside the code it specifies — is your call. Both topologies are valid; see [`TOOLING.md`](./TOOLING.md).

---

## The ADRIAN loop — four phases

| Phase | Who leads | Output | Time share |
|---|---|---|---|
| **Align** | PM + requirements agent | An approved Request Contract | 15–30% |
| **Implement** | Dev + AI agents | Working code, tests, docs | 40–50% |
| **Deliver** | CI/CD + QA | Verified production deploy | 15–25% |
| **Adapt** | Whole team + agents | Lessons learned → team practice AND agent context (`CLAUDE.md`, intake prompts) | 5–10% |

The Adapt phase feeds two learners: the **team** (the Quality Review — planning, check, and retro in one meeting) and the **agents** (updated rules files, updated intake prompts). Both compound.

---

## File index

Read in this order if you're new to ADRIAN:

1. [`METHODOLOGY.md`](./METHODOLOGY.md) — The full framework, all four phases in detail
2. [`ROLES.md`](./ROLES.md) — What PM, Dev, and QA actually do per phase
3. [`REQUEST-CONTRACT.md`](./REQUEST-CONTRACT.md) — The Request Contract template, the artifact everything downstream derives from
4. [`PHASE-GATES.md`](./PHASE-GATES.md) — Checklists for moving a Request between phases
5. [`FLEX-MODES.md`](./FLEX-MODES.md) — Compliance switches (none / SOC2 / HIPAA / PCI / gov)
6. [`CLAUDE-MD-TEMPLATES.md`](./CLAUDE-MD-TEMPLATES.md) — Agent instruction files for TS/Node, Python, Go
7. [`QUALITY-DASHBOARD.md`](./QUALITY-DASHBOARD.md) — Five metrics, thresholds, what each triggers
8. [`ADOPTION-PLAN.md`](./ADOPTION-PLAN.md) — Week-by-week rollout
9. [`TOOLING.md`](./TOOLING.md) — What ADRIAN requires of a requirements system, and how common tools map onto it
10. [`REFERENCES.md`](./REFERENCES.md) — What this draws from and what's original

Additional repo files:

- [`CONTRIBUTING.md`](./CONTRIBUTING.md) — How to propose changes
- [`LICENSE`](./LICENSE) — MIT
- [`CITATION.cff`](./CITATION.cff) — If you cite ADRIAN in writing

---

## About BugLady (and Ramjet)

[BugLady](https://buglady.ai) is an in-app product manager that lives inside each application you produce or maintain. It captures requirements through a chat interview by analyzing the codebase, business goals, and compliance posture, then generates the Request Contract that drives ADRIAN's Align phase — including the interview itself as context for the dev team and coding agents.

At Disruptica, our internal deployment of BugLady is codenamed **Ramjet**. Same tool — we run it against our own products first. Every ADRIAN practice in this repo was exercised with Ramjet before being published.

**It is the reference implementation, not a requirement.** ADRIAN asks six things of a requirements system; BugLady satisfies them natively because the two were built together, and plenty of tooling you already own satisfies them with a template and a convention. Without any requirements agent, the PM conducts the interview and drafts the contract — Phase 1 costs more human time and the methodology is unchanged. [`TOOLING.md`](./TOOLING.md) has the requirements and maps the common alternatives onto them.

Mentions of BugLady are confined to this README, [`TOOLING.md`](./TOOLING.md), [`CONTRIBUTING.md`](./CONTRIBUTING.md), and [`REFERENCES.md`](./REFERENCES.md). The methodology documents name no tool at all — if you find one that does, that's a bug worth reporting.

---

## About Disruptica

ADRIAN was born at [Disruptica](https://disruptica.com) and is the framework we use internally to build and maintain our products. We open-sourced it because the practices are more valuable shared than hoarded — and because training material, workshops, and developer education derive from this canonical source.

---

## License

MIT — see [`LICENSE`](./LICENSE). Use it, fork it, rename it. If you improve it, consider contributing back via [`CONTRIBUTING.md`](./CONTRIBUTING.md).

## Citation

If you reference ADRIAN in writing, please cite:

> Anaya, J. (2026). *ADRIAN — AI-Driven, Request-Intelligent, Agentic eNgineering* (Version 0.2) [Framework specification]. Disruptica. https://github.com/disruptica/adrian-framework

A machine-readable citation is in [`CITATION.cff`](./CITATION.cff).
