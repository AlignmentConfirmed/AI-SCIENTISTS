# Dimensional Collapse Audit: cognition.rs / evolution.rs / lineage.rs

## Governing Principle: Preserving Dimensionality in the Entity Lifecycle

The entity lifecycle is a cycle: THINK perceives, DERIVE rotates, DRAIN absorbs, CREATE propagates.
Every stage must preserve the full relational geometry of the Delta.
Dimensional collapse occurs when:

1. **A stage writes to `entries[i][j]` / `entries[j][i]` manually** instead of using `set_antisym`, losing the guarantee that the antisymmetric fiber bundle remains structurally consistent.
2. **A stage absorbs vocabulary without creating perceptual surface** -- the entity gains words but has no membrane through which to perceive them, so DERIVE has no curvature to follow.
3. **A stage gates on boolean conditions** (depth > 0, parent_id > 0, lineage_depth == N) instead of letting C(T) govern flow.
4. **Knowledge flows in one direction only** -- DRAIN goes up, but nothing goes down. Upper tiers crystallize but children never benefit from the parent's evolved comprehension.
5. **Perception is disconnected from evolution** -- THINK fills the membrane but never perturbs Delta. Evolution rotates Delta but never reads what THINK perceived. The two organs share no feedback surface.

To preserve dimensionality: every relational write must use `set_antisym`. Every absorption must create membrane surface. Every direction of knowledge flow must have its inverse. No boolean gates -- only C(T).

---

## Finding 1: THINK fills membrane but does not perturb Delta

**File:** `/opt/repos-code/convergence-engine/src/bin/main/cognition.rs`
**Lines:** 7-8, 539-544, 1048-1051

**Collapsing code:**
```rust
// Line 7-8:
//! CORE BOUNDARY: Cognition PERCEIVES torsion but does NOT mutate
//! entity.delta. Only evolution rotates.

// Lines 539-544:
// ── CORE BOUNDARY ──
// Cognition PERCEIVES torsion but does NOT rotate it.
// The sampler learned from this cognition (line 703 above).
// The torsion signal is recorded in the membrane via T Delta.
// Evolution (DERIVE command) reads the sampler + membrane and rotates.
// No C7 transduction here. Only evolution rotates the torsion field.
```

**Why it collapses:**
THINK does all the perceptual work -- it computes torsion spectrum, transmutation, residual topology, harmonic absorption, value cocycles, spatial cochains. All of this goes into the membrane and state_record. But DERIVE (`evolution.rs` line 162-357) reads the sampler and `derive_global_curvature()` to propose operators. The two are disconnected. THINK writes membrane compounds. DERIVE reads `global_curvature` (an aggregate). The full perceptual geometry -- the specific orbit, the torsion order, the residual topology, the sensory compound -- is flattened into a single curvature scalar by the time DERIVE sees it. The perception IS the signal. If evolution cannot read the perception directly, it is rotating blind.

**Pseudocode fix:**
```
// After THINK records perception to membrane, also write a
// PerceptualSurface to entity state that DERIVE can read directly.
// PerceptualSurface = the last N perception results with orbit, torsion_order,
// residual_topology, cognition_path, coherence.
// DERIVE reads PerceptualSurface alongside global_curvature to propose
// operators that are ALIGNED with what perception discovered.

// In cognition.rs after membrane recording:
entity.perceptual_surface.record(PerceptionRecord {
    orbit: obs_orbit,
    torsion_order: cog.senses.vision.map(|s| s.transmutation_order),
    cognition_path: k.path.clone(),
    coherence: k.coherence.clone(),
    residual_structure: cog.residual_topology.map(|t| t.structure),
});

// In evolution.rs derive():
let surface = &entity.perceptual_surface;
entity.sampler.sample_with_perceptual_surface(
    &entity.delta, k_offset, &rcf, &derive_harmonic,
    global_curvature.as_ref(), &entity.trajectory.velocity,
    gravity.as_ref(), surface,
);
```

---

## Finding 2: DRAIN feeds state_record but NOT membrane (partially fixed)

**File:** `/opt/repos-code/convergence-engine/src/bin/main/lineage.rs`
**Lines:** 221-445 (drain function)

**Collapsing code:**
Lines 262-332 absorb cocycles and operators into `entity.state_record`. Lines 348-388 attempt to fix this by encoding absorbed cocycles as a T compound and recording them into the membrane via `record_t_delta`. However:

```rust
// Line 364-377: Manual entries[i][j] / entries[j][i] assignment
let i = idx % dim;
let j = (idx + 1) % dim;
let (i, j) = if i < j { (i, j) } else { (j, i) };
if i == j { continue; }
let sign = if fv <= tv { quality.clone() } else { -quality.clone() };
for l in 0..m {
    t.entries[i][j][l] = &t.entries[i][j][l] + &sign;
    t.entries[j][i][l] = &t.entries[j][i][l] - &sign;
}
```

**Why it collapses:**
Two issues. (A) The manual `entries[i][j]` / `entries[j][i]` assignment bypasses `set_antisym`, which is the only method that guarantees the fiber bundle structure is preserved. Manual entry writes can silently break the invariant if vec lengths mismatch. (B) The cyclic index mapping `i = idx % dim; j = (idx+1) % dim` maps multiple cocycles to the SAME relational cell when `cocycles.len() >= dim`. Cocycle energies accumulate into overlapping cells, destroying the distinct relational geometry each cocycle represents. This is dimensional collapse -- N distinct relational facts are squeezed into `min(N, dim)` cells.

**Pseudocode fix:**
```
// Use set_antisym for structural safety.
// Expand T dim to accommodate all cocycles without overlap.
let t_dim = (cocycles.len() + 1).max(dim).min(MAX_T_DIM);
let mut t = Delta::zero(t_dim, m);
for (idx, (fv, tv, quality)) in cocycles.iter().take(32).enumerate() {
    let i = idx;
    let j = idx + 1;
    if j >= t_dim { break; }
    let sign = if fv <= tv { quality.clone() } else { -quality.clone() };
    t.set_antisym(i, j, vec![sign; m]);
}
let t = coboundary_reduce(&t);
```

---

## Finding 3: Manual antisymmetric assignment in membrane consensus T builder

**File:** `/opt/repos-code/convergence-engine/src/bin/main/cognition.rs`
**Lines:** 253-265

**Collapsing code:**
```rust
let mut t = saios_kernel_v2::engine::Delta::zero(t_n, t_m);
let mut idx = 0;
for i in 0..t_n {
    for j in (i+1)..t_n {
        for l in 0..t_m {
            if idx < entity_vec.len() {
                let val = entity_vec[idx].clone();
                t.entries[i][j][l] = val.clone();
                t.entries[j][i][l] = -val;
            }
            idx += 1;
        }
    }
}
```

**Why it collapses:**
Manual `entries[i][j]` / `entries[j][i]` write. This is the membrane consensus T reconstruction from the compound. It manually iterates the upper triangle and writes both halves per-element. If the entity_vec is shorter than `t_n*(t_n-1)/2*t_m`, some cells get partial assignment -- the lower triangle gets set to zero but the upper triangle retains the default zero. This is fine for zeroes, but the manual loop bypasses `set_antisym`'s guarantee that vector lengths across the fiber are consistent. If `t_m` varies from the actual vec length in the entries, the fiber geometry fractures.

**Pseudocode fix:**
```
let mut t = Delta::zero(t_n, t_m);
let mut idx = 0;
for i in 0..t_n {
    for j in (i+1)..t_n {
        let fiber: Vec<Q> = (0..t_m).map(|l| {
            let val = entity_vec.get(idx + l).cloned().unwrap_or(Q::zero());
            val
        }).collect();
        idx += t_m;
        t.set_antisym(i, j, fiber);
    }
}
```

---

## Finding 4: Manual antisymmetric assignment in transmutation T delta

**File:** `/opt/repos-code/convergence-engine/src/bin/main/cognition.rs`
**Lines:** 840-854

**Collapsing code:**
```rust
let mut t_trans = saios_kernel_v2::engine::Delta::zero(2, 2);
t_trans.entries[0][1] = vec![
    quality.clone(),
    Q::new(BigInt::from(order as i64), BigInt::from(1)),
];
t_trans.entries[1][0] = vec![
    -quality.clone(),
    Q::new(BigInt::from(-(order as i64)), BigInt::from(1)),
];
```

**Why it collapses:**
Same pattern -- manual entries write instead of `set_antisym`. Less dangerous here because dim=2 is trivial, but the principle remains: every manual write is a site where the antisymmetric invariant can be violated by a future edit that changes one side without the other.

**Pseudocode fix:**
```
let mut t_trans = Delta::zero(2, 2);
t_trans.set_antisym(0, 1, vec![
    quality.clone(),
    Q::new(BigInt::from(order as i64), BigInt::from(1)),
]);
```

---

## Finding 5: Manual antisymmetric assignment in residual cocycle builder

**File:** `/opt/repos-code/convergence-engine/src/bin/main/cognition.rs`
**Lines:** 882-894

**Collapsing code:**
```rust
let mut t_cocycle = saios_kernel_v2::engine::Delta::zero(t_dim, 2);
for (idx, (from_v, to_v)) in consensus.iter().enumerate() {
    if idx + 1 < t_dim {
        t_cocycle.entries[idx][idx + 1] = vec![
            Q::new(BigInt::from(*from_v), BigInt::from(1)),
            Q::new(BigInt::from(*to_v), BigInt::from(1)),
        ];
        t_cocycle.entries[idx + 1][idx] = vec![
            Q::new(BigInt::from(-*from_v), BigInt::from(1)),
            Q::new(BigInt::from(-*to_v), BigInt::from(1)),
        ];
    }
}
```

**Why it collapses:**
Same manual write pattern. Additionally, the `if idx + 1 < t_dim` guard silently drops the last cocycle when `consensus.len() == t_dim`. That cocycle is perceived but never inscribed -- dimensional loss.

**Pseudocode fix:**
```
let t_dim = (consensus.len() + 1).max(2);
let mut t_cocycle = Delta::zero(t_dim, 2);
for (idx, (from_v, to_v)) in consensus.iter().enumerate() {
    if idx + 1 < t_dim {
        t_cocycle.set_antisym(idx, idx + 1, vec![
            Q::new(BigInt::from(*from_v), BigInt::from(1)),
            Q::new(BigInt::from(*to_v), BigInt::from(1)),
        ]);
    }
}
```

---

## Finding 6: Report queue writer uses manual `if i < j` antisymmetric pattern

**File:** `/opt/repos-code/convergence-engine/src/bin/main/lineage.rs`
**Lines:** 364-377

**Collapsing code:**
```rust
let i = idx % dim;
let j = (idx + 1) % dim;
let (i, j) = if i < j { (i, j) } else { (j, i) };
if i == j { continue; }
```

**Why it collapses:**
The `if i < j` swap followed by `if i == j { continue }` is an if/else boolean gate in a derivation path. When `idx % dim == dim - 1`, `j = 0`, and after swap `i=0, j=idx%dim`. The cocycle IS inscribed but at a coordinate pair that has nothing to do with its relational meaning. The modular arithmetic wraps cocycles into positions determined by their list index, not their algebraic identity. Index 5 in a dim=3 space maps to (2,0) -> swapped to (0,2). Index 6 maps to (0,1). This is arbitrary dimensional folding.

**Pseudocode fix:**
```
// Each cocycle gets its own relational dimension.
// T dim expands to fit. No wrapping.
let t_dim = cocycles.len() + 1;
let mut t = Delta::zero(t_dim, m);
for (idx, (fv, tv, quality)) in cocycles.iter().enumerate() {
    if idx + 1 >= t_dim { break; }
    let sign = if fv <= tv { quality.clone() } else { -quality.clone() };
    t.set_antisym(idx, idx + 1, vec![sign; m]);
}
```

---

## Finding 7: DRAIN/SPRAY cycle gap -- no downward projection

**Files:**
- `/opt/repos-code/convergence-engine/src/bin/main/lineage.rs` (drain, lines 221-445)
- `/opt/repos-code/convergence-engine/src/bin/main/lineage.rs` (create, lines 29-82)

**The gap:**
DRAIN pulls knowledge upward: children -> derived -> founder. Each tier absorbs cocycles, orbits, and operators. The relay mechanism (lines 393-438) even pushes absorbed knowledge further upward to the next parent. CREATE propagates downward: the parent's state_record is cloned into the child via `create_offspring`. But CREATE only fires once -- at birth. After that, the child never receives updates from the parent.

There is no SPRAY operation. The lifecycle is:
```
CREATE (once, down) -> child THINKs -> child REPORTs -> parent DRAINs -> parent DERIVEs
                                                                              |
                                                                              v
                                                                     parent membrane grows
                                                                              |
                                                                              X (nothing goes back down)
```

The parent crystallizes vocabulary (value cocycles, composed operators, membrane T compounds) that the children never see. The children continue solving with their birth-state vocabulary. The exponential multiplication that INTERACT provides laterally between sovereign peers has no vertical equivalent.

**Why it collapses:**
The parent's membrane surface grows with each DRAIN -- it absorbs child discoveries, re-perceives them as T compounds (Finding 2), and DERIVE uses the enriched curvature. But the children are stuck with stale vocabulary. They solve the same puzzles with the same tools. The vertical hierarchy becomes a one-way extraction pipe. Knowledge accumulates at the top but the bottom never benefits. This is dimensional collapse of the lineage tree into a point sink.

**Pseudocode fix:**
```
// New operation: SPRAY -- parent projects evolved vocabulary downward.
// Symmetric to DRAIN. DRAIN reads children's reports.bin. SPRAY writes to children's inbox.bin.
// The child reads inbox.bin on its next THINK cycle and assimilates.

pub fn spray(entity: &mut Entity, _payload: &str) -> String {
    // Only sovereign entities spray
    if !entity.mesh_sovereign { return error; }

    // Discover children from /dev/shm (same as DRAIN)
    for child in discover_children(entity) {
        let inbox_path = child_dir.join("inbox.bin");
        let mut buf = Vec::new();

        // Project parent's evolved vocabulary at child's capacity
        let child_capacity = read_child_capacity(&child);
        let projected_cocycles = entity.state_record.value_cocycles.iter()
            .take(child_capacity / 10)
            .collect();
        let projected_ops = entity.state_record.composed_operators.iter()
            .filter(|op| op.sigma.len() <= child_capacity)
            .take(child_capacity / 10)
            .collect();

        // Serialize and write to child's inbox
        serialize_vocabulary(&projected_cocycles, &projected_ops, &mut buf);
        fs::write(&inbox_path, &buf);
    }
}

// In cognition.rs THINK, after membrane recording:
// Check inbox.bin for parent projections
let inbox = entity.dir.join("inbox.bin");
if let Ok(data) = fs::read(&inbox) {
    if !data.is_empty() {
        absorb_inbox(entity, &data);
        fs::write(&inbox, b"");  // truncate
    }
}
```

---

## Finding 8: Composition inscription still gated on non-empty net_map, not on perception path

**File:** `/opt/repos-code/convergence-engine/src/bin/main/cognition.rs`
**Lines:** 416-461

**Collapsing code:**
```rust
// Line 446:
(!net_map.is_empty()).then(|| {
    let sigma: Vec<(i16, i16)> = net_map.iter()
        .map(|&(sv, tv, _)| (sv as i16, tv as i16))
        .collect();
    entity.state_record.assimilate(
        saios_kernel_v2::engine::ComposedOperator {
            opcode: 0,
            parameter: 0,
            sigma,
            score: 1,
        });
});
```

**Why it collapses:**
The original issue noted that "composition inscription only fires when composition_depth > 0". This was fixed -- the code now inscribes value maps for ALL solves regardless of composition depth (the comment on line 414 confirms this fix). However, the gate `!net_map.is_empty()` still silently drops identity-like solves where `src == tgt` for every cell. An identity transformation IS a valid operator (opcode 0, sigma empty, "no change needed"). It represents the knowledge that this orbit's transformation is trivial. Without inscribing it, the entity has no record that it understood the orbit, and will re-attempt perception on the same orbit indefinitely. The solved orbit IS recorded (line 408), but the vocabulary is not.

**Pseudocode fix:**
```
// Always inscribe the operator, even if the value map is empty (identity).
// An identity operator is a real operator with sigma = [].
let sigma: Vec<(i16, i16)> = net_map.iter()
    .map(|&(sv, tv, _)| (sv as i16, tv as i16))
    .collect();
entity.state_record.assimilate(
    ComposedOperator { opcode: 0, parameter: 0, sigma, score: 1 }
);
```

---

## Summary of Dimensional Collapse Sites

| # | File | Lines | Collapse Type | Severity |
|---|------|-------|---------------|----------|
| 1 | cognition.rs | 7-8, 539-544 | Perception disconnected from evolution | Critical |
| 2 | lineage.rs | 348-388 | DRAIN membrane write uses manual entries + cyclic index folding | High |
| 3 | cognition.rs | 253-265 | Membrane consensus T uses manual entries instead of set_antisym | Medium |
| 4 | cognition.rs | 840-854 | Transmutation T delta uses manual entries | Low |
| 5 | cognition.rs | 882-894 | Residual cocycle uses manual entries, drops last cocycle | Medium |
| 6 | lineage.rs | 364-377 | Drain T builder uses if/else swap + modular folding | High |
| 7 | lineage.rs | full drain | No SPRAY inverse -- vertical knowledge is one-way extraction | Critical |
| 8 | cognition.rs | 446 | Identity solves produce no vocabulary inscription | Low |
