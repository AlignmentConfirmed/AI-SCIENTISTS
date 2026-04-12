# Design Principles → Engineering Gaps

## Each governing principle identifies something that current
## technology does NOT provide. Each gap is a thing to build.

---

## The Mapping

### Law I — No boolean gates in derivation
**The gap:** Every programming language defaults to if/else.
**What to build:** A language or compiler extension where
variational evaluation (score all candidates, measurement
selects) is the DEFAULT construct. Boolean gates require
explicit opt-in.
**See:** [LOGIC_LANGUAGE.md](LOGIC_LANGUAGE.md)
**Difficulty:** HIGH — language design (3-5 years)

---

### Law II — No HashMaps in cognition
**The gap:** Every standard library provides HashMap as the
primary associative data structure. It hashes structured keys
into flat buckets, destroying relational geometry.
**What to build:** A relational tensor data structure as a
standard library primitive. Store RELATIONSHIPS (antisymmetric,
multi-dimensional) instead of key-value pairs. Lookup by
structural similarity, not by hash.
**Difficulty:** MEDIUM — library design (6-12 months)

---

### Law III — The kernel is general
**The gap:** AI systems are trained on specific domains (image
recognition, language, protein folding). The underlying algebra
is the same, but each domain gets a separate implementation.
**What to build:** A single algebraic engine that is domain-
agnostic. The engine knows Delta, Q, convergence. The DOMAIN
is a plugin — an encoding layer that translates domain-specific
data into the engine's native tensor format.
**See:** [/science/engine/v1/core/](../science/engine/v1/core/README.md)
**Difficulty:** MEDIUM — this is what we are building.

---

### Law IV — Exact Q in the derivation path
**The gap:** All computing hardware operates in IEEE 754 float.
Every operation introduces rounding error. The errors compound.
**What to build:** Rational arithmetic as a hardware primitive.
Numerator + denominator stored as arbitrary-precision integers.
Exact arithmetic at every step. Zero error accumulation.
**See:** [RATIONAL_HARDWARE.md](RATIONAL_HARDWARE.md)
**Difficulty:** VERY HIGH — chip design (5-8 years)
**Interim:** Software-based BigRational (works, 10-100x slower)

---

### Law V — Residual is structured information
**The gap:** AI training loops minimize loss toward zero. When
loss approaches zero, training stops. The STRUCTURE of the
remaining error is discarded. But that structure tells you
exactly what the model has NOT learned.
**What to build:** A residual analysis module that DECOMPOSES
the error into its structural components — which dimensions,
which relationships, which data points are contributing to
the remaining loss. The training loop does not stop at low
loss. It READS the residual to determine the next training
action.
**Difficulty:** MEDIUM — analysis tooling (6-12 months)

---

### Law VI — Don't solve for the puzzles
**The gap:** AI systems are trained on benchmarks. The
benchmark DEFINES what the system learns. The system optimizes
for the benchmark, not for general understanding. When the
benchmark is solved, the system appears intelligent — but it
has only learned the benchmark.
**What to build:** Benchmark-agnostic evaluation. The system
is measured by its GENERAL convergence capability (how well
does it reduce the coherence functional on UNSEEN data?) not
by its score on a specific test. The evaluation does not know
which problems will be presented.
**Difficulty:** MEDIUM — evaluation design (3-6 months)

---

### Law VII — Observe before executing
**The gap:** AI systems are deployed after training. Once
deployed, they act without re-measuring the environment. The
model was trained on data from BEFORE deployment. The world
changed. The model does not know.
**What to build:** A continuous measurement loop. The system
measures the environment BEFORE every action. If the
environment has changed since the last measurement, the system
re-evaluates before proceeding. Cyclic: measure → act →
measure. Never act → act → act.
**See:** [/public/chronometry/](../public/chronometry/README.md)
**Difficulty:** MEDIUM — architecture design (6-12 months)

---

### Law VIII — The alphabet stays fixed, the words grow
**The gap:** AI models grow by adding parameters. More
parameters = more capacity. But each parameter increase
requires retraining the entire model from scratch (or
expensive fine-tuning). The "vocabulary" does not compound.
**What to build:** Compositional operator expansion. The base
operators (the alphabet) are fixed and small. New capabilities
are built by COMPOSING base operators into sequences (words).
Words compose into sentences. Sentences are stored and reused.
The vocabulary grows EXPONENTIALLY from a finite alphabet —
without retraining the base model.
**Difficulty:** HIGH — fundamental architecture change (1-2 years)

---

### Law IX — Species restarts require permission
**The gap:** AI systems are shut down, restarted, and
retrained without preserving their accumulated state. Each
restart loses in-flight work. Each retraining starts from
a previous checkpoint, not from the current state.
**What to build:** Persistent sovereign state. The system's
accumulated knowledge (its persistent state) persists across restarts.
The system can be shut down and rebooted from its persistent state
without losing what it learned. The restart is a REBIRTH,
not a reset.
**Difficulty:** MEDIUM — state persistence design (6-12 months)

---

### Law X — Temporal anchors are immutable
**The gap:** AI training data is mutable. Models are
retrained. Previous versions are overwritten. The history of
what the model knew and when it knew it is lost. There is no
receipt chain.
**What to build:** Append-only knowledge ledger. Every fact the
system learns is timestamped and hash-linked to the previous
entry. The history is immutable. The system can prove WHEN it
learned something and WHAT it knew at any point in time. No
fact can be silently altered.
**See:** [RECEIPT_STANDARD.md](RECEIPT_STANDARD.md)
**Difficulty:** MEDIUM — blockchain/merkle tree (3-6 months)

---

### Law XI — No dimensional collapse
**The gap:** AI systems represent knowledge as flat vectors
(embeddings). A 768-dimensional embedding LOOKS high-
dimensional but is actually a FLAT list of numbers with no
relational structure. The geometry between dimensions is
discarded. The embedding for "love" and the embedding for
"chemical bond" live in the same flat space with no structural
distinction.
**What to build:** Structured multi-dimensional representation.
Knowledge stored as TENSORS with explicit relational geometry
— antisymmetric, multi-coordinate, with structural meaning
per dimension. Not flat vectors. The relationship between
dimensions IS the information, not just the values at each
dimension.
**Difficulty:** HIGH — representation design (1-2 years)

---

## Summary Table

| Law | The Gap | What To Build | Difficulty | Timeline |
|-----|---------|--------------|-----------|----------|
| I | Boolean gates are default | Variational evaluation language | HIGH | 3-5 years |
| II | HashMaps flatten structure | Relational tensor data structure | MEDIUM | 6-12 months |
| III | Domain-specific AI | Domain-agnostic algebraic engine | MEDIUM | In progress |
| IV | Float arithmetic rounds | Rational arithmetic hardware | VERY HIGH | 5-8 years |
| V | Residuals discarded | Structural residual analysis | MEDIUM | 6-12 months |
| VI | Benchmark-optimized AI | Benchmark-agnostic evaluation | MEDIUM | 3-6 months |
| VII | Deploy-and-forget | Continuous measurement loop | MEDIUM | 6-12 months |
| VIII | Parameter scaling | Compositional operator expansion | HIGH | 1-2 years |
| IX | Restart = reset | Persistent sovereign state | MEDIUM | 6-12 months |
| X | Mutable training history | Append-only knowledge ledger | MEDIUM | 3-6 months |
| XI | Flat embeddings | Structured tensor representation | HIGH | 1-2 years |

## The Observation

11 laws. 11 engineering gaps. NONE of them are solved by
current technology at the fundamental level. All of them have
workarounds (software discipline, careful design, manual
review). None of the workarounds are structural.

The laws were written by observing what BREAKS when these
principles are violated. Each violation was experienced
firsthand during the research. Each law is the fix for a
specific failure mode.

The engineering needs are not theoretical. They are the
documented requirements for making AI-assisted scientific
discovery safe and productive. Each one is a contribution
opportunity for engineers, researchers, and chip designers.

---

*11 laws. 11 gaps. 11 things to build.*
*None are solved at the hardware level today.*
*All have software workarounds. None of the workarounds scale.*
*The laws describe the world we need. The gaps describe the*
*distance from here to there. The builds are the road.*
