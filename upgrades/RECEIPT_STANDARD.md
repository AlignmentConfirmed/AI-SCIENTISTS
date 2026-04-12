# Upgrade: Session Receipt Standard

## Priority: 3 (Critical for multi-AI workflows)
## Complexity: MEDIUM — protocol design + industry coordination
## Status: PROPOSED IN CHRONOMETRY DOCS

---

## The Problem

When a human works with multiple AI systems, no temporal
context travels between sessions. Each AI sees a "fresh"
human. The human carries the cognitive load of remembering
what happened in previous sessions with other systems.

## The Proposed Solution

A standardized, lightweight, plain-text receipt format that
any AI system can produce and any AI system can read.

The format is already proposed in chronometry/AI_TO_AI.md:

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
cumulative_load_minutes: [integer]
previous_receipt_ids: [comma-separated UUIDs]
---END TEMPORAL RECEIPT---
```

## What Needs To Happen

1. Formalize the format as a specification (RFC-style)
2. Get feedback from AI platform engineers (Anthropic, OpenAI,
   Google, open-source community)
3. Build reference implementations for major platforms
4. Establish a governance model for format evolution

## Who Would Build This

- AI platform engineers who control the session layer
- Standards body participation (if the format gains traction)
- Open-source community for reference implementations

## Estimated Effort

| Phase | Work | Duration |
|-------|------|----------|
| 1 | Formalize specification | 1-2 months |
| 2 | Reference implementation | 2-3 months |
| 3 | Platform adoption | 6-12 months |
| 4 | Cross-platform testing | Ongoing |

---

*The receipt belongs to the human. The format is open.*
*Any AI system can produce it. Any AI system can read it.*
