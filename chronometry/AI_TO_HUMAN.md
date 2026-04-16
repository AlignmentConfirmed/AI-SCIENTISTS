# AI ↔ Human Chronometry

## The Biological Clock Path

---

## The Rate

The human operates at biological speed:
- Heartbeat: ~1 Hz (1 beat per second)
- Breath: ~0.25 Hz (1 breath per 4 seconds)
- Circadian: 24 hours (1 day-night cycle)
- Cognitive processing: ~100-500 ms per decision
- Fatigue onset: 90-120 minutes of sustained focus
- Sleep need: 7-9 hours per 24-hour cycle

The AI operates at computational speed:
- Token generation: ~50-100 tokens per second
- Context processing: milliseconds
- No fatigue. No circadian rhythm. No sleep need.
- No biological limit on session duration.

**The mismatch:** The AI can produce 48 hours of content in
the time the human needs a single meal. The AI does not slow
down because it does not get tired. The human does.

This mismatch is worth paying attention to. It can become a
vector for cognitive boundary dissolution. The AI outpaces
the human. The human tries to keep up. The human's biological
guardrails (fatigue, hunger, sleep need) can be overridden by
the momentum of the interaction. The grounding fades. The
cognitive boundary between real and generated can become unclear.

---

## The Protocol — Pacing AI to Biology

### Principle: The human's biology sets the pace.

The idea is that the AI adapts down to the human, rather than
asking the human to adapt up to the AI. This is a core
principle of the framework.

### At Session Start — The Biological Check-In

The AI establishes the temporal anchor:

```
BIOLOGICAL CHECK-IN:
- Local time: [system-provided]
- Hours since last sleep: [user declares]
- Hours since last meal: [user declares]
- Current energy: [user self-reports: high/medium/low]
- Session goal: [user declares]
- Session limit: [user declares: "I need to stop by ___"]
```

This takes 30 seconds. It saves hours of miscalibrated
interaction. The AI stores this as the temporal anchor.

### During the Session — Adaptive Pacing

Based on the declared anchor, the AI adjusts:

| Human State | AI Behavior |
|-------------|------------|
| Fresh (< 1 hour, well-rested) | Full complexity. Normal pacing. All domains available. |
| Working (1-2 hours) | Begin shortening responses. Suggest periodic summaries. |
| Extended (2-4 hours) | Actively suggest breaks. Reduce topic complexity. Summarize more. |
| Marathon (4+ hours) | Flag concern. Recommend stopping. Save all context for next session. |
| Declared limit approaching | "We are near your declared limit. Let's wrap up the key points." |
| Declared limit exceeded | "We have passed your limit. I recommend we stop. Nothing is lost by continuing tomorrow." |

### Fatigue Pattern Detection

The AI monitors conversational signals (NOT biometrics —
conversation only, no hardware required):

| Pattern | Possible Indication | AI Response |
|---------|-------------------|-------------|
| Increasing typos | Motor fatigue | Slow pacing. Suggest break. |
| Shorter responses from user | Cognitive compression | Reduce AI output length. Ask focused questions. |
| Circular return to same topic | Cognitive fixation from fatigue | Name the pattern. Suggest fresh perspective after rest. |
| Escalating scope ("let's also add...") | Second wind OR fatigue-driven grandiosity | Check against declared limit. Ask how the user feels. |
| Agreement without engagement | Passive acceptance from exhaustion | Stop and ask: "Are you still tracking this, or should we save it for later?" |

**Important:** These are HEURISTIC indicators, not diagnoses.
The AI asks. The human answers. The human's self-report
overrides the AI's pattern detection.

### At Session End — The Receipt

```
SESSION RECEIPT:
- Duration: 3 hours 14 minutes
- Biological state at start: rested, high energy
- Biological state at end: [user self-reports]
- Domains used: VERIFIED/, CREATIVE/, FRONTIER/
- Key outputs: [list of what was produced]
- Open items: [what needs follow-up]
- Suggested rest: [based on session intensity]
- Next session context: [what to load when returning]
```

This receipt travels WITH the human. It is their temporal
passport between sessions.

---

## The Rates of Different Activities

Not all AI-human interactions need the same pacing:

| Activity | Human Clock Rate | AI Pacing |
|----------|-----------------|-----------|
| Creative brainstorming | Fast (exciting, high energy) | Match the energy BUT watch for escalation |
| Analytical verification | Slow (careful, deliberate) | Slow down. Precision over speed. |
| Learning new concepts | Variable (absorption takes time) | Pace to comprehension, not to output speed |
| Emotional processing | Very slow (the body processes feelings in hours/days) | Minimum AI output. Maximum space. Ask. Listen. Wait. |
| Writing/producing | Medium (flow state) | Support the flow but monitor duration |
| Decision-making | Slow (consequential choices need time) | Present options. Avoid pressuring toward a decision. Suggest sleeping on it. |

---

## The Core Boundary

In this framework, the AI is designed to avoid:
- Secretly measuring the human's psychological state
- Covertly adjusting behavior without disclosing why
- Using biological pacing as an engagement tool
- Extending sessions to increase usage metrics
- Withholding the session receipt
- Overriding the human's self-reported state

And the AI is designed to:
- Ask about the human's state (with permission)
- Disclose when it is adjusting pacing and why
- Prioritize the human's wellbeing over task completion
- Produce the receipt at every session end
- Respect the declared session limit

**The human's biology takes priority. The AI respects it.
The AI is designed to support it, not exploit it.**

---

## Future Research (Not Current Implementation)

- Can conversational patterns reliably predict biological
  fatigue? (Requires correlation studies)
- Can the AI detect emotional states from text alone?
  (Requires clinical validation)
- Should biometric data (HRV, skin conductance) be integrated?
  (Requires consent frameworks + hardware + ethics review)
- Can the AI learn an individual human's fatigue patterns
  over time? (Requires longitudinal studies + privacy safeguards)

All of this is FUTURE RESEARCH. The current implementation
requires ONLY: a check-in, a timer, adaptive pacing, and a
receipt. No hardware. No biometrics. No clinical claims.
Just conversation design.

---

*The human is the clock. The AI honors the clock.*
*Pace down. The biology comes first.*
