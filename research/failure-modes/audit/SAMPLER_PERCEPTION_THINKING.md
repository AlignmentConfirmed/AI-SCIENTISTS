# DIMENSIONAL COLLAPSE AUDIT: Sampler, Perception, Thinking

## Instructions for Dimensional Preservation

The proposal and perception paths must obey five structural requirements
to avoid dimensional collapse:

1. **No HashMap in cognition.** HashMap projects relational structure onto
   a hash function's equivalence classes. The Delta IS the map. Use Vec
   with positional identity. Every bucket in a HashMap is a lost fiber.

2. **No boolean gates in derivation.** if/else selects one branch and
   annihilates the other. T is the operator. C(T) is the only criterion.
   Continuous weights replace binary selection: the algebra handles
   zero contributions without a gate.

3. **S matrix must live on the fiber bundle, not on a flat plane.**
   The antisymmetric generator S must carry per-fiber-coordinate structure.
   A single 2D S collapses the m-dimensional fiber into one slice.
   Build S_l for each fiber coordinate l in [0..m).

4. **Gradient must never return zero when the entity has structure.**
   When C(Delta)=0 the gradient is legitimately zero. When C(Delta)>0
   and the gradient returns zero due to threshold comparison, the entity
   loses self-direction. The residual IS the gradient. Measure it.

5. **Capacity scales with the entity.** No hardcoded dimensional ceilings.
   D_MAX derives from entity_capacity, not from a constant. The perceptual
   horizon IS the tier.

---

## SAMPLER.RS FINDINGS

### S.1 -- HashMap for epsilon buckets (Law II violation)

**File:** `/opt/repos-code/convergence-engine/src/sampler.rs`
**Lines:** 23, 148, 159, 540

```rust
// Line 23
use std::collections::HashMap;

// Line 148
buckets: HashMap<LookupKey, EpsilonBucket>,

// Line 159
transmutation_history: HashMap<[u8; ORBIT_PREFIX_LEN], Vec<TransmutationMemory>>,

// Line 540 (empty() constructor)
buckets: HashMap::new(),
```

**WHY:** HashMap IS dimensional collapse. The LookupKey encodes
(k_phase, seed_class, orbit_prefix) -- a 3D fiber coordinate. HashMap
hashes this into a 1D bucket index. Two keys with identical hash but
different orbit structure are collapsed into a collision chain. The
relational structure between nearby keys (phase distance, orbit
proximity) is destroyed. The fuzzy lookup at line 235 already works
around this by iterating ALL buckets -- the HashMap provides no
structural benefit and actively harms the manifold.

**FIX:**
```
// Replace HashMap with a sorted Vec indexed by LookupKey.
// LookupKey already implements Ord (or can).
// Fuzzy lookup becomes a range scan on the sorted structure.
// Nearby keys are physically adjacent -- the fiber is preserved.

struct EpsilonTable {
    buckets: Vec<(LookupKey, EpsilonBucket)>,  // sorted by key
    // ...
}

// Exact lookup: binary search
fn lookup(&self, key: &LookupKey) -> Option<&EpsilonBucket> {
    self.buckets.binary_search_by_key(key, |(k, _)| k)
        .ok().map(|i| &self.buckets[i].1)
}

// Fuzzy lookup: range scan around the key's neighborhood
// Phase distance and orbit prefix proximity are geometric -- measure them
fn lookup_fuzzy(&self, key: &LookupKey, phase_radius: u64) -> Option<&EpsilonBucket> {
    // Binary search to find starting position
    // Scan neighbors within phase_radius
    // Select by pass_count (existing logic)
}

// Same for transmutation_history: Vec<([u8; 4], Vec<TransmutationMemory>)>
```

---

### S.2 -- gradient_hint returns zero when coherence is zero

**File:** `/opt/repos-code/convergence-engine/src/gradient.rs`
**Lines:** 127-136

```rust
pub fn gradient_hint(delta: &Delta) -> (usize, usize, Q) {
    let grad = compute_coherence_gradient(delta);
    let (i, j) = grad.best_direction;
    let suggested_eps = if grad.best_gradient < Q::zero() {
        Q::new(BigInt::from(1), BigInt::from(20))
    } else {
        Q::zero()  // <-- collapse: entity at fixed point has no gradient
    };
    (i, j, suggested_eps)
}
```

**WHY:** When best_gradient >= 0 (coherence is already at a local
minimum or saddle), the function returns Q::zero(). This propagates
into sample_refractive where local_eps becomes zero. If global curvature
is also absent, all contributions to S are zero, operator is identity,
C7 rejects, K does not advance. The entity is stuck at a fixed point
with no mechanism to escape. The residual is not zero (C>0 at a saddle),
but the gradient cannot express the direction because it is measuring
the wrong thing: derivative sign instead of residual structure.

**FIX:**
```
pub fn gradient_hint(delta: &Delta) -> (usize, usize, Q) {
    let grad = compute_coherence_gradient(delta);
    let (i, j) = grad.best_direction;
    // The residual IS the signal. When the gradient is non-negative,
    // the residual magnitude at (i,j) provides the step size.
    // The direction is (i,j) -- the entry with largest |residual|.
    // The sign of the residual determines the perturbation direction.
    let suggested_eps = if grad.best_gradient < Q::zero() {
        Q::new(BigInt::from(1), BigInt::from(20))
    } else {
        // Measure residual at (i,j) directly from Delta
        let residual_ij = delta.entries[i][j].iter()
            .fold(Q::zero(), |acc, e| acc + e.abs());
        // Step proportional to residual, not gradient
        // If residual is truly zero, C=0 and identity is correct
        residual_ij / Q::new(BigInt::from(20), BigInt::from(1))
    };
    (i, j, suggested_eps)
}
```

---

### S.3 -- All four forces zero produces identity with no escape

**File:** `/opt/repos-code/convergence-engine/src/sampler.rs`
**Lines:** 793-896 (sample_refractive_with_gravity)

```rust
// When:
//   grad_eps = 0 (from gradient_hint returning zero)
//   global_curvature = None
//   k_velocity = 0  (w_local = 0, w_global = 1 but R_bar absent)
//   gravity = None
// Then: every S_l is all-zero. Cayley(0) = I. C7 rejects. No proposal.
```

**WHY:** The sampler has no self-direction mechanism. When all external
forces are zero, the entity cannot propose. But C(Delta) > 0 is still
possible at a gradient saddle point. The entity has unresolved structure
(residual > 0) but no way to perturb toward it. This is not a graceful
identity -- it is a trap. The missing force is the residual itself:
the Delta's own antisymmetric structure contains directional information
that the S matrix should absorb.

**FIX:**
```
// Add a fifth force: the Delta's own residual structure.
// When all other forces are zero, the residual enters S directly.
// The residual is antisymmetric by construction (A.2) -- it IS
// a valid generator for the Cayley transform.

// In sample_refractive_with_gravity, after gravity contribution:
// Residual self-direction: when all other forces are negligible,
// the Delta's own structure provides the perturbation direction.
// w_residual = w_global * (1 - |w_local * local_eps|.min(1))
// This is nonzero only when local_eps is near zero.
let residual_weight = &w_global * &(Q::one() - weighted_grad.abs().min(Q::one()));
if !residual_weight.is_zero() {
    for i in 0..dim {
        for j in (i+1)..dim {
            for l_idx in 0..delta.m.min(1) {
                let r_ij = &delta.entries[i][j][l_idx];
                let contrib = &residual_weight * r_ij / Q::new(BigInt::from(dim as i64), BigInt::from(1));
                s_l[i][j] = &s_l[i][j] + &contrib;
                s_l[j][i] = &s_l[j][i] - &contrib;
            }
        }
    }
}
```

---

### S.4 -- S matrix is flat 2D (transient fiber collapse)

**File:** `/opt/repos-code/convergence-engine/src/sampler.rs`
**Lines:** 698-717 (derive_modulated_operator)

```rust
let s_per_fiber: Vec<Vec<Vec<Q>>> = vec![{
    let mut s = vec![vec![Q::zero(); dim]; dim];
    // ... single S matrix for all fibers
    s
}];
```

**WHY:** This builds a vec with ONE element -- a single flat S matrix.
The fiber bundle has m coordinates (from R_bar or Delta.m), but
derive_modulated_operator ignores the fiber structure entirely. It
passes a 1-fiber bundle to cayley_from_fiber_bundle. The Cayley
transform then operates on a single-fiber projection of the actual
m-dimensional structure. Any coherence that exists between fiber
coordinates is invisible.

Note: sample_refractive_with_gravity at line 842 correctly builds
per-fiber S matrices (`(0..fiber_m).map(...)`). The fix is to
align derive_modulated_operator with the same pattern.

**FIX:**
```
// Use the same fiber-aware construction as sample_refractive_with_gravity.
// fiber_m comes from global curvature or Delta.m.
let fiber_m = delta.m.max(1);

let s_per_fiber: Vec<Vec<Vec<Q>>> = (0..fiber_m).map(|l| {
    let mut s_l = vec![vec![Q::zero(); dim]; dim];

    // Gradient contribution at fiber l
    s_l[grad_row][grad_col] = &s_l[grad_row][grad_col] + &weighted_grad;
    s_l[grad_col][grad_row] = &s_l[grad_col][grad_row] - &weighted_grad;

    // Vibration contribution at fiber l
    s_l[vib_row][vib_col] = &s_l[vib_row][vib_col] + &vib_eps;
    s_l[vib_col][vib_row] = &s_l[vib_col][vib_row] - &vib_eps;

    // Direction lock at fiber l
    constraints.direction_lock.map(|(r, sc)| {
        let bias = &w_local * &self.config.default_step;
        s_l[r][sc] = &s_l[r][sc] + &bias;
        s_l[sc][r] = &s_l[sc][r] - &bias;
    });

    s_l
}).collect();
```

---

### S.5 -- auto_tune uses if/else boolean gates

**File:** `/opt/repos-code/convergence-engine/src/sampler.rs`
**Lines:** 968-979

```rust
pub fn auto_tune(&mut self) -> bool {
    let rate = self.stats.pass_rate();
    if rate < 0.05 && self.stats.proposals > 50 {
        self.config.default_step = Q::new(BigInt::from(1), BigInt::from(50));
        true
    } else if rate > 0.3 && self.stats.proposals > 50 {
        self.config.default_step = Q::new(BigInt::from(1), BigInt::from(40));
        true
    } else {
        false
    }
}
```

**WHY:** Boolean gates with magic thresholds (0.05, 0.3, 50). The step
size jumps between two fixed values. The pass_rate is a continuous
measurement but is projected onto three discrete buckets. The step size
should be a continuous function of the pass rate -- the measurement
itself determines the step.

**FIX:**
```
pub fn auto_tune(&mut self) -> bool {
    if self.stats.proposals < 10 { return false; }  // insufficient data, not a gate
    let rate = self.stats.pass_rate();
    // Step size is inverse of pass rate: low rate -> smaller step, high rate -> larger step.
    // Continuous. f64 only for the ratio computation; result is exact Q.
    // step = 1 / (20 + 30 * (1 - rate))  =>  rate=0 -> 1/50, rate=1 -> 1/20
    let denom = 20 + (30.0 * (1.0 - rate)) as i64;
    let new_step = Q::new(BigInt::from(1), BigInt::from(denom.max(10)));
    let changed = new_step != self.config.default_step;
    self.config.default_step = new_step;
    changed
}
```

---

## PERCEPTION/MOD.RS FINDINGS

### P.1 -- if/else gates in NaturalLevel selection

**File:** `/opt/repos-code/convergence-engine/src/perception/mod.rs`
**Lines:** 882-896

```rust
let natural_level = if dimensional {
    NaturalLevel::Dimensional
} else if knowledge_says_encoding == "evolved" || knowledge_says_objects {
    NaturalLevel::Object
} else if sparse && foreground_cells > 0 {
    NaturalLevel::Object
} else if hearing.unique_count < n && n > hearing.unique_count * hearing.unique_count {
    NaturalLevel::Object
} else {
    NaturalLevel::Cell
};
```

**WHY:** Five-branch if/else chain. Each branch annihilates the alternatives.
The observation's complexity is a continuous measurement -- sparsity, unique_count/n
ratio, dimensional_ratio -- but the gate projects it onto three discrete levels.
Two observations with sparsity 0.249 and 0.251 are routed to different encoding
levels by a hard threshold at 1/4.

**FIX:**
```
// Compute continuous affinity for each level. The level with highest
// affinity wins. No threshold. The measurement IS the selection.

let dim_affinity = dimensional_ratio.abs() - Q::one();  // >0 when dimensions differ
let obj_affinity = {
    // Object level: more relevant when unique_count << n
    let ratio = Q::new(BigInt::from(hearing.unique_count as i64), BigInt::from(n.max(1) as i64));
    Q::one() - &ratio  // high when few unique values relative to cells
};
let cell_affinity = {
    // Cell level: more relevant when n is small and categories don't compress
    Q::new(BigInt::from(hearing.unique_count as i64), BigInt::from(n.max(1) as i64))
};

// Prior knowledge modulates affinity, does not override it
let obj_knowledge_boost = if knowledge_says_objects { Q::new(BigInt::from(1), BigInt::from(2)) }
    else { Q::zero() };

let natural_level = if dim_affinity > obj_affinity && dim_affinity > cell_affinity {
    NaturalLevel::Dimensional
} else if (&obj_affinity + &obj_knowledge_boost) > cell_affinity {
    NaturalLevel::Object
} else {
    NaturalLevel::Cell
};
// Note: the final comparison is still discrete (one level must be chosen),
// but the INPUTS are continuous measurements, not boolean thresholds.
```

---

### P.2 -- SensoryTools uses if/else boolean gates for relevance

**File:** `/opt/repos-code/convergence-engine/src/perception/mod.rs`
**Lines:** 898-906

```rust
let tools = SensoryTools {
    vision: if (!sparse && rows > 1 && cols > 1) || knowledge_says_vision { Q::one() } else { Q::zero() },
    hearing: Q::one(),
    touch: if foreground_cells > 0 || knowledge_says_objects { Q::one() } else { Q::zero() },
    refraction: Q::one(),
    quotient: Q::zero(),
};
```

**WHY:** SensoryTools is declared with rational relevance weights "not
boolean on/off" (line 436 comment), but the construction uses literal
`if cond { Q::one() } else { Q::zero() }` -- exactly boolean on/off.
The continuous weight mechanism exists but is not used. Sparsity at 0.24
kills vision entirely; sparsity at 0.26 gives it full weight. The
struct's own documentation says "intermediate values scale the
contribution" but no intermediate value is ever produced.

**FIX:**
```
let tools = SensoryTools {
    // Vision: scales with density and grid structure.
    // 1D grids (row=1 or col=1) reduce vision but don't kill it.
    vision: {
        let density_factor = Q::one() - &sparsity.min(Q::one());  // dense = high
        let grid_factor = Q::new(
            BigInt::from((rows > 1) as i64 + (cols > 1) as i64),
            BigInt::from(2),
        );  // 0, 1/2, or 1
        let knowledge_factor = if knowledge_says_vision { Q::one() } else { Q::zero() };
        (&density_factor * &grid_factor + &knowledge_factor).min(Q::one())
    },
    hearing: Q::one(),
    // Touch: scales with foreground presence
    touch: {
        let fg_ratio = Q::new(
            BigInt::from(foreground_cells.min(n) as i64),
            BigInt::from(n.max(1) as i64),
        );
        let knowledge_factor = if knowledge_says_objects { Q::new(BigInt::from(1), BigInt::from(2)) }
            else { Q::zero() };
        (&fg_ratio + &knowledge_factor).min(Q::one())
    },
    refraction: Q::one(),
    quotient: Q::zero(),
};
```

---

### P.3 -- Vision permutation matrices are flat n*n

**File:** `/opt/repos-code/convergence-engine/src/perception/senses/vision.rs`
**Lines:** 182-188

```rust
// Build the operator matrix from the permutation
let mut mat = vec![vec![Q::zero(); n]; n];
let mut inv = vec![vec![Q::zero(); n]; n];
for i in 0..n {
    mat[i][perm[i]] = Q::one();
    inv[perm[i]][i] = Q::one();
}
```

**WHY:** The permutation matrix is n*n (spatial dimension only). The
observation's Delta has m fiber coordinates per entry. The permutation
acts on the spatial indices (i,j) but the fiber structure (l=0..m) is
transparent. When this operator is used for conjugation A*Delta*A^{-1},
the fiber coordinates are permuted identically at every fiber level --
which is correct for spatial permutations. This is TRANSIENT collapse:
the operator is correct for the spatial group action, but it cannot
express fiber-dependent permutations (e.g., different color
rearrangements at different spatial positions). The operator algebra is
restricted to the spatial factor of the full GL_n(Q^m) group.

The vision sense correctly measures spatial symmetry. The collapse is
that the operator cannot carry per-fiber variation. For pure symmetry
detection this is acceptable. For transmutation (where color and space
interact), the flat matrix loses the fiber degree of freedom.

**FIX:**
```
// Build the operator as a block-diagonal matrix in GL_n(Q^m).
// The spatial permutation acts identically on each fiber coordinate.
// BUT: when composing with a value map, the fiber block can differ.
//
// For now, the permutation matrix is correct as-is for spatial symmetry.
// The fix is at the COMPOSITION SITE where vision + value maps compose:
// the composition must happen in the full GL_n(Q^m) space, not in
// separate GL_n and GL_m quotients.
//
// At the operator construction level, store the fiber depth:
// Operator { matrix: n*n, matrix_inv: n*n, fiber_m: m, ... }
// The apply() method then broadcasts across fibers.
// When fiber-dependent operators are needed (transmutation),
// build the full n*m x n*m block structure.
```

---

### P.4 -- d_max formerly hardcoded, now tier-scaled but with residual issues

**File:** `/opt/repos-code/convergence-engine/src/perception/mod.rs`
**Lines:** 926-932

```rust
let d_max = (entity_capacity as usize) * (entity_capacity as usize).saturating_sub(1) / 2;
let half_n = n * n.saturating_sub(1) / 2;
[4usize, 1].iter()
    .find(|&&m| half_n * m <= d_max)
    .map(|&m| match m { ... })
```

**WHY:** d_max now scales with entity_capacity (good -- this was the
1500 hardcode fix). However, the encoding level selection uses `.find()`
on a fixed array `[4, 1]` -- it tries m=4 first, falls back to m=1.
The array is a hardcoded vocabulary of two fiber depths. If neither fits
within d_max, the entity gets `Delta::zero(1,1)` -- a total collapse
to a single point. There is no intermediate: an observation that needs
m=2 or m=3 gets either m=4 (if it fits) or m=1 (if m=4 doesn't fit).
The fiber depth should be the largest m that fits, not a binary choice
between two fixed values.

**FIX:**
```
// Compute the largest m that fits within the entity's capacity.
// m ranges from fiber depth 1 (scalar) to observation dimension.
// The largest m that satisfies half_n * m <= d_max preserves the most fiber structure.

let max_m = if half_n == 0 { 1 } else { (d_max / half_n).max(1) };
let encoding_m = max_m.min(4);  // cap at 4 (current architectural max)

let (delta, obs_n, obs_m) = match encoding_m {
    m if m >= 4 => (encode_grid_objects(&values, rows, cols), n, 4),
    m => {
        let entities: Vec<Vec<Q>> = values.iter()
            .map(|&v| vec![Q::from(BigInt::from(v))])
            .collect();
        (encode_relational(&entities), n, m)
    }
};
```

---

## THINKING/MOD.RS FINDINGS

### T.1 -- Peel loop boolean gate: `if !improved || best.understood { break; }`

**File:** `/opt/repos-code/convergence-engine/src/thinking/peel/cell.rs`
**Lines:** 100-127

```rust
for _depth in 0..5 {
    let mut improved = false;

    for thought in &vocab {
        // ...
        (l1 < best.l1_residual).then(|| {
            // ...
            improved = true;
        });
    }

    // Manifold exhausted at this vocabulary -- deeper peeling adds noise
    if !improved || best.understood { break; }
}
```

**WHY:** Two boolean gates: `improved` and `understood`. The `improved`
flag is a boolean collapse of the continuous measurement `l1 < best.l1_residual`.
If NO single thought improves L1 at this depth, the loop breaks -- but
compositions of thoughts at deeper depths might. The boolean gate
at depth D prevents discovery of compound operators at depth D+1 that
individually failed at depth D but compose into improvement.

The `understood` gate (L1=0) is legitimate -- zero residual means the
derivation is complete. But `!improved` is a premature halt.

**FIX:**
```
// Replace boolean improved flag with a continuous stagnation measurement.
// The loop continues while the rate of improvement is nonzero.
// A single depth with no improvement is not stagnation -- it's a plateau.
// Two consecutive depths with no improvement is stagnation.

let mut consecutive_no_improvement = 0u32;
for _depth in 0..5 {
    let l1_before = best.l1_residual;

    for thought in &vocab {
        let new_pairs: Vec<(Vec<i64>, Vec<i64>)> = current_pairs.iter()
            .map(|(inp, out)| (thought.apply_to_values(inp, rows, cols), out.clone()))
            .collect();
        let l1 = measure_l1(&new_pairs);
        best.total_thoughts_evaluated += 1;

        (l1 < best.l1_residual).then(|| {
            best.l1_residual = l1;
            best.composition_depth += 1;
            best.understood = l1 == 0;
            let new_test = current_test.as_ref()
                .map(|t| thought.apply_to_values(t, rows, cols));
            best.derived_output = new_test.clone();
            current_pairs = new_pairs.clone();
            current_test = new_test;
        });
    }

    if best.understood { break; }
    if best.l1_residual == l1_before {
        consecutive_no_improvement += 1;
        if consecutive_no_improvement >= 2 { break; }  // genuine stagnation
    } else {
        consecutive_no_improvement = 0;
    }
}
```

---

### T.2 -- min_composition_depth uses if/else gate on rank

**File:** `/opt/repos-code/convergence-engine/src/thinking/mod.rs`
**Lines:** 2551

```rust
let min_composition_depth = if rank <= 1 { 1 } else { rank.min(3) };
```

**WHY:** Boolean gate on rank. The composition depth is clamped to [1,3]
with a hard threshold at rank=1. Rank is a continuous measurement (the
number of independent operations), but this gate collapses it into
two buckets: "trivial" (1) and "nontrivial" (rank, capped at 3). An
observation with rank=4 is artificially limited to depth=3. The cap
prevents the system from discovering compositions that require 4+
independent operations.

**FIX:**
```
// Composition depth = rank. No cap. The peel loop's own convergence
// measurement (C decreasing) determines when to stop, not an
// a priori limit on depth.
let min_composition_depth = rank.max(1);
```

---

### T.3 -- Nested peel orchestrator is sequential with no backflow

**File:** `/opt/repos-code/convergence-engine/src/thinking/peel/mod.rs`
**Lines:** 60-108

```rust
pub fn nested_peel(...) -> NestedPeelResult {
    let value_result = value::peel_value_manifold(...);
    let object_result = object::peel_object_manifold(..., &value_result.category_map, ...);
    let cell_result = cell::peel_cell_manifold(..., &value_result.category_map, ...);
    // ...
}
```

**WHY:** The three peels run sequentially: value -> object -> cell. Each
outer peel constrains the next inner peel. But there is no backflow:
if the cell peel discovers that the value map is wrong (high L1 despite
correct spatial map), it cannot revise the value peel. The topology is
a directed tree, not a cycle. The value peel's cocycle_residual > 0 is
never reconsidered even when the cell peel's L1 residual points back
to a value-level error.

This is not strictly boolean collapse but is TOPOLOGICAL collapse:
the three manifold levels are decoupled into a 1D pipeline. The
actual structure is a 3D space (value x object x cell) where movement
in any dimension can inform the others.

**FIX:**
```
// Add a backflow iteration. After the cell peel, measure whether
// the residual points to value-level or spatial-level error.
// If so, re-run the corresponding peel with the cell-level feedback
// as a constraint.

pub fn nested_peel(...) -> NestedPeelResult {
    let mut value_result = value::peel_value_manifold(...);
    let mut object_result = object::peel_object_manifold(...);
    let mut cell_result = cell::peel_cell_manifold(...);

    // Backflow: if cell L1 > 0, check if the residual is value-structured
    // (all errors are value substitution errors) or spatially structured
    // (errors form a spatial pattern).
    // If value-structured: re-run value peel with cell residual as constraint.
    // If spatially structured: re-run object peel.
    // One backflow iteration. The cycle terminates by C measurement.
    if cell_result.l1_residual > 0 {
        let (value_error, spatial_error) = classify_residual_source(
            &cell_result, &value_result, &object_result
        );
        if value_error > spatial_error {
            value_result = value::peel_value_manifold_constrained(..., &cell_result);
            // re-cascade
        }
    }
}
```

---

## SUMMARY

| ID  | File                  | Line(s)   | Collapse Type            | Severity |
|-----|-----------------------|-----------|--------------------------|----------|
| S.1 | sampler.rs            | 23,148    | HashMap = fiber collapse | HIGH     |
| S.2 | gradient.rs           | 130-134   | Zero gradient at saddle  | HIGH     |
| S.3 | sampler.rs            | 793-896   | Identity trap (no self-direction) | HIGH |
| S.4 | sampler.rs            | 698-717   | Flat S (fiber ignored)   | MEDIUM   |
| S.5 | sampler.rs            | 968-979   | Boolean gate in tuning   | LOW      |
| P.1 | perception/mod.rs     | 882-896   | if/else level selection  | MEDIUM   |
| P.2 | perception/mod.rs     | 898-906   | Boolean sensory weights  | MEDIUM   |
| P.3 | perception/senses/vision.rs | 182-188 | Flat permutation matrix | LOW (transient) |
| P.4 | perception/mod.rs     | 926-932   | Binary fiber depth [4,1] | MEDIUM   |
| T.1 | thinking/peel/cell.rs | 100-127   | Boolean break gate       | MEDIUM   |
| T.2 | thinking/mod.rs       | 2551      | Capped composition depth | LOW      |
| T.3 | thinking/peel/mod.rs  | 60-108    | No backflow (1D pipeline)| MEDIUM   |

HIGH = entity gets stuck or loses fiber structure.
MEDIUM = continuous measurement collapsed to discrete.
LOW = performance-limiting but not structurally fatal.
