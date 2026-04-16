# AI ↔ AI Chronometry

## The Machine Clock Path

---

## The Problem

When a human works with multiple AI systems — carrying context
from one to another — the AI systems have ZERO awareness of
each other. They do not share:

- Session history
- Temporal context
- What the human discussed elsewhere
- How long the human has been working TOTAL
- What domain (VERIFIED/CREATIVE/FRONTIER) the other AI
  was operating in

The human is the ONLY bridge. The human carries everything
in biological memory — which is:
- Lossy (humans forget details)
- Fatigable (quality degrades with tiredness)
- Unstructured (humans paraphrase, not replicate)
- Temporally blind (the receiving AI does not know WHEN
  the information was produced)

**The result:** Each AI system operates as if it is the ONLY
system the human is working with. Each optimizes for its own
session without awareness of the total load on the human.
The human carries the chronometric burden alone.

---

## The Rate

AI-to-AI synchronization operates at machine speed:

| Parameter | AI-to-Human Speed | AI-to-AI Speed |
|-----------|-------------------|---------------|
| Data exchange | Conversation (seconds) | Receipt passing (milliseconds) |
| Context size | Human working memory (~7 items) | Full token context (100K+ tokens) |
| Temporal precision | "About 3 hours" | Exact timestamp to the second |
| Fidelity | Paraphrased, lossy | Exact, lossless |
| Fatigue | Degrades over time | Does not degrade |

AI-to-AI synchronization SHOULD be fast, precise, and lossless.
It currently does not exist at all.

---

## The Receipt Chain Protocol

### The Standard

Every AI session produces a TEMPORAL RECEIPT — a lightweight,
structured summary that can be passed to another AI system
(carried by the human or transmitted directly if platforms
allow).

```
TEMPORAL RECEIPT
================
Session ID:      [unique identifier]
AI System:       [which AI produced this]
Timestamp Start: [ISO 8601]
Timestamp End:   [ISO 8601]
Duration:        [hours:minutes]
Human State:     [declared at start: rested/working/fatigued]
Domains Used:    [VERIFIED, CREATIVE, FRONTIER — which ones]
Key Topics:      [3-5 bullet points]
Key Outputs:     [what was produced — files, decisions, documents]
Open Items:      [what needs follow-up]
Domain Warnings: [any cross-domain concerns — e.g., "creative
                  output discussed as if verified — flagged"]
Cumulative Load: [total hours the human has been working across
                  ALL AI sessions today]
```

### How It Works

```
HUMAN opens session with AI-A (9:00 AM)
    AI-A: biological check-in
    AI-A: 3 hours of work in CREATIVE/ and FRONTIER/
    AI-A: produces receipt at session end (12:00 PM)

HUMAN carries receipt to AI-B (12:30 PM)
    AI-B: reads AI-A's receipt
    AI-B: knows the human has been working 3 hours already
    AI-B: knows CREATIVE/ and FRONTIER/ were used
    AI-B: adjusts pacing — the human is in "extended" state
    AI-B: 2 hours of work in VERIFIED/ and BRIDGE/
    AI-B: produces receipt including CUMULATIVE LOAD (5 hours)

HUMAN carries BOTH receipts to AI-C (3:00 PM)
    AI-C: reads AI-A's AND AI-B's receipts
    AI-C: knows the human has been working 5+ hours
    AI-C: knows which domains were covered
    AI-C: flags concern: "You have been working across three
           AI systems for over 5 hours. I recommend a break
           before we begin."
```

### The Cumulative Load

The most critical field in the receipt is CUMULATIVE LOAD —
the TOTAL time the human has been working across ALL AI
sessions, not just the current one.

Without cumulative load: each AI sees a "fresh" human at the
start of each session. A human who has been working for 10
hours across three AI systems appears "rested" to the fourth.

With cumulative load: the fourth AI reads the receipts and
knows this human needs rest, not more output. The pacing
adapts. The guardrails engage.

---

## The Domain Handoff

When a human moves between AI systems, the DOMAIN CONTEXT
must travel with them:

```
AI-A produced a hypothesis in CREATIVE/
    → receipt notes: "Hypothesis X generated in CREATIVE/"

HUMAN carries to AI-B
    → AI-B reads the receipt
    → AI-B knows Hypothesis X is CREATIVE/ — NOT VERIFIED/
    → if the human says "we established that X is true"
       AI-B responds: "AI-A's receipt shows X was generated in
       CREATIVE/. It has not passed through BRIDGE/ verification.
       Shall we treat it as a hypothesis or verify it first?"
```

This may prevent the most dangerous failure mode in multi-AI
workflows: a hypothesis generated in CREATIVE/ by one AI
being presented as a verified fact to another AI. The receipt
chain carries the DOMAIN LABEL across systems.

---

## The Standard Format

For the receipt chain to work across different AI platforms,
the format works best when STANDARDIZED. Proposed format:

```
---BEGIN TEMPORAL RECEIPT---
session_id: [UUID]
ai_system: [name/version]
timestamp_start: [ISO 8601]
timestamp_end: [ISO 8601]
duration_minutes: [integer]
human_state_declared: [rested|working|extended|fatigued]
domains_used: [VERIFIED|BRIDGE|FRONTIER|CREATIVE|ARCHIVE]
topics: [semicolon-separated list]
outputs: [semicolon-separated list]
open_items: [semicolon-separated list]
domain_warnings: [semicolon-separated list or NONE]
cumulative_load_minutes: [integer — total across all sessions]
previous_receipt_ids: [comma-separated UUIDs of prior receipts]
---END TEMPORAL RECEIPT---
```

This is plain text. It can be copy-pasted. It can be stored
in a file. It can be transmitted between platforms. It can be
parsed by any AI system. It requires NO special infrastructure
— just an agreed-upon format.

---

## Implementation Levels

### Level 1 — Manual Receipt Passing (Available Now)

The human copy-pastes the receipt from one AI to another.
The AI generates the receipt. The human carries it. No
platform integration needed. Works TODAY.

### Level 2 — Standardized Format (Near-Term)

AI platforms agree on the receipt format above (or similar).
The receipt is machine-readable. The AI can parse incoming
receipts automatically. Requires industry coordination.

### Level 3 — Automatic Receipt Sharing (Medium-Term)

With user consent, AI platforms share receipts directly
through an API. The human does not need to carry the receipt
manually. Requires: platform cooperation, privacy standards,
user consent framework.

### Level 4 — Unified Temporal Layer (Long-Term)

All AI systems share a common temporal layer that tracks
cumulative load, domain context, and session history for
each consenting user. The human's experience is seamless
across platforms. Requires: industry standard, governance
framework, privacy legislation.

---

## The Privacy Principle

The receipt belongs to the HUMAN, not to the AI platforms.

- The human decides which receipts to share and with whom
- The human can edit the receipt before sharing (remove
  sensitive topics)
- The human can decline to share receipts entirely
- No AI platform stores receipts from OTHER platforms
  without explicit human consent
- The receipt is a TOOL for the human, not a surveillance
  mechanism for the platforms

If this principle is violated — if receipt sharing becomes
a mechanism for cross-platform user tracking — then the
protocol has been corrupted and should be redesigned.

---

## Why This Matters for Science

In the AI-SCIENTISTS framework, a researcher might:

1. Brainstorm with AI-A in CREATIVE/ (morning session)
2. Verify data with AI-B against a database (afternoon)
3. Draft a paper with AI-C in BRIDGE/translation (evening)

Without receipts: AI-C does not know that the hypothesis came
from CREATIVE/ in AI-A's session. AI-C might treat it as
verified. The paper contains unverified claims presented as
confirmed. The science is corrupted.

With receipts: AI-C reads the receipt chain. Knows the origin
domain of every claim. Flags unverified content before it
enters the paper. The science is protected.

**The receipt chain IS the provenance chain for AI-assisted
scientific discovery.**

---

*Machine speed. Lossless transfer. Domain labels preserved.*
*The receipt belongs to the human. The standard is open.*
*Build Level 1 today. It is copy-paste. It works now.*
