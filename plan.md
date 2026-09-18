# Plan — Lockbox Bracket

**Status:** awaiting approval. Per `PROJECT_BOOTSTRAP.md` Step 5, no geometry changes are
executed until this plan is approved.

**Scope:** this is a *remediation* plan, not an initial build. The model already exists and is
geometrically healthy. The work below closes the parametric gaps found in the intake audit.

---

## Intake audit — what was measured

Read via the FreeCAD MCP bridge (FreeCAD 1.1.3, GUI, xmlrpc). All facts below are measured,
not inferred.

**Healthy:**

- Single valid manifold solid, 1 solid, volume 147 019.56 mm³, bbox 225.5 × 195 × 52 mm.
- Zero objects in an error or touched state.
- All 4 sketches report `FullyConstrained = True`.
- All 4 sketches attach to **origin planes** (`XY_Plane`, `XZ_Plane`) with `MapMode = FlatFace`.
  Global hard rule #3 (no feature-face sketch attachment) is **already satisfied** — no
  remediation needed.
- 13 expression bindings already in place, including both `AttachmentOffset.Base.z` bindings.

**Defects to fix:**

| # | Object | Property | Value | Problem |
|---|---|---|---|---|
| 1 | `Fillet` | `Radius` | 8.0 mm | Hard-coded. Flagged by audit. |
| 2 | `Chamfer` | `Size` | 0.4 mm | Hard-coded. Flagged by audit. |
| 3 | `Hole` | `Diameter` | 3.4 mm | Hard-coded. **Missed by the audit script.** |
| 4 | `Hole` | `HoleCutDiameter` / `HoleCutCountersinkAngle` | 6.7 mm / 90° | Hard-coded. Missed by audit. |
| 5 | `VarSet.HoleDiameter` | — | 4.0 mm | **Dead knob.** Drives only the `Sketch003` circle, which `PartDesign::Hole` ignores (it uses the sketch as a position reference and cuts at its own `Diameter`). Changing this variable changes nothing in the solid. |
| 6 | `Sketch003` + `LinearPattern` | spacing | — | **Three inconsistent divisors** for one concept. |

### Defect 6 in detail

Three expressions describe the same hole pattern using three different formulas:

```
Sketch003.Constraints[2] = Depth / NumHoles            = 32.5     (first-hole offset)
LinearPattern.Length     = Depth - Depth/(NumHoles-2)  = 139.286  (span)
LinearPattern.Offset     = Depth / (NumHoles + 1)      = 27.857   (inert in Length mode)
```

Measured result: 6 holes at y = −65, −37.143, −9.286, 18.571, 46.429, 74.286. Spacing is a
uniform 27.857 mm, but the end margins are **32.5 mm at one end and 23.21 mm at the other** —
the pattern is not centred. `NumHoles - 2` in the span also means the model breaks if
`NumHoles` is ever set to 2.

---

## PARAMETERS

`VarSet` stays **in-document** (deliberate deviation from the bootstrap's `Params.FCStd`
requirement — see `CLAUDE.md`). Expressions keep the local `VarSet.Name` form.

### Existing (unchanged)

| Name | Type | Value |
|---|---|---|
| `Width` | `App::PropertyLength` | 169.5 mm |
| `Depth` | `App::PropertyLength` | 195.0 mm |
| `WallHeight` | `App::PropertyLength` | 48.0 mm |
| `WallThickness` | `App::PropertyLength` | 2.0 mm |
| `CornerRadius` | `App::PropertyLength` | 7.0 mm |
| `FlangWidth` | `App::PropertyLength` | 30.0 mm |
| `FlangThickness` | `App::PropertyLength` | 4.0 mm |
| `FlangRadius` | `App::PropertyLength` | 2.0 mm |
| `NumHoles` | `App::PropertyInteger` | 6 |

### To add

| Name | Type | Value | Drives | Why a new knob |
|---|---|---|---|---|
| `OuterFilletRadius` | `App::PropertyLength` | 8.0 mm | `Fillet.Radius` | Distinct concept from `CornerRadius` (7.0) — different value today, so per global rule #5 it does not get folded into the existing variable. |
| `ChamferSize` | `App::PropertyLength` | 0.4 mm | `Chamfer.Size` | First-layer edge break; its own concern. |
| `HoleCsinkDiameter` | `App::PropertyLength` | 6.7 mm | `Hole.HoleCutDiameter` | M3 countersink head Ø. |
| `HoleCsinkAngle` | `App::PropertyAngle` | 90.0 ° | `Hole.HoleCutCountersinkAngle` | Countersink included angle. |

### To retask

| Name | From | To | Why |
|---|---|---|---|
| `HoleDiameter` | 4.0 mm, drives an inert sketch circle | **3.4 mm**, bound to `Hole.Diameter` | Makes the dead knob live. 3.4 mm is the standard M3 clearance bore and is the diameter the solid is actually cut at today, so **the geometry does not change** — the variable simply starts controlling it. The `Sketch003` circle keeps its binding and stays inert-by-design (it is a position reference). |

Fastener clearance is deliberately kept as its own concept (global rule #4): `HoleDiameter`
is a **bolt-hole clearance** interface and is not shared with any other fit.

---

## FEATURE TREE

Unchanged. No features are added, removed, or reordered. Tip stays `LinearPattern`.

```
Body
├── Sketch         (XY_Plane)  outer footprint      → Width, Depth
├── Pad            48 mm                            → WallHeight
├── Fillet         2 edges, r8                      → OuterFilletRadius      [BIND]
├── Sketch001      (XZ_Plane)  wall section         → WallThickness ×3, CornerRadius
├── Pocket         UpToFace Fillet.Face1            (hollows the channel)
├── Sketch002      (XY_Plane, offset z=WallHeight)  flange profile → FlangWidth, FlangRadius
├── Pad001         4 mm                             → FlangThickness
├── Mirrored       about Sketch002 V_Axis           (second flange)
├── Chamfer        12 edges, 0.4                    → ChamferSize            [BIND]
├── Sketch003      (XY_Plane, offset z=WallHeight)  hole centre → HoleDiameter, FlangWidth/2, spacing
├── Hole           M3 c'sunk, depth=FlangThickness  → HoleDiameter, HoleCsink*  [BIND]
└── LinearPattern  6 occurrences along Sketch003 V_Axis → NumHoles, spacing  [REVISE]
```

---

## CONSTRAINT STRATEGY

1. **No sketch geometry is touched.** All four sketches are already fully constrained and
   correctly bound. No coordinate edits (global rule #2), no constraint deletion/re-adding.
2. **Bindings are added via `setExpression`** on feature properties only — `Fillet.Radius`,
   `Chamfer.Size`, `Hole.Diameter`, `Hole.HoleCutDiameter`, `Hole.HoleCutCountersinkAngle`.
3. **Hole spacing is re-expressed** so one concept has one formula. Proposed symmetric scheme:

   ```
   Sketch003.Constraints[2] = Depth / (2 * NumHoles)     = 16.25   (half-pitch edge margin)
   LinearPattern.Length     = Depth - Depth / NumHoles   = 162.5   (span)
   LinearPattern.Occurrences= NumHoles                             (unchanged)
   ```

   Result: uniform 32.5 mm pitch, **equal 16.25 mm margins at both ends**, and no division by
   `NumHoles - 2`, so the pattern stays valid down to `NumHoles = 1`. **This moves the holes.**
   It is the only change in the plan that alters the printed part.

   *Alternative if the current 32.5 mm end margin is deliberate:* keep
   `Constraints[2] = Depth / NumHoles` and set `LinearPattern.Length = Depth - 2 * (Depth / NumHoles)`
   — that centres the pattern while preserving the existing first-hole position, at 26 mm pitch.
   **Say which you want; symmetric half-pitch is the default if you don't.**

4. **Delivery:** one reviewable, idempotent `macros/bind_unbound_params.FCMacro` (global rule
   #7 — no typed MCP tool covers `setExpression`, so this is a legitimate macro/Python case).
   The macro skips any Param that already exists and re-applies bindings safely on re-run.

---

## TOOLING FIX (outside the model)

`scripts/audit_parametric.py` line 172 checks only
`["Length", "Length2", "Radius", "Offset", "Size"]`, so it silently misses `Hole.Diameter`,
`Hole.Depth`, and the countersink dimensions — defects 3 and 4 above passed a "clean" audit.
This project's copy will be extended to cover `Diameter` and the `Hole` property group.

The root canonical copy is **not** edited (per standing instruction: propagate fixes outward
only with Bradley's say-so). This is a third known bug in that script, after the two recorded
in memory.

---

## VALIDATION

Run in order; all must pass before the work is called done.

1. `python3 scripts/audit_parametric.py` → **0 issues** (with the extended check in place).
2. Recompute the document → no object in an error or touched state.
3. Shape integrity: `isValid() == True`, exactly **1 solid**, still watertight.
4. **Geometry regression:** volume and bbox compared against the pre-change baseline
   (147 019.56 mm³ / 225.5 × 195 × 52 mm). Steps 1–2 and the binding work must leave both
   *identical* — the retasked `HoleDiameter` is specifically chosen not to move material.
   Only the step-3 spacing change may alter volume, and it must alter it only by relocating
   holes (hole count and bore unchanged).
5. **Parametric proof:** flex each new Param (e.g. `OuterFilletRadius` 8 → 6, `NumHoles`
   6 → 8), recompute, confirm the solid changes as expected and stays valid, then restore the
   original values and confirm the volume returns to baseline exactly.
6. Re-measure hole positions via `Shape.Faces` cylinder centres to confirm symmetric margins.

Nothing is saved to disk until validation passes and Bradley approves the result
(`PROJECT_BOOTSTRAP.md` Step 6 — never save automatically).
