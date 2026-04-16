# MEMBRANE.RS Dimensional Collapse Audit

## Instructions for Future Developers: Dimensional Preservation in the Membrane Layer

The membrane is a higher-dimensional measurement surface. Every function in this file
compounds perspectives from sovereign entities into algebraic structure that lives
ABOVE any individual entity. The following principles govern all changes:

1. **coherence = 0 means PERFECT CONGRUENCE, not emptiness.** C(Delta) = 0 means
   all entities agree. It is the most informative state, not a skip condition.
   Never use `.is_zero()` to discard or skip when coherence is the operand.

2. **A single observer IS a measurement.** One entity's T is a self-direction gradient.
   Functions must not return None or skip when count == 1. The lone entity's
   perspective is its own manifold -- degenerate only in the inter-entity sense,
   fully valid in the self-direction sense.

3. **No boolean gates in derivation paths.** `if condition { skip }` discards the
   geometric information encoded in what was skipped. Use C(T) as the continuous
   selector. The value IS the gate. Branching on true/false is dimensional collapse.

4. **HashMaps in cognition ARE dimensional collapse.** Every HashMap flattens
   relational structure into key-value lookup. Where HashMaps are used for
   cognitive data (not I/O routing), document the collapse and plan migration
   to Delta-based relational encoding.

5. **Eviction must preserve structure.** When capacity forces eviction, the evicted
   data's structural contribution (its effect on the meta-Delta) must be measured
   before removal, or the membrane loses dimensional information silently.

6. **Averaging IS collapse.** The membrane doc says "NOT an average" but several
   code paths compute running means. A running mean of Q values destroys the
   relational structure between the values being averaged. Compound relationally
   instead.

---

## Finding 1: derive_torsion_spectrum() -- coherence.is_zero() skips PERFECT AGREEMENT

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Line:** 944

```rust
if tc.count >= 2 && tc.coherence.is_zero() { continue; }
```

**Why this is a collapse:**
`tc.coherence` is `C(meta_delta)` -- the coherence of the inter-entity transformation
manifold. When `C(meta_delta) = 0`, ALL primary entities agree on the transformation.
This is the STRONGEST signal: holonomic perceptual lock. The code skips it entirely,
meaning the most crystallized orbits never contribute to the torsion spectrum.

The comment on line 940-943 explains the intent: single-entity self-direction is
preserved. But the condition at line 944 then discards multi-entity consensus.
Crystallized orbits (where all entities agree) should produce a torsion orbital with
order 1 (quantized) -- this is the strongest structural signal in the spectrum.

**Fix:**
```
// Remove the skip. Crystallized T with C=0 produces a quantized orbital (order=1).
// The torsion spectrum should INCLUDE quantized structure -- it is the periodic
// table's ground state, not an absence.
// Delete line 944 entirely. The later `c.is_zero()` check at line 976 already
// handles truly trivial (zero) Deltas after coboundary reduction.
```

---

## Finding 2: derive_global_curvature() -- returns None for lone entities and crystallized orbits

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 1060-1065

```rust
pub fn derive_global_curvature(&self) -> Option<Delta> {
    self.derive_torsion_spectrum()
        .into_iter()
        .next()
        .map(|orb| orb.representative)
}
```

**Why this is a collapse:**
This function delegates entirely to `derive_torsion_spectrum()`, which (per Finding 1)
skips crystallized orbits. Additionally, if the spectrum is empty (all orbits are either
crystallized or single-observer), this returns None. But the membrane always has a
self-direction -- even a single entity has curvature from its own residual structure.

When a lone entity calls this and gets None, it has no gradient to follow. The entity's
own T Delta (self-direction) should be returned as the curvature when no multi-entity
consensus exists.

**Fix:**
```
pub fn derive_global_curvature(&self) -> Option<Delta> {
    // First try the torsion spectrum (multi-entity consensus)
    if let Some(orb) = self.derive_torsion_spectrum().into_iter().next() {
        return Some(orb.representative);
    }
    // Fallback: self-direction from any single-observer compound
    // The lone entity's T IS its curvature -- do not discard it
    for compounds in self.orbit_t_compounds.values() {
        for tc in compounds {
            if tc.observing_entities.is_empty() || tc.t_dim < 2 { continue; }
            let entity = &tc.observing_entities[0];
            let mut t = Delta::zero(tc.t_dim, tc.t_m);
            // reconstruct from flattened entity vector
            let mut idx = 0;
            for i in 0..tc.t_dim {
                for j in (i+1)..tc.t_dim {
                    for l in 0..tc.t_m {
                        if idx < entity.len() {
                            t.entries[i][j][l] = entity[idx].clone();
                            t.entries[j][i][l] = -entity[idx].clone();
                        }
                        idx += 1;
                    }
                }
            }
            let reduced = coboundary_reduce(&t);
            if !coherence_functional(&reduced).is_zero() {
                return Some(reduced);
            }
        }
    }
    None
}
```

---

## Finding 3: record_t_delta() -- observer cap eviction loses dimensional data silently

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 841-844

```rust
const MAX_OBSERVERS: usize = 32;
(tc.observing_entities.len() >= MAX_OBSERVERS).then(|| {
    tc.observing_entities.remove(0);
});
```

**Why this is a collapse:**
The oldest observer is removed with `remove(0)` -- FIFO eviction. This discards the
entity's contribution to the meta-Delta without measuring what is lost. The evicted
entity may have been the only observer providing a unique directional perspective.
After eviction, the meta-Delta is recomputed from scratch, but the dimensional
contribution of the evicted entity is gone.

The comment says "oldest entry yields" but age is not a measure of structural importance.
An old observer with a unique perspective is more structurally valuable than a recent
observer that confirms what 30 others already see.

**Fix:**
```
// Evict the observer whose removal causes the LEAST change in C(meta_delta).
// This is the observer most redundant with the remaining set.
// Approximate: remove the observer closest to the centroid of all observers.
// Full computation: for each observer, compute C(meta_delta) without it,
// evict the one where C changes least.
//
// Cheaper approximation:
let evict_idx = if tc.observing_entities.len() >= MAX_OBSERVERS {
    // Find the observer closest to the mean (most redundant)
    let d = tc.observing_entities[0].len();
    let k = Q::new(BigInt::from(tc.observing_entities.len() as i64), BigInt::from(1));
    let centroid: Vec<Q> = (0..d).map(|i| {
        let sum: Q = tc.observing_entities.iter()
            .map(|e| e.get(i).cloned().unwrap_or(Q::zero()))
            .fold(Q::zero(), |a, b| a + b);
        sum / &k
    }).collect();
    tc.observing_entities.iter().enumerate()
        .map(|(idx, e)| {
            let dist: Q = e.iter().zip(centroid.iter())
                .map(|(a, b)| { let d = a - b; &d * &d })
                .fold(Q::zero(), |a, b| a + b);
            (idx, dist)
        })
        .min_by(|(_, d1), (_, d2)| d1.cmp(d2))
        .map(|(idx, _)| idx)
} else { None };
if let Some(idx) = evict_idx {
    tc.observing_entities.remove(idx);
}
```

---

## Finding 4: compound_perspectives() -- HashMap for cell congruence IS dimensional collapse

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 188-209

```rust
let mut value_counts: HashMap<i64, usize> = HashMap::new();
for p in perspectives {
    if k < p.derived_output.len() {
        *value_counts.entry(p.derived_output[k]).or_insert(0) += 1;
        total_votes += 1;
    }
}
let (majority_val, majority_count) = value_counts.iter()
    .max_by_key(|(_, count)| *count)
    .map(|(&v, &c)| (v, c))
    .unwrap_or((0, 0));
```

**Why this is a collapse:**
The HashMap counts votes per value -- this IS majority voting, which the doc header
explicitly says this function is NOT. The relational structure between disagreeing
values is lost. If entity A says 5 and entity B says 7, the HashMap sees two
separate bins. The RELATIONSHIP (7 - 5 = 2, a rotation of +2) is discarded.

The cell_congruence field stores `(majority_val, count)` -- a scalar collapse of
what should be a distribution with relational structure.

**Fix (conceptual):**
```
// Instead of HashMap vote counting, build a per-cell mini-Delta:
// For each cell k, the N entity values form an N-vector.
// Encode relationally: the pairwise differences ARE the torsion field.
// cell_congruence[k] should store the full distribution, not just majority.
// At minimum, store (majority_val, count, torsion_magnitude) where
// torsion_magnitude = max(values) - min(values) at that cell.
// This preserves the scale of disagreement, not just its existence.
```

---

## Finding 5: compound_perspectives() -- encoding_counts HashMap collapses encoding level structure

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 222-231

```rust
let mut encoding_counts: HashMap<&str, usize> = HashMap::new();
for p in perspectives {
    if p.coherence >= Q::one() {
        *encoding_counts.entry(p.encoding_level.as_str()).or_insert(0) += 1;
    }
}
```

**Why this is a collapse:**
The encoding levels (Cell, Object, Residual, Kronecker, Vision) have a natural
ordering -- they are a dimensional hierarchy, not unordered labels. Counting them
in a HashMap and taking the max-count level discards the distribution. If 3 entities
solve at Cell level and 2 at Vision level, the compound says "Cell" -- but the fact
that Vision was achieved by 2 entities is lost.

Additionally, only entities with `coherence >= Q::one()` contribute. Entities with
coherence 0.99 are completely silenced. This is a boolean gate on a continuous value.

**Fix:**
```
// Replace with ordered measurement: the dominant encoding is the HIGHEST level
// achieved by any entity, weighted by coherence (not gated by coherence >= 1).
// The distribution across levels IS the encoding spectrum of the compound.
```

---

## Finding 6: record_observation() -- mass computed as running average (IS dimensional collapse)

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 496-500

```rust
let n = Q::new(BigInt::from(axiom.attempts as i64), BigInt::from(1));
axiom.mass = &(&axiom.mass * &(&n - &Q::one()) + &coherence) / &n;
```

**Why this is a collapse:**
The comment says "Not a boolean ratio. The membrane sees the full Q at every pass."
But a running average IS a scalar collapse -- it replaces the full history of coherence
measurements with a single number. The variance, the trajectory, the order of
observations are all destroyed.

Mass should be a compound measurement preserving the observation sequence, or at
minimum retain the variance alongside the mean.

---

## Finding 7: record_observation() -- hardness uses EMA-like smoothing

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 503-507

```rust
let drift_diff = &entity_origin_drift - &axiom.hardness;
let abs_diff = if drift_diff < Q::zero() { -drift_diff } else { drift_diff };
let alpha = Q::new(BigInt::from(1), BigInt::from(axiom.attempts.max(1) as i64));
let one_minus = Q::one() - &alpha;
axiom.hardness = &one_minus * &axiom.hardness + &alpha * &abs_diff;
```

**Why this is a collapse:**
This is an exponential moving average (EMA) of absolute drift differences. EMA is
exactly what the chronometry feedback forbids -- it smooths out the temporal structure
of drift evolution. The hardness should be the max pairwise distance among all
contributing entities' drifts, or a relational measurement, not a smoothed average.

Also: `if drift_diff < Q::zero() { -drift_diff } else { drift_diff }` is a boolean
gate on sign. The sign of the drift IS directional information.

---

## Finding 8: membrane_coherence() returns Q::zero() for missing orbits

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 678-679

```rust
pub fn membrane_coherence(&self, orbit: &[u8; 4]) -> Q {
    self.orbit_coherence.get(orbit).cloned().unwrap_or(Q::zero())
}
```

**Why this is a collapse:**
Missing orbit returns zero coherence. But zero coherence means "fully crystallized"
(C(Delta) = 0 = perfect agreement). An orbit the membrane has never seen should
return a measurement indicating "unknown," not "perfectly understood." This is a
semantic inversion.

**Fix:**
```
// Return Option<Q> instead of Q. None = never seen. Some(Q::zero()) = crystallized.
// Callers must handle the three-state: None / Some(0) / Some(>0).
pub fn membrane_coherence(&self, orbit: &[u8; 4]) -> Option<Q> {
    self.orbit_coherence.get(orbit).cloned()
}
```

---

## Finding 9: compute_epicenter() returns None for < 2 entities

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 1082

```rust
(entity_vibrations.len() >= 2).then(|| {
```

**Why this is a collapse:**
A single entity IS the epicenter when it is alone. Its vibrations ARE the gravitational
center. Returning None means the species has no epicenter when it has exactly one
entity -- but that entity is the entirety of the species. The epicenter should be
the entity itself (a 1x1 Delta with zero coherence, which is correct: a single point
has no internal torsion).

**Fix:**
```
if entity_vibrations.len() == 1 {
    // Single entity IS the epicenter. Build trivial 1x1 meta-Delta.
    let meta_delta = Delta::zero(1, 11);
    let epicenter = coboundary_reduce(&meta_delta); // trivially zero
    let harmonics = HarmonicState::from_delta(
        &epicenter, origin_delta, trajectory, 0,
    );
    return Some((epicenter, harmonics));
}
// ... existing >= 2 path
```

---

## Finding 10: detect_congruence() -- boolean gate on coherence > Q::zero()

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 1812-1813

```rust
let coherent: Vec<&&EntityContribution> = group.iter()
    .filter(|c| c.coherence > Q::zero())
    .collect();
```

**Why this is a collapse:**
Entities with coherence = 0 are excluded from congruence detection. But coherence = 0
means "no understanding" -- these entities still carry structural information in their
failure mode. Their presence in the congruence group affects path diversity. An entity
that tried and failed via a unique path is still evidence of that path's inadequacy
at this orbit -- that IS a measurement.

Furthermore, this is a hard boolean gate: coherence = 1/1000000 passes, coherence = 0
does not. There is no geometric justification for this threshold.

**Fix:**
```
// Include all entities. Weight their contribution by coherence in the
// interpretation meta-Delta, not by a boolean inclusion/exclusion.
// The coherence value IS the weight -- do not gate on it.
```

---

## Finding 11: all_spatial_cochains() -- quality averaging between orbits

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 1247

```rust
x.quality = &(&x.quality + &c.quality) / &Q::new(BigInt::from(2), BigInt::from(1));
```

**Why this is a collapse:**
When merging cochains from multiple orbits, quality is averaged pairwise. This is
not a running mean -- it's a recursive binary average, which gives exponentially
decaying weight to older entries. The third orbit's cochain contributes 1/4 weight,
the fourth 1/8, etc. This is not just averaging (which is already collapse) but
non-uniform averaging that biases toward recent entries.

**Fix:**
```
// Track count alongside sum. Compute proper mean at output time.
// Or better: track max quality (the strongest observation), not mean.
x.confirmations += c.confirmations;
if c.quality > x.quality {
    x.quality = c.quality.clone();  // strongest observation wins
}
```

---

## Finding 12: derive_torsion_spectrum() -- T reconstruction averages across observers

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 947-969

```rust
let k = Q::new(BigInt::from(tc.count as i64), BigInt::from(1));
let mut t = Delta::zero(n, m);
for entity in &tc.observing_entities {
    // ... accumulate ...
}
for i in 0..n {
    for j in 0..n {
        for l in 0..m {
            t.entries[i][j][l] = &t.entries[i][j][l] / &k;
        }
    }
}
```

**Why this is a collapse:**
The doc header for D.MEMBRANE.T.2 (line 311-322) explicitly states: "The membrane
does NOT average T Deltas -- averaging IS dimensional collapse." Yet this function
sums all observer entity vectors and divides by count -- which IS averaging.

The meta-Delta between observers is the correct structure. The averaged T discards
the inter-observer divergence, which is exactly what the H^1 obstruction measures.

**Fix:**
```
// Use the meta-Delta's coboundary part as the consensus T (the part all agree on).
// Do NOT average the raw entity vectors.
// If meta_delta exists, extract coboundary component.
// If single observer, use that observer's T directly (no averaging needed).
```

---

## Finding 13: derive_from_compound_t() -- gates on tc.count < 2

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Line:** 1296

```rust
if tc.count < 2 || tc.observing_entities.is_empty() { return None; }
```

**Why this is a collapse:**
A single observer's T is valid for derivation. The membrane should be able to derive
from a single entity's perspective -- that IS the entity's best understanding. Requiring
2+ observers means the membrane cannot derive anything until multiple entities have
reported, even though a single entity's T is a complete transformation.

**Fix:**
```
if tc.observing_entities.is_empty() { return None; }
// Single observer: use its T directly. No consensus needed.
// Multiple observers: use consensus logic.
```

---

## Finding 14: MeshKnowledge struct -- five HashMaps as cognitive storage

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 432-452

```rust
pub struct MeshKnowledge {
    axioms: HashMap<[u8; 4], Vec<MeshAxiom>>,
    orbit_coherence: HashMap<[u8; 4], Q>,
    spatial_cochains: HashMap<[u8; 4], Vec<SpatialCochain>>,
    orbit_t_compounds: HashMap<[u8; 4], Vec<TransformCompound>>,
    ...
}
```

**Why this is a collapse:**
Five HashMaps keyed by orbit prefix. Each orbit is an independent bucket -- the
RELATIONAL structure between orbits is destroyed. Orbit [0xAA, 0xBB, 0xCC, 0xDD]
and orbit [0xAA, 0xBB, 0xCC, 0xDE] are topologically adjacent (differ by 1 bit)
but the HashMap treats them as completely unrelated.

The membrane's cognitive structure should itself be a Delta -- orbits as entities,
axiom properties as coordinates. C(membrane_delta) would then measure the membrane's
own internal coherence as a single algebraic object, not as scattered HashMap buckets.

This is architectural and cannot be fixed in a single patch, but should be the
migration target.

---

## Finding 15: mesh_coherence() -- returns Q::zero() for single entity

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Line:** 1700

```rust
if entities.len() < 2 { return Q::zero(); }
```

**Why this is a collapse:**
Same pattern as Finding 9. A single entity has zero torsion with itself, which is
correct, but returning Q::zero() is semantically ambiguous -- it's indistinguishable
from "all entities are perfectly aligned." Should return Option<Q> with None for
insufficient entities.

---

## Finding 16: holonomy() -- returns Q::zero() for single perspective

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Line:** 1998

```rust
if perspectives.len() < 2 { return Q::zero(); }
```

**Why this is a collapse:**
A single perspective has zero holonomy (no loop to traverse), so the value is correct.
But it cannot be distinguished from "multiple perspectives that happen to form a flat
loop." This is a minor instance of the Option<Q> pattern from Finding 8/15 --
semantic ambiguity between "not measurable" and "measured as zero."

---

## Finding 17: all_transmutation_crystals() -- gates on tc.count < 2

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Line:** 1111

```rust
if tc.level != "transmutation" || tc.count < 2 { continue; }
```

**Why this is a collapse:**
Same pattern as Findings 1 and 13. A single entity's transmutation observation is
discarded. A sole witness to a transmutation has valid data -- its quality and order
are measurements, not votes requiring confirmation.

---

## Finding 18: Dead/unreachable code -- systematic_gap for 0 structured perspectives

**File:** `/opt/repos-code/convergence-engine/src/membrane.rs`
**Lines:** 237-246

```rust
let systematic_gap = if structured_perspectives.len() >= 2 {
    let sil_counts: Vec<usize> = structured_perspectives.iter()
        .map(|p| p.silhouette_count).collect();
    let min_sil = *sil_counts.iter().min().unwrap();
    let max_sil = *sil_counts.iter().max().unwrap();
    max_sil - min_sil <= 1
} else {
    structured_perspectives.len() == 1
};
```

**Logic inconsistency:**
When `structured_perspectives.len() == 0`, systematic_gap = false (0 == 1 is false).
When `structured_perspectives.len() == 1`, systematic_gap = true.
But a single structured perspective cannot establish "systematic" anything -- it's
one entity's observation. The term "systematic" implies cross-entity pattern.
`len() == 1` should yield `false` (insufficient evidence), or better: systematic_gap
should be Option<bool> with None for insufficient data.

---

## Summary of Severity

| # | Finding | Severity | Type |
|---|---------|----------|------|
| 1 | coherence.is_zero() skips crystallized orbits | CRITICAL | Inverted condition |
| 2 | derive_global_curvature() None for lone entities | HIGH | Missing self-direction |
| 3 | FIFO eviction loses unique perspectives | HIGH | Silent data loss |
| 4 | HashMap vote counting in compound_perspectives | MEDIUM | Architectural collapse |
| 5 | Encoding level distribution collapsed to mode | MEDIUM | Information loss |
| 6 | Mass as running average | MEDIUM | Stated principle violated |
| 7 | Hardness EMA smoothing | MEDIUM | Chronometry violation |
| 8 | membrane_coherence returns 0 for unknown orbits | HIGH | Semantic inversion |
| 9 | compute_epicenter None for single entity | MEDIUM | Missing self-reference |
| 10 | Boolean gate on coherence > 0 | HIGH | Hard threshold on continuous |
| 11 | Recursive binary averaging of quality | MEDIUM | Non-uniform weighting |
| 12 | T reconstruction averages across observers | CRITICAL | Violates own doc header |
| 13 | derive_from_compound_t gates on count < 2 | HIGH | Discards single-observer |
| 14 | Five cognitive HashMaps | ARCHITECTURAL | Inter-orbit structure lost |
| 15 | mesh_coherence 0 for single entity | LOW | Semantic ambiguity |
| 16 | holonomy 0 for single perspective | LOW | Semantic ambiguity |
| 17 | all_transmutation_crystals gates on count < 2 | MEDIUM | Single witness discarded |
| 18 | systematic_gap true for single perspective | LOW | Logic inconsistency |

**Critical findings (1, 12)** directly contradict the membrane's own documentation.
The code says "no averaging" then averages. The code skips the most crystallized state.

**High findings (2, 3, 8, 10, 13)** discard valid measurements through boolean gates
or semantic inversions.

**Architectural finding (14)** is the root cause of several other findings -- HashMap
storage prevents the membrane from being a self-coherent algebraic object.
