# References

ADRIAN is a synthesis of public 2025–2026 practices, adapted to a specific team structure. This file documents what's borrowed and what's original.

If you find prior art for anything labeled "original," please open an issue — accurate attribution matters more than novelty.

---

## What's borrowed

### Spec-driven development as the spine

The idea that requirements are executable contracts, not passive documentation, is from the spec-driven development movement that emerged in late 2025 and accelerated in 2026. ADRIAN adapts it into **request-driven** contracts: the contract is born from a user interview, carries the interview as agent context, and lives in the requirements tool rather than the repository.

- GitHub Spec Kit (2025, ongoing) — the most widely-used implementation
- Augment Code's Intent platform with the Coordinator/Implementor/Verifier pattern
- arXiv: "Spec-Driven Development: From Code to Contract in the Age of AI" (Feb 2026)
- DeepLearning.AI course: "Spec-Driven Development with Coding Agents" with Paul Everett (JetBrains)
- Chris Bao, "Exploring Spec Driven Development (SDD)" — practical guide with GitHub SpecKit (March 2026)

### Six-section request contract

The structure (outcomes, scope, constraints, decisions, tasks, verification) is adapted from Augment Code's published SDD framework. ADRIAN renames "outcomes" to "Intent" and "tasks" to "Acceptance" because those names match what the sections actually do in this methodology. The substance is theirs; the renaming and the UX-state requirement is the adaptation.

### Coordinator / Implementor / Verifier pattern

Augment Code's published agent pattern. Also seen in MetaGPT (academic). ADRIAN doesn't implement this as a multi-agent system — it implements it as a workflow distributed across PM (specifying), agent (implementing), and QA + automated checks (verifying). Same shape, different distribution.

### CLAUDE.md / agent instruction files

Standard practice in Claude Code, Cursor, Aider, Continue, and similar tools by 2026. The technique of "treat the rules file as compounding loop, update on every drift finding" is a community practice, not original.

### Symbol footprint check

Steve Yegge's "brute squad" framing, popularized via Sourcegraph's "Agentic Coding in 2026: A Practical Guide for Big Code." The specific phrasing: "when an agent says it's done, search the codebase for any other usage of the symbols it touched. If something turns up that the agent never opened, the task isn't done." This goes into ADRIAN verbatim because it's the highest-yield single technique published.

### Mutation testing for AI-generated code

Not new — mutation testing predates AI — but the specific recommendation that mutation testing is *especially* important for AI-generated code because of tautological test risk is from 2026 community discussion (CodeRabbit, Augment, others).

### DORA + AI-attribution metrics

DORA is foundational (Forsgren, Humble, Kim — well-established). The addition of AI-attribution to DORA metrics is from 2026 vendors (DX, Faros AI, Exceeds AI). The specific metric of "30-day rework rate on AI-touched code" is published by Exceeds AI.

The five-metric dashboard composition (which metrics specifically) is ADRIAN's choice, made for a small-team context. The metrics themselves are not original.

### Architectural fitness functions

From Neal Ford, Rebecca Parsons, Patrick Kua's "Building Evolutionary Architectures" (2017, second edition 2023). The application to AI-first codebases is a 2025–2026 trend, not original to ADRIAN.

### Self-managing team concept

Drawn broadly from Scrum's team autonomy, Holacracy, and the "Squad" model from Spotify (with the caveat that Spotify themselves moved away from that model). ADRIAN's specific definition — supervision is metrics and gates, not approval chains — is a synthesis of these traditions rather than a borrowing from any one.

### Phase structure inspiration

The four-phase Align/Implement/Deliver/Adapt structure is influenced by:
- Shape Up's shaping/building/cool-down (Basecamp, 2019)
- Spec Kit's specify/plan/tasks/implement structure
- The Agentsway framework from arXiv (Oct 2025) — which has a different shape but addresses the same problem

The specific four-phase structure with these names is ADRIAN's synthesis.

---

## What's original

To be precise about what ADRIAN contributes vs. assembles:

### The ADRIAN name and the four-phase loop
Not previously a recognized framework — created at Disruptica. The name, the Align/Implement/Deliver/Adapt loop, and the Request as the unit of work are ADRIAN's contribution.

### The specific gate checklists
The base checklists in `PHASE-GATES.md` are ADRIAN's synthesis. Individual items are mostly borrowed (e.g., symbol footprint check), but the consolidation into three gates with these specific items is the contribution.

### The "Flex Modes" compliance overlay
Treating compliance as an overlay on a single base methodology, with specific additions per regime (SOC2/HIPAA/PCI/gov), is ADRIAN's framing. Most compliance literature treats regulated work as a fork of the process. ADRIAN argues for a single process with additive overlay.

### The two-word "QA / Success" role definition
Combining quality assurance and customer success into one role, with the explicit four responsibilities (adversarial tests, UX states, production verification, customer loop), is ADRIAN's framing. The components exist elsewhere; the role consolidation is the contribution.

### The PM's four checks in Phase 1
The wrong-feature check, one-way door check, UX state check, and domain flag check as the PM's explicit pressure-test loop is ADRIAN's framing. Components are borrowed (one-way door is from Bezos's writing on decision types).

### The eight-week adoption plan
The week-by-week structure is original to ADRIAN. Individual week activities are mostly borrowed (week 1 foundations, week 7 tuning, etc.). The sequencing is the contribution.

### The threshold values in the dashboard
The specific bands (e.g., defect respawn <10% healthy, mutation score >75% healthy) are calibrated from published benchmarks but are ADRIAN's specific choices. They're starting points; expect to revise after two months of your own data.

---

## What's deliberately NOT included

Things I considered and left out, with reasoning:

### Multi-agent orchestration
ADRIAN assumes a single Dev steering one or more agents on a request. Multi-agent orchestration (a "Coordinator" agent dispatching to "Implementor" agents to a "Verifier" agent) is a real pattern from Augment and others, but it adds complexity that doesn't pay off for small teams on small requests. Revisit when requests routinely take >2 days of agent work.

### Backstage / Internal Developer Platform
Backstage is the #5 CNCF project by velocity in 2026. It's a real and useful tool. But ADRIAN is a methodology, not a platform. The platform-engineering questions (golden paths, self-service, scorecards) are a separate concern from the methodology questions. They compose, but they don't belong in this doc.

### Specific tool choices for AI coding
Claude Code, Cursor, Aider, Continue, Cody, Windsurf — all are reasonable. ADRIAN is tool-agnostic; the methodology works with any. Tool choice is a separate decision, made by the team, revised as the tools mature.

### Vibe coding
Vibe coding (Karpathy's term, early 2025) is a legitimate mode for prototyping and personal projects. ADRIAN is explicitly the post-vibe-coding methodology — the one you adopt when "passes basic tests" isn't enough. They aren't competitors; they're tools for different contexts.

### A specific BugLady implementation
[BugLady](https://buglady.ai) is the companion tool to ADRIAN, developed alongside the methodology. ADRIAN describes what BugLady should produce (the Request Contract) and how it should behave (drafts, doesn't approve), but doesn't prescribe how BugLady is built internally. The methodology stands without it — the PM does more drafting — but the two are designed to compose.

---

## Reading list

If you want to go deeper on any of the borrowed elements:

### Spec-driven development
- DeepLearning.AI course: "Spec-Driven Development with Coding Agents" (Paul Everett, April 2026)
- GitHub Spec Kit documentation
- Augment Code's published SDD guides
- arXiv: "Spec-Driven Development: From Code to Contract in the Age of AI" (Feb 2026)

### Agentic coding at scale
- Sourcegraph: "Agentic Coding in 2026: A Practical Guide for Big Code"
- Anthropic: "2026 Agentic Coding Trends Report"
- NxCode: "Agentic Engineering: The Complete Guide to AI-First Software Development Beyond Vibe Coding"

### AI code quality
- CodeRabbit blog: "2025 was the year of AI speed. 2026 will be the year of AI quality."
- Exceeds AI: 2026 AI Code Analysis Benchmarks
- Pensero: "AI Code Review for Enterprises in 2026"

### Metrics
- DORA's annual State of DevOps reports
- DX blog: "DORA metrics tools in 2026: What to measure, and what's missing"
- Oobeya: "DORA Metrics Are Not Enough in 2026"

### Foundational
- "Accelerate" (Forsgren, Humble, Kim) — DORA's foundation
- "Building Evolutionary Architectures" (Ford, Parsons, Kua) — fitness functions
- "Shape Up" (Singer/Basecamp) — adjacent methodology with structural parallels
- "Team Topologies" (Skelton, Pais) — team structure thinking

---

## Versioning ADRIAN itself

This is v0.1.

If you fork ADRIAN for your team, change the name. If you contribute back, propose changes via the same `Request` → `Request Contract` flow you'd use for code. The methodology eats its own cooking.
