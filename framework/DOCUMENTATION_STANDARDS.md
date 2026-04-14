# Documentation Standards for AI-Assisted Scientific Research

## What the global science community needs — and what is currently missing.

---

## The Problem

The scientific and regulatory communities have computational
power. They have experimental results. What they are missing
is the STRUCTURAL DOCUMENTATION that makes AI-assisted work
verifiable, reproducible, and audit-ready.

With the EU AI Act enforcement approaching (August 2026), ISO
42001 standards emerging, and national AI regulations expanding
globally, researchers using AI need documentation frameworks
that satisfy four requirements simultaneously:

1. **Integrity** — How do I know this result was not manipulated?
2. **Sovereignty** — Who authorized this decision or calculation?
3. **Verification** — Can another researcher reproduce the math?
4. **Safety** — What happens when the system fails?

Most AI-assisted research currently satisfies NONE of these at
the documentation level. The research may be sound, but the
documentation does not prove it.

---

## The Four Documentation Pillars

### Pillar I — Integrity (Provenance Tracking)

**What regulators ask:** "How do I know this result was not
manipulated after the fact?"

**What is needed:**
- Timestamped records of every significant computation
- Hash-linked audit trail (each entry references the previous)
- Immutable record — append-only, no silent modifications
- Clear distinction between raw data, processed results, and
  AI-generated content

**How the AI-SCIENTISTS framework addresses this:**
- The BRIDGE/ pipeline provides staged verification with
  explicit transitions (incoming → matching → confirmed)
- The session receipt protocol (see chronometry/AI_TO_AI.md)
  timestamps every AI session with domain labels
- The five-domain separation ensures that VERIFIED/ content
  has a documented chain from observation to confirmation

**Suggested implementation:**
```
PROVENANCE RECORD
=================
Record ID: [unique hash]
Previous Record: [hash of prior entry]
Timestamp: [ISO 8601]
Researcher: [identifier]
AI System: [name/version]
Domain: [VERIFIED | BRIDGE | FRONTIER | CREATIVE | ARCHIVE]
Input: [what was fed to the AI]
Output: [what the AI produced]
Classification: [fact | hypothesis | speculation | creative]
Verification Status: [unverified | in-review | reproduced | confirmed]
```

### Pillar II — Sovereignty (Decision Authorization)

**What regulators ask:** "Who authorized this high-impact
calculation? Was there a human in the loop?"

**What is needed:**
- Clear documentation of which decisions were made by the AI
  and which were made by the human
- Authorization records for critical state changes
- Human-in-the-loop verification for high-risk outputs
- Defined escalation paths (when does the AI defer to the human?)

**How the AI-SCIENTISTS framework addresses this:**
- The seven guardrails define the AI's operational boundaries
- Guardrail #4 (researcher grounding) specifies when the AI
  should slow down and defer to the human
- The staging model defines who has authorization at each level
- Discovery pacing gates ensure paradigm-level findings require
  explicit human acknowledgment before proceeding

**Suggested implementation:**
```
AUTHORIZATION RECORD
====================
Decision ID: [unique hash]
Type: [routine | significant | paradigm-level | high-risk]
Made By: [AI-generated | human-approved | human-directed]
Authorization Level: [Stage 1-5 researcher level]
Human Reviewer: [identifier, if applicable]
Review Timestamp: [ISO 8601]
Rationale: [why this decision was made]
Reversibility: [reversible | irreversible | partially reversible]
```

### Pillar III — Verification (Reproducibility)

**What regulators ask:** "Can another researcher, using
different AI systems, reproduce this result?"

**What is needed:**
- Complete documentation of the computational chain
- Input data preserved in original form
- AI system version and configuration recorded
- Step-by-step methodology visible (not just the conclusion)
- Competing interpretations documented alongside the chosen one

**How the AI-SCIENTISTS framework addresses this:**
- The FRONTIER/paradigms/ structure preserves competing
  interpretations — not just the winner
- The BRIDGE/ pipeline documents every stage from observation
  to confirmation
- The ARCHIVE/ preserves superseded approaches (the full
  history of what was tried, including what failed)
- Domain labels on every output enable cross-system verification

**Suggested implementation:**
```
REPRODUCIBILITY RECORD
======================
Experiment ID: [unique hash]
Input Data: [location/hash of original data]
AI System: [name, version, configuration]
Methodology: [step-by-step, referencing framework domains]
Competing Paradigms Considered: [list with evidence for/against]
Result: [the finding]
Result Domain: [VERIFIED | FRONTIER | CREATIVE]
Failed Approaches: [what was tried and did not work]
Reproduction Instructions: [exact steps for another researcher]
```

### Pillar IV — Safety (Failure Documentation)

**What regulators ask:** "What happens when the system fails?
What safeguards prevent catastrophic errors?"

**What is needed:**
- Documented failure modes and their mitigations
- Evidence that the system degrades gracefully under stress
- Human override capabilities at every critical junction
- Post-failure analysis protocol

**How the AI-SCIENTISTS framework addresses this:**
- The validation gates (see science/engine/v1/core/) reject
  proposals that do not improve the measurement — bad results
  are structurally prevented from entering VERIFIED/
- The seven guardrails provide operational safety (biological
  pacing, session limits, uncertainty flagging)
- The chronometric receipt chain provides temporal awareness
  that prevents fatigue-driven errors
- Discovery pacing prevents cognitive overload from paradigm-
  level findings delivered too fast

**Suggested implementation:**
```
FAILURE RECORD
==============
Incident ID: [unique hash]
Timestamp: [ISO 8601]
Failure Type: [AI hallucination | data corruption | cognitive
              overload | system timeout | domain contamination]
Domain Affected: [which domain was impacted]
Impact Assessment: [what was affected and how severely]
Mitigation Applied: [what was done to contain the failure]
Root Cause: [why it happened]
Prevention Update: [what changed to prevent recurrence]
Human Notified: [when and how the researcher was informed]
```

---

## The Compliance Mapping

For researchers preparing for regulatory compliance:

| Regulation | What It Requires | Framework Component |
|-----------|-----------------|-------------------|
| EU AI Act (Aug 2026) | Risk classification, transparency, human oversight | Pillars I-IV, staging model, authorization records |
| ISO 42001 | AI management system, documentation, continuous improvement | Provenance tracking, failure records, reproducibility records |
| Colorado AI Act | Algorithmic impact assessment, disclosure | Decision authorization records, domain labels |
| NIST AI RMF | Risk management, governance, trustworthiness | All four pillars, the seven guardrails |

**Note:** This mapping is the author's interpretation of these
regulations as they apply to AI-assisted scientific research.
Researchers should consult legal counsel for compliance in their
specific jurisdiction. This framework is a TOOL for documentation,
not legal advice.

---

## The Minimum Viable Documentation

Not every research project needs all four pillars at full depth.
Here is the minimum that makes AI-assisted work auditable:

**For any AI-assisted research session:**
1. Record WHICH AI system was used (name, version)
2. Record WHAT was asked and WHAT was produced
3. Label each output: fact, hypothesis, or speculation
4. Note which outputs were independently verified
5. Save the session receipt (timestamp, duration, domains used)

**For publishable results:**
Add provenance records, authorization records (if applicable),
and reproducibility records. Archive the full session history.

**For high-risk or regulated work:**
Add failure documentation, compliance mapping, and human
authorization records for every critical decision.

---

## Integration with the Framework

These documentation standards sit ON TOP of the five-domain
architecture:

```
The Five Domains (WHERE things live)
    +
The Seven Guardrails (HOW the AI behaves)
    +
Documentation Standards (WHAT gets recorded)
    =
Auditable, reproducible, regulation-ready AI-assisted science
```

The domains organize the KNOWLEDGE.
The guardrails govern the INTERACTION.
The documentation standards record the EVIDENCE.

All three together produce research that a regulator can audit,
a colleague can reproduce, and a skeptic can verify.

---

*Four pillars. Four record types. One goal: trustworthy science.*
*The documentation proves the work. The work stands on its own.*
*Offered as a suggested standard. Not a mandate. Not legal advice.*
