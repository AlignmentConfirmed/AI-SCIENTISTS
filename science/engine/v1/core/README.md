# Core Engine v1 — The Recursive Convergence Algebra

## The shared mathematical foundation for all scientific disciplines.

---

## What This Engine Does

The engine takes a MEASUREMENT (encoded as a multi-dimensional
rational tensor called a Delta) and recursively converges it
toward its irreducible truth through exact arithmetic.

```
RAW MEASUREMENT (noisy, multi-dimensional)
    ↓ encode as Delta (antisymmetric rational tensor)
    ↓ propose an operator (a transformation hypothesis)
    ↓ apply the operator (conjugation: AΔA⁻¹)
    ↓ validate (did the measurement improve?)
    ↓ converge (project to irreducible form)
    ↓ repeat (next K-step — next orbit of the scientific method)
    ↓
IRREDUCIBLE FORM (the measurement converged to its minimum representation)
```

## The Core Components

### The Delta — The State

An antisymmetric rational tensor. For n entities with m
coordinate dimensions:
- Δ_{ij} ∈ Q^m (exact rational, never floating point)
- Δ_{ij} = -Δ_{ji} (antisymmetric — the relationship from
  i to j is the negation of j to i)
- Δ_{ii} = 0 (self-relationship is zero)

The Delta encodes RELATIONSHIPS between entities, not the
entities themselves. The entities are defined entirely by their
relationships to each other.

### The Convergence — coboundary_reduce

The fundamental operation. Takes a noisy Delta and projects it
onto its irreducible form (the H^1 cohomology class representative).

After convergence:
- All coordinate distortion is removed
- All entries are integers (denominator = 1)
- The result is the MINIMUM representation of the truth
- Two measurements that look different but encode the same
  structure converge to the same irreducible form

### The Measurement — coherence_functional

The L1 norm of the residual. Measures how much unresolved
structure remains in the Delta. Lower = closer to convergence.
Zero = fully resolved.

This is NOT L2 (no squaring). L1 preserves the structure of the
residual — you can read WHERE the unresolved information lives,
not just how much there is.

### The Residual — for_each_residual

Iterates every triple (i,j,k) in the Delta and computes the
residual R_{ijk,l}. The residual is the STRUCTURED INFORMATION
about what has not yet been resolved. It is signal, not noise.

### The Operators — Exact Group Elements

Transformations that act on the Delta through conjugation:
AΔA⁻¹. All operators are in GL_n+(Q) — the group of
n×n rational matrices with positive determinant. They preserve
the antisymmetric structure. They compose (if A and B are
operators, AB is an operator).

### The Validation — C7 Triple Lock

Every proposed transformation must pass three checks:
1. Productivity: did the coherence functional decrease? (T_k > 0)
2. Conservation: is the antisymmetric structure preserved?
3. Bounded magnitude: is the step size within safe limits?

Only proposals that pass ALL THREE are accepted. The K-index
advances. The Delta evolves. The orbit continues.

## Properties

| Property | Value |
|----------|-------|
| Arithmetic | Exact rational (num_rational::BigRational) — zero rounding |
| Convergence | Cohomological (projects to H^1 representative) |
| Measurement | L1 (preserves residual structure) |
| Operators | GL_n+(Q) — exact, invertible, determinant > 0 |
| Validation | Triple lock — productivity, conservation, bounded |
| Recursion | Structural termination — bad proposals are rejected by C7 |
| State | Antisymmetric rational tensor — Δ_{ij} = -Δ_{ji} |

## What This Engine Does NOT Do

- Does not access the network
- Does not read or write files (IO stripped in the release)
- Does not execute system commands
- Does not connect to any external service
- Does not contain any domain-specific knowledge (biology,
  chemistry, physics — those are in the applications)

The engine is PURE MATH. It takes a Delta in. It produces a
converged Delta out. What the Delta REPRESENTS is determined
by the application layer, not by the engine.

## For Researchers

To use this engine for your science:

1. ENCODE your measurement as a Delta (the application layer
   teaches you how for your specific field)
2. Let the engine recursively converge
3. READ the converged result — the irreducible truth of your
   measurement with all noise removed
4. READ the residual — the structured information about what
   has NOT yet been resolved (this tells you where to measure
   NEXT)

The engine does the math. You do the science.

## Dependencies

- `num-rational` — exact rational arithmetic
- `num-bigint` — arbitrary precision integers
- `num-traits` — numeric trait abstractions
- `tiny-keccak` — BLAKE3 hashing (for receipts/verification)

No other dependencies. No runtime. No framework. Just math.

---

*One engine. Exact arithmetic. Recursive convergence.*
*The engine does the math. You do the science.*
