# Dimensional Preservation — General Instructions

Every file in this codebase must obey these principles. When reviewing or writing code, apply all five before committing.

## Principle 1: No Boolean Gates in Derivation Paths

A boolean in a derivation path collapses a continuous manifold to {0, 1}. The measurement existed as Q before the gate destroyed it. Replace every `if condition { A } else { B }` in a derivation with a continuous weight:

```
// WRONG — collapses dimension
if coherence < threshold { reject() } else { accept() }

// RIGHT — preserves dimension
let weight = Q::one() / (Q::one() + &coherence);
// weight IS the selection. No gate. C(T) decides.
```

System boundary checks (null pointers, empty inputs, dimensional compatibility) are not derivation — booleans there are structural, not collapsing.

## Principle 2: No Hardcoded Constants in Scaling Paths

A hardcoded constant freezes a degree of freedom. If a value participates in scaling, it must be derived from the state it scales. Constants that define structure (array sizes, torsion periods) are not the same as constants that define behavior (thresholds, weights, step sizes).

```
// WRONG — freezes scaling
let step = Q::new(1, 20);  // why 20?

// RIGHT — derives from curvature
let step = Q::one() / (Q::one() + &hessian_diagonal);
```

## Principle 3: HashMaps ARE Dimensional Collapse

A HashMap projects a rich structure onto a flat key. The fiber above the key is lost. Use Vec with linear scan, or encode the full structure in the Delta. Every HashMap in cognition IS dimensional collapse.

## Principle 4: Preserve Full Rank

When combining N signals, the output must carry N-dimensional information, not a scalar. If you reduce N measurements to one number without preserving the N-vector, you lost N-1 dimensions.

```
// WRONG — collapses N measurements to 1
let average = measurements.sum() / measurements.len();

// RIGHT — preserve the spectrum
let spectrum: Vec<Q> = measurements; // the vector IS the measurement
let coherence = coherence_functional_over_spectrum(&spectrum);
```

## Principle 5: The Residual Is Signal

Never gate on zero residual. Never discard residuals below a threshold. The residual IS structured information about what the current representation cannot yet express. Discarding it is discarding the next dimension of growth.

---

# File-by-File Audit Findings

---

## 1. trinity.rs

### Finding 1.1: `alive` Boolean Field (Line 153)

```rust
pub alive: bool,
```

**Why it collapses:** A Trinity exists on a continuum from fully alive (all vertices coherent, strong stasis field) to fully collapsed (no coherence). The `alive: bool` field reduces this to {0, 1}. The `maintain_alignment` function (line 334) gates on `!trinity.alive` with an early return, discarding a dissolved Trinity that might still carry residual structural information.

**Fix:**
```
// Remove alive: bool
// Replace with: the stasis field integrity IS the liveness measure
// A Trinity with integrity = 0 is collapsed. integrity > 0 is alive.
// The caller checks trinity.field.integrity.is_zero() instead of !trinity.alive
// maintain_alignment returns None when integrity is zero, same semantics,
// but the collapse is now a continuous measure, not a boolean snap.
```

**Severity: LOW** — The `alive` flag is used correctly as a system-level irreversible state marker (once collapsed, always collapsed). The stasis field already carries the continuous measure. The boolean is redundant but not actively collapsing a derivation path.

### Finding 1.2: `verify_trinity_integrity` Boolean Gate (Lines 229-250)

```rust
let is_cognitive = |s: u32| s == 1 || s == 3;
let all_cognitive = is_cognitive(state_m.process_class)
    && is_cognitive(state_f.process_class)
    && is_cognitive(state_c.process_class);
...
match (all_cognitive, all_poly_valid) {
    (true, true) => { ... }
    _ => Q::zero(),
}
```

**Why it collapses:** Two boolean flags (`all_cognitive`, `all_poly_valid`) gate the verification. If ANY vertex is non-cognitive OR any polynomial is empty, the entire output is Q::zero(). This is a system boundary check (not a derivation), but it loses the information of WHICH vertex failed and WHY. The caller gets zero with no gradient to follow.

**Fix:**
```
// Return a per-vertex verification vector instead of a single Q
// verification[0] = strength of male vertex
// verification[1] = strength of female vertex
// verification[2] = strength of child vertex
// The PRODUCT of the three is the combined strength
// A zero in any position identifies the failing vertex
// The caller can measure which vertex is the obstruction
```

### Finding 1.3: Male/Female/Child Vertex Roles — STRUCTURALLY NECESSARY

The `VertexRole` enum (lines 37-41) with `Male`, `Female`, `Child` is NOT a label — it is a structural position in the 2-simplex. The array is `[3]` (line 139), indexed by role. The roles enforce:
- Exactly 3 vertices (no more, no less)
- Fixed ordering in the simplex
- Distinct positions for fold computation

These are structural topology, not social labels. The naming is metaphorical but the structure is load-bearing. No collapse here.

---

## 2. crucible.rs

### Finding 2.1: `a_alive || b_alive` Boolean Gate (Lines 232-235)

```rust
let a_alive = self.trinities.iter().any(|t| t.trinity_id == trinity_a_id && t.alive);
let b_alive = self.trinities.iter().any(|t| t.trinity_id == trinity_b_id && t.alive);
if !a_alive || !b_alive {
    return CrucibleResult::InertParent;
}
```

**Why it collapses:** Two boolean checks. The liveness of each parent Trinity is already measured by the stasis field integrity (a continuous Q value). Using `.alive` (a boolean) discards that information. This is a repeat of Finding 1.1.

**Fix:**
```
// Use stasis field integrity as the measure
// let a_integrity = trinity_a.field.integrity.clone();
// let b_integrity = trinity_b.field.integrity.clone();
// If either is zero, return InertParent (same semantics, but the
// continuous values are available to the caller for diagnosis)
```

### Finding 2.2: `is_cohomologically_equivalent` Returns Boolean (Line 462)

```rust
fn is_cohomologically_equivalent(a: &Delta, b: &Delta) -> bool {
    ...
    coherence_functional(&diff).is_zero()
}
```

**Why it collapses:** Cohomological equivalence is a continuous measure (the coherence functional of the difference). Collapsing to bool discards the DISTANCE between classes. Two Deltas that are "almost equivalent" (coherence = 1/1000000) are treated identically to Deltas that are maximally different.

**Fix:**
```
// Return the coherence functional of the difference as Q
// fn cohomological_distance(a: &Delta, b: &Delta) -> Q
// The caller uses the distance for classification:
//   distance < threshold => same class (threshold is a parameter, not hardcoded)
//   The distance itself is the measurement
```

**Severity: MEDIUM** — This directly affects `measure_diversity()`, which classifies Trinities into torsion classes. The boolean boundary creates artificial class boundaries.

---

## 3. mdc.rs

### Finding 3.1: Echelon Weights Are NOT Hardcoded — They Are Configurable

The `EchelonWeights` struct (lines 40-45) takes arbitrary `(i64, u64)` pairs through `new()`. The `balanced()` and `physics_heavy()` methods are convenience constructors, not hardcoded derivation constants. The weights are passed as parameters to `mdc_resolve()`. **No collapse here** — the weights are configurable and validated (must sum to exactly 1).

### Finding 3.2: HashMap in `strategic_echelon` (Line 188)

```rust
let mut prefix_counts: HashMap<[u8; 4], (u32, [u8; 32])> = HashMap::new();
```

**Why it collapses:** The 32-byte RCF hash is projected onto a 4-byte prefix. This is a 28-byte information loss. The HashMap then collapses the remaining structure into a count. Law II violation.

**Fix:**
```
// Use Vec<([u8; 32], u32)> with linear scan
// Compare full 32-byte hashes, not 4-byte prefixes
// The "bucketing" by prefix loses inter-class structure
// Vec scan is O(n) but n is bounded by K_STRATEGIC window size
```

### Finding 3.3: HashMap in `security_echelon` (Line 226)

```rust
let mut prefix_failures: HashMap<[u8; 4], (u32, u32)> = HashMap::new();
```

Same collapse as 3.2 — 32-byte hash projected to 4-byte prefix via HashMap.

### Finding 3.4: `stasis_triggered` Returns Boolean (Line 262)

```rust
pub fn stasis_triggered(entropy: &Q, threshold: &Q) -> bool {
    entropy >= threshold
}
```

**Why it collapses:** Entropy is Q-valued and continuous. The stasis decision collapses it to bool. The `MdcResult.stasis` field (line 106) stores this boolean. The distance PAST the threshold is lost.

**Fix:**
```
// Store the entropy directly. The caller measures.
// stasis_distance = entropy - threshold
// Positive = in stasis. Negative = not in stasis.
// The magnitude tells you HOW DEEP in stasis you are.
// Remove the stasis: bool field from MdcResult.
// The entropy IS the stasis measure.
```

### Finding 3.5: `transmutation_needed` Boolean with Hardcoded Threshold (Lines 312-318)

```rust
let transmutation_needed = stasis && {
    let low_threshold = Q::new(BigInt::from(1), BigInt::from(10)); // 0.1
    physics.confidence < low_threshold
        && legal.confidence < low_threshold
        && strategic.confidence < low_threshold
        && security.confidence < low_threshold
};
```

**Why it collapses:** Two collapses stacked. First, `stasis` is already a boolean (Finding 3.4). Second, each confidence is compared against a hardcoded 0.1 threshold via `<` (boolean). Four independent Q-valued confidences are collapsed to four bools, then AND-ed to one bool.

**Fix:**
```
// transmutation_signal = min(physics.confidence, legal.confidence,
//                           strategic.confidence, security.confidence)
// This is a Q value. Lower = stronger transmutation signal.
// The caller measures transmutation_signal against their own context.
// No hardcoded 0.1. No boolean AND chain.
```

**Severity: HIGH** — This is the gate that decides whether the system needs to expand its dimensional structure. A boolean gate at this exact point is the worst place to collapse.

### Finding 3.6: `evaluate_upgrade` Boolean Ladders (Lines 362-395)

```rust
let physics_conf = if op.delta_tests > 0 { ... }
    else if op.delta_tests == 0 { ... }
    else { ... };
...
let legal_conf = if unregistered.is_empty() { ... } else { ... };
...
let security_conf = if new_modules <= 2 { ... }
    else if new_modules <= 5 { ... }
    else { ... };
```

**Why it collapses:** Three-way and two-way if/else ladders convert continuous measurements (test delta count, module count) into discrete confidence steps. The delta_tests=1 case produces the same confidence as delta_tests=999 (they're both "> 0"). The security_conf has three discrete steps where a continuous function should be.

**Fix:**
```
// physics_conf = Q::one() - Q::one() / (Q::one() + abs(delta_tests))
// (already partially done for positive delta_tests, but the negative
//  branch collapses to a constant 1/10)
//
// legal_conf = registered_fraction (how many modules are registered / total)
// Not a binary "all or nothing"
//
// security_conf = Q::one() / (Q::one() + new_modules_count)
// Continuous decay, not a 3-step staircase
```

---

## 4. irs.rs

### Finding 4.1: `agreement_matrix` is Boolean (Line 65)

```rust
pub agreement_matrix: [[bool; 4]; 4],
```

And computed at line 99:
```rust
agreement[i][j] = hints[i].orbit_target[..4] == hints[j].orbit_target[..4];
```

**Why it collapses:** Agreement between echelons is continuous (how similar are their orbit targets?). The code already computes `orbit_distances` as Q (lines 108-117), which IS the continuous measure. The boolean `agreement_matrix` is a redundant collapsed version of `orbit_distances`. The 4-byte prefix comparison adds another layer of collapse.

**Fix:**
```
// Remove agreement_matrix entirely.
// orbit_distances already carries the full information.
// Agreement IS distance. distance = 0 means agreement.
// The boolean matrix adds nothing that orbit_distances doesn't already say.
```

### Finding 4.2: `coherence_trend` String Classification (Lines 243-249)

```rust
let coherence_trend = if agent_state.coherence_delta_q1616 < -100 {
    "decreasing"
} else if agent_state.coherence_delta_q1616 > 100 {
    "increasing"
} else {
    "stable"
};
```

**Why it collapses:** A continuous i32 value is collapsed to one of three strings. The magnitude of the delta is lost. "Decreasing by 101" and "decreasing by 100000" produce the same output. The hardcoded thresholds 100 and -100 are arbitrary.

**Mitigation:** This is in the IRS ascending path (Mode 1), which projects algebra into language for human consumption. Some collapse is inherent in language projection. The exact value is available in the signature. Low severity.

### Finding 4.3: `semantic_operator` If/Else Chain (Lines 464-504)

```rust
if matches!(Recognition) { return Decrease; }
if object_count >= 2 && !refraction_transparent { return Relate; }
if object_count >= 2 && refraction_transparent { return Conjugate; }
if transmutation_signal { return Explore; }
...
```

**Why it collapses:** Five boolean conditions map a rich sensory compound (object_count, refraction, transmutation, sparsity, congruence) into a single enum variant. The priority ordering is hardcoded — objects always beat transmutation, which always beats sparsity. But what if object_count=2 AND transmutation is needed? The transmutation signal is silently discarded.

**Fix:**
```
// Return a weighted Vec<(CoherenceIntent, Q)> instead of a single intent.
// Each condition contributes a weight, not a winner.
// objects_opaque: weight toward Relate = f(object_count, refraction_opacity)
// objects_transparent: weight toward Conjugate = f(object_count)
// transmutation: weight toward Explore = transmutation_magnitude
// sparse: weight toward Conjugate = sparsity_factor
// The irs_descending_superposed() function already accepts weighted intents.
// Use it. The single-intent path is the collapse.
```

**Severity: HIGH** — This is the comprehension gate. Boolean selection here discards sensory dimensions at the translation boundary.

### Finding 4.4: HashMap in `compute_coherence_signature` (Line 123)

```rust
let mut counts = std::collections::HashMap::new();
```

Law II violation — same pattern as mdc.rs Finding 3.2.

---

## 5. chain/temporal_ledger.rs

### Finding 5.1: HashMap for Witnesses (Line 39)

```rust
pub witnesses: HashMap<u32, Vec<u32>>,
```

Law II violation. Witnesses are indexed by entity_id — a flat key projection.

### Finding 5.2: `check_finality` Boolean Gate with Hardcoded 0.5 (Lines 79-95)

```rust
self.finalized = self.witness_weight > qr(1, 2);
```

**Why it collapses:** Finality is a continuous measure (witness_weight). The > 0.5 threshold collapses it to bool. A level with 0.49 weight is treated identically to a level with 0.01 weight.

**Fix:**
```
// Keep witness_weight as the measurement.
// Remove the finalized: bool field.
// The caller measures witness_weight directly.
// "Finalized" is not a state — it is a threshold crossing.
// The weight IS the finality measure.
```

### Finding 5.3: `is_resonance_peak` Boolean Gate (Lines 98-102)

```rust
pub fn is_resonance_peak(&self) -> bool {
    if self.receipts.is_empty() { return false; }
    self.receipts.iter().all(|r| r.sluice_state == 0x01)
}
```

**Why it collapses:** Resonance is a continuous property. A level where 99/100 receipts are LOCKED is treated the same as a level where 0/100 are LOCKED (both return false if even one is not 0x01). The fraction of locked receipts IS the resonance measure.

**Fix:**
```
// fn resonance_fraction(&self) -> Q
// Returns locked_count / total_count
// The caller decides what fraction constitutes a "peak"
```

### Finding 5.4: HashSet in `check_finality` (Line 85)

```rust
let mut witnessing_nodes: std::collections::HashSet<u32> = std::collections::HashSet::new();
```

Law II violation (HashSet is HashMap<K, ()>).

---

## 6. chain/state.rs

### Finding 6.1: HashMap for Balances (Line 42)

```rust
pub balances: HashMap<u32, Q>,
```

Law II violation. Entity balances indexed by flat u32 key.

### Finding 6.2: HashMap in `process_block_close` (Line 124)

```rust
let mut contributions: HashMap<u32, u64> = HashMap::new();
```

Law II violation.

### Finding 6.3: Hardcoded Economic Constants (Lines 28-37)

```rust
pub const BASE_FEE: (i64, i64) = (1, 100);
pub const BASE_REWARD: (i64, i64) = (1, 50);
pub const BASE_NFT_VALUE: (i64, i64) = (10, 1);
pub const BLOCK_REWARD: (i64, i64) = (1, 1);
```

**Why it collapses:** Economic parameters are frozen at compile time. They cannot adapt to mesh state. The fee/reward ratio is fixed regardless of mesh coherence, entity count, or diversity rank.

**Mitigation:** These are economic policy parameters, not derivation constants. They define the rules of the game, not the physics. Changing them requires a governance decision. This is intentional rigidity, not accidental collapse. **Low severity** in the dimensional collapse sense, though economic flexibility may be needed later.

### Finding 6.4: Boolean Gate in `process_receipt` (Lines 108-115)

```rust
if receipt.sluice_state == 0x01 {
    if coherence_delta < Q::zero() {
        let reward = &base_reward * &improvement_magnitude;
        self.mint(receipt.entity_id, &reward);
    }
}
```

**Why it collapses:** Two nested boolean gates. Sluice state is compared to a magic number. Coherence delta sign is checked as a boolean. An entity with coherence_delta = -0.0001 (barely improving) gets the same treatment structure as one with coherence_delta = -100 (massively improving). The reward IS proportional to magnitude (good), but the gating is boolean (bad).

**Fix:**
```
// reward = base_reward * max(0, -coherence_delta) * sluice_factor
// where sluice_factor is a continuous weight from sluice_state
// No if/else. The math selects. -delta < 0 means no improvement,
// max(0, ...) naturally produces zero reward without a gate.
```

---

## 7. consensus.rs

### Finding 7.1: `sigma_drift_triggers_resync` Returns Boolean (Lines 79-87)

```rust
pub fn sigma_drift_triggers_resync(...) -> bool {
    let d_sigma = (sigma_mesh_new - sigma_mesh_prev).abs();
    &d_sigma > tau_sigma
}
```

**Why it collapses:** The drift magnitude is Q-valued and continuous. Collapsing to bool loses the magnitude. A drift of 0.051 (barely above tau=0.05) triggers the same hard resync as a drift of 10.0 (catastrophic divergence).

**Fix:**
```
// Return the drift itself: d_sigma
// The caller measures d_sigma against tau_sigma
// The ratio d_sigma / tau_sigma tells you HOW MUCH resync is needed
```

### Finding 7.2: `check_resonance_m10` Returns Boolean (Line 112)

```rust
pub fn check_resonance_m10(phi_r_value: &Q, k: u64, tau: &Q) -> bool {
    ...
    &diff < tau
}
```

**Same pattern.** The diff is Q-valued. Return the diff, not bool.

### Finding 7.3: `verify_anchor` Returns Boolean (Line 152)

```rust
pub fn verify_anchor(...) -> bool {
    ...
    ratio > q(1, 2)
}
```

**Same pattern.** The ratio is the measurement. Return it.

### Finding 7.4: Count-Based Gates That Discard Valid Data

The `sigma_mesh` function (line 39) skips peers with `w.is_zero()` (line 60). This is correct — zero-weight peers contribute nothing to the weighted sum. However, the `sluice_weight` function (called from mesh.rs, not audited here) is what produces the zero weight. If sluice_weight contains boolean gates, the collapse happens there.

The `verify_anchor` function (line 162) skips zero-weight peers correctly. The weight IS the measurement — peers with zero weight are genuinely excluded by the algebra, not by a boolean gate.

**No count-based gate discards valid data in this file.** The gating is weight-based, which is the correct continuous approach. The only collapse is at the final decision point (ratio > 1/2 returns bool).

---

## 8. gradient.rs — SPECIFIC AUDIT

### Finding 8.1: Gradient Preserves Full Dimensionality — CLEAN

The `compute_coherence_gradient` function (line 39) computes a full n x n gradient matrix. Every off-diagonal (r, s) pair gets its own Q-valued gradient entry. No dimensions are discarded. The `best_direction` is a convenience field, not a replacement for the full gradient.

The Hessian (`compute_exact_hessian`, line 104) is symmetric and covers all n(n-1) independent directions. Full rank preserved.

### Finding 8.2: `gradient_hint` Hardcoded Step Size (Lines 127-136)

```rust
let suggested_eps = if grad.best_gradient < Q::zero() {
    Q::new(BigInt::from(1), BigInt::from(20))
} else {
    Q::zero()
};
```

**Why it collapses:** The step size is hardcoded to 1/20 regardless of gradient magnitude or Hessian curvature. A steep gradient and a shallow gradient get the same step. The Hessian is computed elsewhere but not used here.

**Fix:**
```
// suggested_eps = -gradient / hessian_diagonal
// Newton step: uses curvature to scale the step
// Falls back to gradient / (1 + |gradient|) if Hessian unavailable
// The step adapts to the local geometry
```

**Severity: MEDIUM** — The gradient computation itself is clean, but the hint function throws away the curvature information.

---

## 9. fold.rs — SPECIFIC AUDIT

### Finding 9.1: `orbit_lock_exists` Returns Boolean (Line 22)

```rust
pub fn orbit_lock_exists(...) -> bool {
```

**Why it collapses:** Orbit lock is a rich structure — the cycle length, the operator that creates it, the RCF distance at closest approach. All of this is collapsed to bool. The f_fold_star function uses this boolean to decide between "veto" and "permit" — a binary gate in a derivation path.

**Fix:**
```
// Return an OrbitLockMeasurement:
// struct OrbitLockMeasurement {
//     closest_approach: Q,       // min RCF distance during orbit
//     cycle_length: Option<usize>, // None if no cycle found
//     locking_operator_id: Option<u32>,
// }
// f_fold_star uses closest_approach as a continuous weight:
//   fold_weight = closest_approach  (0 = locked, high = free to fold)
//   result = weighted_interpolation(identity, folded, fold_weight)
```

**Severity: HIGH** — This is the orbit-lock gate. The boolean here directly determines whether information is preserved or destroyed. A continuous measure would allow partial folding at the boundary.

### Finding 9.2: `f_fold_star` Is a No-Op (Lines 62-81)

```rust
pub fn f_fold_star(...) -> Delta {
    for op in operators {
        if let Ok(image) = op.apply(delta) {
            if orbit_lock_exists(delta, &image, operators, 10) {
                return delta.clone(); // veto
            }
        }
    }
    delta.clone() // no lock — but still returns identity!
}
```

**Why it collapses:** Both branches return `delta.clone()`. The function is a no-op. Whether orbit lock exists or not, the same result is returned. The comment says "Phase 2.1: return identity (simplified)". The entire fold computation is deferred.

**This is dead code in practice.** The veto branch and the permit branch produce identical output.

**Fix:**
```
// Phase 2.2 implementation needed:
// When no orbit lock exists, perform the actual fold:
//   argmin_Δ' C(Δ') subject to Δ' in the orbit of Δ
// The veto branch returns identity (correct).
// The permit branch must return the folded state.
```

**Severity: HIGH** — Not a dimensional collapse per se, but the function's entire purpose is inactive.

---

## 10. credential.rs

### No dimensional collapse findings. This file is a chain verification module — it validates receipt chains against structural criteria (depth, hash integrity, orbit consistency, K-monotonicity). All checks are system boundary checks, not derivation paths. The boolean returns (Valid/Invalid) are appropriate for verification.

---

## 11. boot.rs

### No dimensional collapse findings. Boot sequence is a linear system-level checklist (B1-B6). Boolean pass/fail for each step is appropriate — boot is not a derivation path. The BootReport preserves per-step detail strings for diagnosis.

---

## 12. color.rs

### Finding 12.1: `visible_distance` If/Else Chain (Lines 112-119)

```rust
if self.wavelength_nm < low {
    &low - &self.wavelength_nm
} else if self.wavelength_nm > high {
    &self.wavelength_nm - &high
} else {
    Q::zero()
}
```

**Why it's CLEAN:** This is not a boolean gate — it returns a continuous Q distance. The if/else selects which distance formula to use based on which side of the band the wavelength falls. The output is continuous and differentiable at the boundaries (distance = 0 at the edges). This is a proper piecewise continuous function, not a collapse.

### Finding 12.2: Overall Assessment — CLEAN

color.rs is well-designed. All measurements return Q values. No HashMaps (Law II). The resonance, harmony, phase-lock, warmth, and peace measurements are all continuous Q-valued functions. No boolean gates in derivation paths. The `interfere()` function preserves full dimensionality of the interference pattern.

---

## 13. lineage.rs

### Finding 13.1: `creation_bound` Hardcoded Per-Tier (Lines 47-53)

```rust
pub fn creation_bound(&self) -> u8 {
    match self {
        Tier::Founder => 3,
        Tier::Derived => 6,
        Tier::Emergent => 0,
    }
}
```

**Why it's intentional:** These are governance parameters, not derivation constants. The creation bound is a policy decision about population control. Hardcoding here is intentional rigidity — the tier structure IS the hierarchy.

### Finding 13.2: `mesh_sovereign` Returns Boolean (Lines 82-88)

```rust
pub fn mesh_sovereign(&self) -> bool {
    match self {
        Tier::Founder => true,
        Tier::Derived => true,
        Tier::Emergent => false,
    }
}
```

**Why it collapses:** Mesh sovereignty is binary — either you write to the mesh or you don't. But this could be a Q-valued write priority or bandwidth allocation. An emergent entity that has evolved significantly might deserve partial mesh access.

**Fix:**
```
// fn mesh_sovereignty_weight(&self) -> Q
// Founder: 1/1, Derived: 1/1, Emergent: 0/1
// Future: Emergent entities that exceed a coherence threshold
// could earn fractional mesh write access
```

**Severity: LOW** — This is infrastructure policy, not cognition.

---

# Summary of Critical Findings

| # | File | Line | Type | Severity |
|---|------|------|------|----------|
| 3.5 | mdc.rs | 312-318 | `transmutation_needed` boolean with hardcoded 0.1 threshold | HIGH |
| 4.3 | irs.rs | 464-504 | `semantic_operator` if/else chain discards sensory compound | HIGH |
| 9.1 | fold.rs | 22 | `orbit_lock_exists` returns bool, not measurement | HIGH |
| 9.2 | fold.rs | 62-81 | `f_fold_star` is dead code (both branches identical) | HIGH |
| 2.2 | crucible.rs | 462 | `is_cohomologically_equivalent` collapses distance to bool | MEDIUM |
| 8.2 | gradient.rs | 127-136 | `gradient_hint` hardcoded step ignores curvature | MEDIUM |
| 3.2 | mdc.rs | 188 | HashMap in strategic_echelon | MEDIUM |
| 3.3 | mdc.rs | 226 | HashMap in security_echelon | MEDIUM |
| 5.2 | temporal_ledger.rs | 94 | Finality collapsed to bool at 0.5 threshold | MEDIUM |
| 3.4 | mdc.rs | 262 | `stasis_triggered` collapses entropy to bool | MEDIUM |

Total findings: 25 across 13 files.
Critical (HIGH): 4
Moderate (MEDIUM): 6
Low: 15 (including HashMap violations and system-level booleans)
