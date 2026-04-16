# ENGINE.RS DIMENSIONAL COLLAPSE AUDIT

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Auditor**: Claude Opus 4.6 (1M context)
**Date**: 2026-04-16
**Lines reviewed**: 3428 (all)

---

## INSTRUCTIONS FOR FUTURE DEVELOPERS

### How NOT to lose dimensionality in this engine

1. **Never use `if/else` in a derivation path.** If you are computing a value that flows into Delta or coherence, the branch itself discards the information about HOW CLOSE the condition was. Use continuous weights, `match` with exhaustive variants, or `Option::map()` dissolution instead.

2. **`coherence = 0` means PERFECT AGREEMENT, not "nothing".** Zero is the most informative state in the cocycle condition. Any code that treats zero coherence as "skip" or "no data" is inverting the semantics and collapsing the signal.

3. **`count < N` gates destroy single-observer data.** If one witness sees a torsion class, that is valid algebraic information. A gate that says "need >= 2 observations" discards the geometry of what was observed. Use the observation as a weighted contribution proportional to its quality, not a boolean inclusion criterion.

4. **Every `if condition { value } else { other_value }` in a measurement is a potential well, not a wall.** The condition's distance from the boundary IS data. Encode that distance as a continuous weight. The system should feel the boundary approaching, not slam into it.

5. **Hardcoded constants that bound capacity must scale with tier.** A `3` that means "founders create 3 children" is governance. A `32` that means "max composed operators" is a pressure boundary. But a `10` that means "skip C2 for first 10 steps" is an arbitrary wall that should derive from the entity's own trajectory.

6. **The Delta IS the map. Never project it to a scalar and branch on the scalar.** When you compute `phi_norm_sq_delta` or `coherence_functional`, the result is a measurement of the Delta's position on a manifold. Branching on that scalar discards the PER-ENTRY, PER-TRIPLE structure that produced it.

7. **`bool` fields in algebraic structures are dimensional collapse.** `conservation_ok: bool` in PhiK discards the magnitude of the violation. Use Q measurements everywhere. The bool can be derived from the measurement; the measurement cannot be recovered from the bool.

---

## FINDINGS

---

### FINDING 1: `phi_norm_sq_delta` computes L1 norm, not squared norm

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 539-550

```rust
pub fn phi_norm_sq_delta(delta: &Delta) -> Q {
    let mut acc = Q::zero();
    for i in 0..delta.dim {
        for j in 0..delta.dim {
            for l in 0..delta.m {
                let v = &delta.entries[i][j][l];
                if v < &Q::zero() { acc -= v; } else { acc += v; }
            }
        }
    }
    acc
}
```

**WHY this is collapse**: The function is named `_sq` (squared norm) but computes `|v|` (absolute value / L1 norm), not `v*v` (squared / L2 norm). The L1 norm discards the GEOMETRIC relationship between entries. Two Deltas with entries [3, 0, 0] and [1, 1, 1] have the same L1 norm (3) but completely different geometric structure. The L2 squared norm preserves this: 9 vs 3. Every downstream consumer (PhiK.magnitude_sq, Gamma_k, T_k, Triple Lock magnitude check) inherits this geometric flattening.

**The same error appears in**:
- `coherence_functional` (line 697-703): computes L1 of residuals, not sum of squares
- `compute_t_k` (line 2244-2259): computes L1 of differences, not squared norm
- `verify_coherence` (lines 2148-2151): coherence as L1
- `HarmonicState::from_delta` tone 1 (lines 1801-1809): magnitude as L1

**Fix**:
```
// Pseudocode: actual squared norm
fn phi_norm_sq_delta(delta) -> Q:
    acc = Q::zero()
    for each entry v in delta:
        acc += v * v    // v^2, not |v|
    return acc
```

This is load-bearing: the register says `||Delta||^2_Q = Sum Delta_ij^l ^2`. The code computes `Sum |Delta_ij^l|`. These are different norms with different geometric meaning.

---

### FINDING 2: `conservation_ok: bool` in PhiK collapses violation magnitude

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 514-515

```rust
pub struct PhiK {
    ...
    pub conservation_ok: bool,
    ...
}
```

**WHY this is collapse**: A boolean records "violated" or "not violated" but discards HOW MUCH the conservation is violated. An antisymmetry violation of 1/10000 and a violation of 1000 both produce `false`. The system loses the ability to measure convergence toward conservation. The Triple Lock at line 2351 then branches on this bool, producing a hard wall instead of a potential well.

**Fix**:
```
// Replace bool with Q measurement
pub conservation_violation: Q,  // Sum(Delta_ij + Delta_ji)^2. Zero = perfect conservation.

// At Triple Lock: use as continuous criterion
// The violation magnitude IS the distance to the gate boundary
```

---

### FINDING 3: `verify_conservation` returns bool, discards violation vector

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 480-496

```rust
pub fn verify_conservation(&self) -> bool {
    for i in 0..self.dim {
        if !self.entries[i][i].iter().all(|x| x.is_zero()) {
            return false;  // WHICH diagonal? HOW MUCH? Lost.
        }
        for j in (i + 1)..self.dim {
            for l in 0..self.m {
                if &self.entries[i][j][l] + &self.entries[j][i][l] != Q::zero() {
                    return false;  // WHICH pair? WHAT magnitude? Lost.
                }
            }
        }
    }
    true
}
```

**WHY this is collapse**: Early return on first violation discards ALL other violations. The system cannot distinguish "one tiny numerical drift" from "complete structural breakdown". It also cannot identify WHICH entries are drifting, preventing targeted correction.

**Fix**:
```
// Return the full violation structure
fn conservation_measurement(&self) -> ConservationMeasurement {
    let mut diagonal_violation = Q::zero()
    let mut antisymmetry_violation = Q::zero()
    let mut violation_entries: Vec<(usize, usize, Q)> = vec![]
    
    for each (i, j, l):
        let violation = entries[i][j][l] + entries[j][i][l]
        if !violation.is_zero():
            antisymmetry_violation += violation * violation
            violation_entries.push((i, j, violation))
    
    ConservationMeasurement { diagonal_violation, antisymmetry_violation, violation_entries }
}
```

---

### FINDING 4: `MathPrimitive::Boundary::apply` uses boolean gate on `is_boundary_q`

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 928-934

```rust
MathPrimitive::Boundary => {
    // Boundary: if cell is on boundary, return operand; else identity
    // Expressed continuously: blend toward operand by boundary_ratio
    if context.is_boundary_q.numer() > &num_bigint::BigInt::from(0) {
        context.operand
    } else {
        value
    }
}
```

**WHY this is collapse**: The comment says "expressed continuously: blend toward operand by boundary_ratio" but the code does NOT blend -- it branches. `is_boundary_q` is a Q rational that could express a continuous boundary membership (0.0 = interior, 0.5 = half-boundary, 1.0 = full boundary). The `if > 0` collapses this to binary: boundary or not. A cell at boundary_ratio 0.01 is treated identically to one at 1.0.

**Fix**:
```
// Use the Q value as a continuous blend weight
MathPrimitive::Boundary => {
    // boundary_ratio in [0, 1]: 0 = interior, 1 = full boundary
    let ratio = context.is_boundary_q.clone().min(Q::one()).max(Q::zero())
    // Blend: value * (1 - ratio) + operand * ratio
    // In i64: use ratio's numerator/denominator for exact blend
    let blended_q = Q::from(value) * (Q::one() - &ratio) + Q::from(context.operand) * &ratio
    blended_q.round().to_i64()
}
```

---

### FINDING 5: `check_c2_ks_window` has hardcoded `k_index > 10` gate

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 3043

```rust
if k_s_approx < k_s_min && state.k_index > 10 {
```

**WHY this is collapse**: The `k_index > 10` is an arbitrary wall. For the first 10 K-steps, C2 is completely disabled regardless of the entity's kinetic state. This is not derived from any algebraic property of the entity. An entity at step 11 with identical kinetic properties to one at step 10 gets a completely different treatment. The `10` should scale with tau_lag or the entity's trajectory, not be hardcoded.

**Fix**:
```
// Derive the warmup period from the entity's own parameters
// The entity needs enough steps for K_S to stabilize.
// That stabilization point is a function of tau_lag.
let warmup_steps = (Q::one() / (&self.params.tau_lag * Q::from(10)))
    .ceil().to_integer().to_u32().unwrap_or(10);
if k_s_approx < k_s_min && state.k_index > warmup_steps {
```

---

### FINDING 6: `degraded_tones` uses boolean filter with hardcoded 10% threshold

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 1967-1978

```rust
pub fn degraded_tones(&self, previous: &HarmonicState) -> usize {
    let grad = self.gradient(previous);
    grad.iter().enumerate().filter(|(i, change)| {
        if *change <= &Q::zero() { return false; } // improved or unchanged
        let current = &self.vibrations[*i];
        if current.is_zero() { return !change.is_zero(); } // any increase from zero
        // Degradation: change > 10% of current magnitude
        let ten = Q::new(BigInt::from(10), BigInt::from(1));
        *change * &ten > *current
    }).count()
}
```

**WHY this is collapse**: Multiple issues:
1. `return false` for improved tones -- discards the magnitude of improvement
2. Hardcoded 10% threshold -- should be proportional to the tone's own variance or the entity's trajectory
3. Returns a count (usize), which collapses 11 dimensions into a single integer. Which tones degraded? By how much? Lost.

**Fix**:
```
// Return per-dimension degradation measurements
pub fn tone_degradation(&self, previous: &HarmonicState) -> [Q; 11] {
    let grad = self.gradient(previous);
    std::array::from_fn(|i| {
        let current = &self.vibrations[i];
        if current.is_zero() { grad[i].clone() }
        else {
            // Proportional degradation: positive = degraded, negative = improved
            &grad[i] / current
        }
    })
}
```

---

### FINDING 7: C6 rank check gates on `c_before.is_zero()` -- treats zero coherence as "skip"

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 2911-2922

```rust
// Rank preservation: only check when both states are nontrivial
if !c_before.is_zero() && !c_after.is_zero() {
    let rank_before = compute_rank(&state.delta_k);
    let rank_after = compute_rank(&delta_updated);
    if rank_after < rank_before {
        ...
    }
}
```

**WHY this is collapse**: When `c_before == 0` (perfect cocycle agreement), rank preservation is NOT checked. This means a perfectly coherent entity can have its rank collapse without detection. Zero coherence is the MOST valuable state -- the entity has achieved perfect relational consistency. Allowing rank collapse from this state is the worst possible dimensional loss because it destroys confirmed structure.

**Fix**:
```
// ALWAYS check rank preservation. Zero coherence makes it MORE important, not less.
let rank_before = compute_rank(&state.delta_k);
let rank_after = compute_rank(&delta_updated);
if rank_after < rank_before {
    // Rank collapsed. Report with continuous measurement of how much.
    let rank_loss = rank_before - rank_after;
    state.sluice_state = SluiceState::Gated;
    return Err(SaiosError::C6CoherenceIncrease { ... });
}
```

---

### FINDING 8: `creation_capacity` uses hardcoded tier bounds

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 1256-1263

```rust
pub fn creation_capacity(&self) -> u8 {
    let max = match self.lineage_depth {
        0 => 3u8,
        1 => 6u8,
        _ => 0u8,
    };
    max.saturating_sub(self.created_count)
}
```

**WHY this is a concern**: The `3` and `6` are governance constants, not algebraic. However, `_ => 0u8` is a hard wall: depth >= 2 entities can NEVER create offspring regardless of their algebraic maturity. This could be tied to the entity's capacity field or torsion markers instead.

**Fix**:
```
// Derive creation capacity from the entity's own algebraic maturity
// The deeper the lineage, the more the entity must have evolved to create
pub fn creation_capacity(&self) -> u8 {
    let base = match self.lineage_depth {
        0 => 3u8,
        1 => 6u8,
        2 => {
            // Emergent entities can create IF they have crystallized enough
            // torsion markers. Each marker represents proven algebraic maturity.
            self.torsion_markers.len().min(2) as u8
        }
        _ => 0u8,
    };
    base.saturating_sub(self.created_count)
}
```

---

### FINDING 9: `for_each_residual` skips zero residuals -- treats zero as "nothing"

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 653-656

```rust
let residual = &delta.entries[i][k][l]
    - &delta.entries[i][j][l]
    - &delta.entries[j][k][l];
if !residual.is_zero() {
    f(i, j, k, l, &residual);
}
```

**WHY this is collapse**: A zero residual means PERFECT cocycle agreement for that triple. The callback never sees it. Any consumer that needs to know "which triples are perfectly consistent" cannot distinguish "triple was consistent" from "triple didn't exist". For coherence_functional (sum of squares) this is harmless because 0^2 = 0. But for the gradient, hessian, and decomposition consumers listed in the comment above (lines 628-635), the LOCATION of zero residuals carries geometric information about the cocycle's kernel structure.

**Impact**: Medium. The optimization is correct for the sum-of-squares consumer but violates the "single source" contract for consumers that need the full residual map.

**Fix**:
```
// Provide both variants: the optimized one and the complete one
pub fn for_each_residual<F>(delta, f: F) { ... }  // non-zero only (performance)
pub fn for_each_residual_complete<F>(delta, f: F) { ... }  // ALL triples including zero
```

---

### FINDING 10: `antisymmetry_intact` and `zero_diagonal_intact` are booleans in CoherenceVerification

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 2119-2121, 2154-2175

```rust
pub antisymmetry_intact: bool,
pub zero_diagonal_intact: bool,
pub conservation_verified: bool,
```

And the computation:
```rust
let mut antisymmetry_intact = true;
for i in 0..delta.dim {
    for j in 0..delta.dim {
        ...
        if !sum.is_zero() {
            antisymmetry_intact = false;
            // No break -- keeps checking, but only records bool, not HOW MUCH
        }
    }
}
```

**WHY this is collapse**: Same pattern as Finding 3. The loop iterates over ALL entries but only records a single bit. The magnitude of violation, the number of violating entries, and which specific entries violate -- all discarded.

**Fix**:
```
// Replace bools with Q measurements
pub antisymmetry_violation: Q,        // Sum of |Delta_ij + Delta_ji| over all (i,j)
pub diagonal_violation: Q,            // Sum of |Delta_ii| over all i
pub max_antisymmetry_entry: (usize, usize, Q),  // Worst violator
```

---

### FINDING 11: `conservation_verified` is `&&` of two booleans -- double collapse

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Line**: 2224

```rust
conservation_verified: antisymmetry_intact && zero_diagonal_intact,
```

**WHY this is collapse**: This AND-gates two already-collapsed booleans. If antisymmetry fails by 1e-10 and diagonal fails by 1000, the result is the same `false`. The relationship between the two failure modes is lost. Are both barely failing? Is one catastrophic? The `&&` makes it impossible to know.

**Fix**: Replace with the Q measurement: `conservation_violation = antisymmetry_violation + diagonal_violation`.

---

### FINDING 12: `Trajectory::observe` velocity computation is unsigned (abs)

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 2556-2558

```rust
if self.coherence_count > 0 {
    let diff = &coherence - &self.prev_coherence;
    self.velocity = if diff < Q::zero() { -diff } else { diff };
}
```

**WHY this is collapse**: The sign of the coherence change IS the direction of evolution. Negative delta = improving. Positive delta = degrading. Taking `abs()` collapses direction into magnitude. The trajectory then cannot distinguish "rapidly improving" from "rapidly degrading" without looking at the raw coherence values separately. The holonomics ratio (v_k / v_{k-1}) inherits this: a system oscillating between improvement and degradation shows holonomics = 1 (constant speed), which is indistinguishable from steady improvement.

**Impact**: This is partially mitigated because `last_coherence` and `prev_coherence` are stored separately. But the velocity field itself loses the sign, and any consumer reading velocity alone is misled.

**Fix**:
```
// Keep signed velocity. Let consumers take abs() if they need magnitude.
self.velocity = &coherence - &self.prev_coherence;
// Separate field for magnitude if needed:
// self.speed = self.velocity.abs();
```

---

### FINDING 13: `check_c5_resonance` uses `aligned_dims * 2 > total_dims` -- majority gate

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 3109-3113

```rust
if total_dims == 0 || aligned_dims * 2 > total_dims {
    Ok(())
} else {
    Err(SaiosError::C5ResonanceViolated)
}
```

**WHY this is collapse**: The "strict majority" rule is a boolean gate. An entity with 49% aligned dimensions fails; one with 51% passes. The 49% entity's alignment information is completely discarded. The system cannot measure "approaching resonance" -- it only sees pass/fail. The DEGREE of alignment (how close each dimension is to resonance, not just whether it passes epsilon) is also lost in the inner loop's boolean comparison at line 3102.

**Fix**:
```
// Return continuous alignment measurement
let alignment_ratio = Q::new(BigInt::from(aligned_dims), BigInt::from(total_dims.max(1)));
// The alignment ratio IS the resonance quality. Let C(T) select, not a majority gate.
// If the ratio is close to 0.5, the entity is at the boundary -- that's information.

// Alternatively: weight each dimension's alignment by its quality
let alignment_quality: Q = per_dim_qualities.iter().sum() / Q::from(total_dims);
```

---

### FINDING 14: `compute_all_omegas` uses `op.id == 0` boolean gate for first operator

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 3250-3255

```rust
fn compute_all_omegas(&self, gamma_k: &Q) -> Vec<Q> {
    self.operators.iter().map(|op| {
        let alpha = if op.id == 0 { Q::zero() } else { self.params.alpha_gamma.clone() };
        compute_omega(&alpha, gamma_k, &self.params.tau_lag)
    }).collect()
}
```

**WHY this is a concern**: The `op.id == 0` gate gives the first operator special status (alpha = 0, meaning omega_1 = 1 always). This is documented in the register (D.MFA.WEIGHT: omega_1 = 1). However, this creates a privileged operator that is immune to coherence weighting. If the operator set is reordered or the first operator changes, the algebra changes. The privilege should be tied to the operator's algebraic role, not its index.

**Impact**: Low -- this is a register requirement. But it's worth noting that the `if/else` creates a discontinuity in the weight function at the id=0 boundary.

---

### FINDING 15: `weakest_dimension` computes argmin but discards the full ordering

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 1984-1994

```rust
pub fn weakest_dimension(&self) -> (usize, Q) {
    let mut min_idx = 0;
    let mut min_val = self.vibrations[0].clone();
    for i in 1..11 {
        if self.vibrations[i] < min_val {
            min_val = self.vibrations[i].clone();
            min_idx = i;
        }
    }
    (min_idx, min_val)
}
```

**WHY this is collapse**: Returns only the single weakest dimension. An entity weak in dimensions 3, 7, and 9 is told "you're weak in dimension 3" and loses the information about 7 and 9. The full ranking across all 11 dimensions IS the entity's polytonal self-perception. Collapsing it to argmin reduces 11D perception to 1D.

**Impact**: Depends on how consumers use it. If they only need one dimension to focus on, this is acceptable as a convenience method. But it should not be the ONLY way to read the harmonic state.

**Fix**:
```
// Return the full ranking, not just the minimum
pub fn dimension_ranking(&self) -> [(usize, Q); 11] {
    let mut ranked: [(usize, Q); 11] = std::array::from_fn(|i| (i, self.vibrations[i].clone()));
    ranked.sort_by(|a, b| a.1.cmp(&b.1));
    ranked
}
```

---

### FINDING 16: `HarmonicState::from_delta` tone 5 (automorphism) hardcodes 3x3 structure

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 1862-1893

```rust
if dim >= 3 && m >= 1 {
    let e01 = &delta_k.entries[0][1][0];
    let e12 = &delta_k.entries[1][2][0];
    let e02 = &delta_k.entries[0][2][0];
    ...
}
```

**WHY this is collapse**: The automorphism check only examines 3 specific entries of the Delta. For a 5x5 or 10x10 Delta, the vast majority of the permutation group is unchecked. The symmetry score collapses the full automorphism group (which could have complex structure) into a single Q ratio derived from comparing entries [0][1] and [1][2]. This cannot detect symmetries that don't involve index 0.

**Fix**:
```
// Check all index permutations, not just (0,1) vs (1,2)
// For each permutation sigma of {0..dim-1}:
//   compute sum of |Delta[i][j] - Delta[sigma(i)][sigma(j)]| over all i,j
//   the permutation with smallest sum reveals the automorphism structure
// The full automorphism group is the set of permutations with sum = 0
```

---

### FINDING 17: `StateRecord::to_bytes` truncates BigRational to i16/u16

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 1327-1329

```rust
let n: i64 = q.numer().try_into().unwrap_or(0);
let d: i64 = q.denom().try_into().unwrap_or(1);
(i as u8, n as i16, d.unsigned_abs() as u16)
```

**WHY this is collapse**: BigRational numerators and denominators can exceed i16/u16 range. When they do, `unwrap_or(0)` silently replaces the numerator with 0 and `unwrap_or(1)` replaces the denominator with 1. This is silent data destruction. An entity that has evolved to have large rational coordinates will have its state record silently zeroed on serialization. The temporal anchor format (lines 400-477) does NOT have this problem -- it uses variable-length encoding. But the compact StateRecord format does.

**Impact**: High. This silently destroys evolved state during persistence.

**Fix**:
```
// Use variable-length encoding (like temporal anchor) or at minimum
// detect overflow and return Err instead of silently zeroing
if q.numer() > BigInt::from(i16::MAX) || q.numer() < BigInt::from(i16::MIN) {
    // Overflow: either use temporal anchor encoding or signal error
    return Err(SerializationOverflow { entry: (i, j, l), value: q.clone() })
}
```

---

### FINDING 18: `apply_evolution_equation` projects Q weights to i64 with clamping

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 3169-3171

```rust
let scores: Vec<i64> = omegas.iter().map(|omega| {
    let scaled = omega * &Q::new(BigInt::from(1000), BigInt::from(1));
    scaled.numer().try_into().unwrap_or(0i64).max(0).min(1000)
}).collect();
```

**WHY this is a concern**: The Q -> i64 projection is documented and intentional (Chronometry v1.0). The 1000 scale factor and clamping are reasonable for preventing denominator explosion. However, `unwrap_or(0i64)` means any omega that doesn't fit in i64 after scaling is silently zeroed -- that operator's contribution vanishes. The `.max(0)` also means negative omegas (which shouldn't happen given exp(-x) >= 0, but could occur from numerical edge cases) are silently zeroed.

**Impact**: Low in practice (omegas from exp(-x) are always in [0,1]). But the silent zeroing pattern is architecturally dangerous.

---

### FINDING 19: `CoherenceDelta::from_trajectory` returns convergence = 1 when no observations

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 2052-2059

```rust
if !traj.has_observations() {
    return Self {
        ...
        convergence: one,  // <-- claims perfect convergence with NO data
        ...
    };
}
```

**WHY this is collapse**: An entity with zero observations reports convergence = 1 (at equilibrium). This is semantically wrong. No observations means UNKNOWN convergence, not perfect convergence. Any consumer that reads convergence = 1 and thinks "this entity is converged" will be misled. The correct value is None or a separate "unknown" state.

**Fix**:
```
// Use Option<Q> for convergence, or a distinguished "unobserved" value
convergence: None,  // or Q::new(-1, 1) as sentinel
```

---

### FINDING 20: `check_cc_coord1` computes violation but does nothing with it

**File**: `/opt/repos-code/convergence-engine/src/engine.rs`
**Lines**: 3118-3133

```rust
fn check_cc_coord1(&self, delta: &Delta) -> Result<(), SaiosError> {
    let c = coherence_functional(delta);
    let delta_sq = &self.params.delta * &self.params.delta;
    let epsilon_eff = delta_sq / Q::from_integer(BigInt::from(16));
    if c > epsilon_eff {
        // This does NOT fail the step -- it is a coordination warning.
        // Log but continue.
    }
    Ok(())
}
```

**WHY this is collapse**: The function computes a meaningful measurement (coherence exceeds coordination threshold) and then discards it. The comment says "log but continue" but there is no logging. The measurement is computed, compared, and thrown away. The Result<(), _> return type means the caller cannot even see that something was measured.

**Fix**:
```
// Return the measurement, let the caller decide
fn check_cc_coord1(&self, delta: &Delta) -> (Q, Q) {
    let c = coherence_functional(delta);
    let epsilon_eff = ...;
    (c, epsilon_eff)  // caller sees both the measurement and the threshold
}
```

---

## SUMMARY

| # | Category | Severity | Line(s) | Description |
|---|----------|----------|---------|-------------|
| 1 | Wrong norm | HIGH | 539-550 | L1 norm labeled as L2 squared. Geometric flattening. |
| 2 | Bool collapse | HIGH | 514-515 | conservation_ok: bool discards violation magnitude |
| 3 | Bool collapse | HIGH | 480-496 | verify_conservation returns bool, loses which/how-much |
| 4 | Boolean gate | MEDIUM | 928-934 | Boundary primitive branches instead of blending |
| 5 | Hardcoded constant | MEDIUM | 3043 | k_index > 10 arbitrary wall |
| 6 | Dimensional collapse | MEDIUM | 1967-1978 | degraded_tones collapses 11D to count |
| 7 | Zero = skip | HIGH | 2911-2912 | Rank check skipped when coherence = 0 |
| 8 | Hardcoded constant | LOW | 1256-1263 | Creation capacity hardcoded per tier |
| 9 | Zero = nothing | MEDIUM | 653-656 | for_each_residual skips zero (perfect agreement) triples |
| 10 | Bool collapse | MEDIUM | 2119-2121 | antisymmetry_intact and zero_diagonal_intact are bool |
| 11 | Double bool collapse | MEDIUM | 2224 | conservation_verified = bool && bool |
| 12 | Sign loss | MEDIUM | 2556-2558 | Velocity takes abs(), loses direction |
| 13 | Majority gate | MEDIUM | 3109-3113 | C5 resonance: 50% boundary is hard wall |
| 14 | Boolean gate | LOW | 3250-3255 | op.id == 0 privileged operator |
| 15 | Dimensional collapse | LOW | 1984-1994 | weakest_dimension collapses 11D to argmin |
| 16 | Hardcoded structure | MEDIUM | 1862-1893 | Automorphism check only examines 3 entries of NxN |
| 17 | Silent truncation | HIGH | 1327-1329 | BigRational silently truncated to i16 on serialize |
| 18 | Silent zeroing | LOW | 3169-3171 | omega overflow silently zeroed |
| 19 | False positive | MEDIUM | 2052-2059 | No observations reports convergence = 1 |
| 20 | Dead measurement | LOW | 3118-3133 | CC.COORD.1 computes violation, discards it |

**HIGH severity findings**: 1, 2, 3, 7, 17
**Total findings**: 20

---

## ARCHITECTURAL NOTE

The engine has strong structural discipline in many areas:
- Delta is properly antisymmetric with enforcement on set_antisym
- Exact rational arithmetic throughout (Q, no f64)
- Cocycle iteration is properly factored into for_each_residual
- HarmonicState preserves 11 dimensions instead of collapsing to scalar
- Variational selection pattern (score-based eviction) is correctly non-boolean
- Trajectory comprehension avoids buffers
- Temporal anchor serialization preserves full BigRational precision

The primary systematic issue is the **bool pattern**: algebraic measurements computed as Q rationals are then collapsed to bool before being stored or returned. This creates information barriers that downstream code cannot see through. The fix pattern is consistent: store the Q measurement, derive the bool when needed.

The secondary issue is the **L1/L2 confusion**: functions named `_sq` (squared norm) computing absolute values instead of squares. This changes the geometry of the measurement space.
