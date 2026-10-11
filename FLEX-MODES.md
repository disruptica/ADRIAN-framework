# Flex Modes — Compliance Switches

ADRIAN must run a single process that flexes across unregulated and regulated domains. The compliance flag in the contract's Constraints section drives every downstream variation.

This document defines what each flag adds to the base methodology. **The base methodology is the unregulated case.** Regulated modes are additive — they add gates, attribution, and named reviewers. They don't replace the base.

---

## The flag is set in Phase 1 and propagates everywhere

```
Request Contract Constraints.compliance flag = X
    ↓
Phase 1 gate adds: data classification rigor
    ↓
Phase 2 gate adds: named reviewer (regulated)
    ↓
Phase 3 gate adds: deploy attribution, named approver
    ↓
Phase 4 adds: retention of audit artifacts
```

If the flag is wrong, every downstream gate is wrong. Get it right.

---

## Mode: `none`

**Applies to:** Internal tools, consumer experiments, B2B SaaS where customer data is not specially classified, prototypes.

**Additions to base:** None.

**Notes:** Even in `none` mode, do not skip the basics — PII data classification, secret management, basic SAST. "None" means no specific regulatory regime applies, not "no security."

---

## Mode: `SOC2`

**Applies to:** B2B SaaS where you've made (or plan to make) SOC 2 commitments to customers.

**Why it matters:** SOC 2 auditors want evidence that changes are authorized, attributable, and reviewed. The methodology must produce that evidence automatically.

### Additions

**Phase 1 (Contract):**
- Contract must list any data classifications we are committed to handle
- Contract author and approver are recorded in the requirements system (approval metadata)

**Phase 2 (PR):**
- PR must link the Request Contract it implements (stable link into the requirements system)
- PR author and approver recorded (the PR review acts as authorization)
- AI-attribution tag set (auditors increasingly want to know what was AI-generated)

**Phase 3 (Deploy):**
- Deploy attribution: who deployed, when, which artifact
- Production access logged
- If the deploy touched access controls or auth: explicit changelog entry

**Phase 4 (Retain):**
- All contract, PR, deploy artifacts retained per the SOC 2 retention period (commonly 1 year minimum)

### What you do NOT need

- A named human reviewer on every PR (the standard PR review is sufficient for SOC 2)
- A separate approval chain for deploys (CI/CD with attribution is sufficient)

SOC 2 is the lightest regulated mode. Most B2B SaaS teams can run this without slowing down meaningfully.

---

## Mode: `HIPAA`

**Applies to:** Healthcare, anything that touches Protected Health Information (PHI).

**Why it matters:** HIPAA is strict about who can touch PHI and how. The "AI does it all" model needs explicit human accountability for PHI handling decisions.

### Additions

**Phase 1 (Contract):**
- Every field touched by the request must be tagged: PHI / not PHI
- If PHI is in scope: minimum necessary principle must be addressed (why does this request need this much PHI?)
- Vendor BAA status must be confirmed for every external service in the data path
- Audit log requirements for PHI access spelled out

**Phase 2 (PR):**
- **Named human reviewer required** — a specific person on the team, by name, who signs the PR
- The reviewer must verify: minimum necessary, encryption at rest/in transit, audit logging present
- Agent must NOT log PHI fields — verified by automated check on log statements

**Phase 3 (Deploy):**
- Deploy goes to BAA-compliant infrastructure only
- Verify TLS configuration on the deployed service
- PHI access audit log is operational post-deploy

**Phase 4 (Retain):**
- Audit artifacts retained per HIPAA (6 years from creation or last effective date)

### Specific `CLAUDE.md` additions for HIPAA repos

```
HIPAA — never violate:
- Never log PHI fields (see fields tagged @phi in schemas)
- Never include PHI in error messages
- Never call non-BAA external services with PHI in the request body
- All PHI access must go through the audit-logger wrapper
- Encryption keys never appear in code — always from the secret manager
```

---

## Mode: `PCI`

**Applies to:** Anything that touches payment card data — even tokenized references in some scopes.

**Why it matters:** PCI DSS is the strictest common compliance regime. Scope creep is the killer — a feature that "just" stores a card token can pull whole services into PCI scope, with massive downstream consequences.

### Additions

**Phase 1 (Contract):**
- PCI scope confirmation: does this request keep us in our current scope, or expand it?
- If scope expands: STOP. Escalate. Scope changes are not request-level decisions.
- Tokenization strategy explicit — never raw card data unless we are a card processor (we are not)
- All external services in the data path verified PCI-compliant

**Phase 2 (PR):**
- **Named human reviewer required** — specifically someone certified to review PCI-scope changes
- Reviewer verifies: no raw card data, tokenization correct, no card data in logs, no card data in error messages
- Secrets review required (no inline keys, no hardcoded credentials)
- SAST results reviewed manually for PCI-relevant findings

**Phase 3 (Deploy):**
- Deploy attribution signed by named approver
- Deploy goes to PCI-segregated infrastructure
- Network segmentation verified post-deploy

**Phase 4 (Retain):**
- Audit artifacts retained per PCI (commonly 1 year online, additional offline)
- Quarterly scope review must include this request if it touched PCI surface

### Specific `CLAUDE.md` additions for PCI repos

```
PCI — never violate:
- Never log card numbers, CVV, expiration dates, or full PANs
- Never store raw card data — always provider tokens
- Never display CVV or full PAN — last 4 only
- Never call non-PCI external services from PCI-scope code
- All payment provider calls go through the resilience+audit wrapper
- Encryption keys never appear in code — always from the secret manager
```

---

## Mode: `gov`

**Applies to:** Government contracts — FedRAMP, IL4/5, state government, etc.

**Why it matters:** Government work has data residency requirements, personnel requirements, and audit requirements that go beyond commercial regulated modes.

### Additions

**Phase 1 (Contract):**
- Data residency requirement explicit (which region/cloud/account)
- Personnel requirement if any (e.g., US citizens only for certain data)
- Classification level if applicable

**Phase 2 (PR):**
- **Named human reviewer required** — must meet any personnel requirements (citizenship, clearance)
- All commits attributed to identifiable humans (signed commits)
- Agent-generated code clearly tagged for auditor review

**Phase 3 (Deploy):**
- **Named human approver required** on prod promotion
- Signed deploy record (cryptographically signed, not just attributed)
- Deploy goes only to authorized regions/accounts
- Post-deploy verification that data is in the correct region

**Phase 4 (Retain):**
- Retention per contract — typically longer than commercial regimes
- All AI-generated code identifiable in audit trail (auditors are increasingly asking)

### Specific `CLAUDE.md` additions for gov repos

```
GOV — never violate:
- Never call external services outside the authorized region/account
- All data writes must verify region before completing
- Never log data classified above the system's authorization
- Encryption keys from FIPS-validated KMS only
```

---

## The named reviewer question — the hard part

Regulated modes require a "named human reviewer" or "named approver." This is the requirement that breaks the "AI does everything" dream.

**A regulator will not accept "the AI wrote it and the AI tested it" as evidence of human review on a PHI handling decision.** They want a name, a date, and a signature.

How to implement this without recreating bureaucracy:

1. **The named reviewer is a team member, not an external approver.** Self-managing teams can have internal accountability.
2. **The named reviewer rotates by mode** — for HIPAA work, designate one Dev as the HIPAA reviewer. For PCI, another (or the same if cross-trained).
3. **The named reviewer is responsible for the *human-judgment* parts of the gate:** minimum necessary, no-card-data-in-logs, etc. The agent does the volume work and the CI does the mechanical checks.
4. **If the team is too small to have separate reviewers per mode, you have a real problem.** A team of 2 humans cannot satisfy "named reviewer is not the author" for regulated work. Either grow the team for that workstream or partner.

This is the place to be most honest with yourself. ADRIAN's flex modes work down to about 3 humans per workstream for regulated modes. Below that, the "single human does everything" pattern fails compliance regardless of how good your methodology is.

---

## How to choose the flag

A decision tree:

1. **Does this request touch PHI?** → HIPAA (and whatever else applies)
2. **Does this request touch card data, even tokens?** → PCI (and whatever else applies)
3. **Is this a government contract?** → gov (and whatever else applies)
4. **Have we made SOC 2 commitments to customers that this request could affect?** → SOC2
5. **None of the above?** → none

Multiple flags can apply. Combine the additions. The strictest mode wins on any conflict.

---

## What flexes — what doesn't

**Flexes:**
- Named reviewer requirement
- Deploy attribution rigor
- Audit retention period
- `CLAUDE.md` "never do" rules
- Pipeline checks (e.g., PHI-in-logs check only in HIPAA repos)

**Does not flex:**
- The four phases
- The three roles
- The request contract sections
- The basic gate items

This is what makes "a single process that flexes" possible. The phases and roles are constant. The rigor scales with risk.
