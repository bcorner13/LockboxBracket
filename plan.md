# Plan — Lockbox Bracket

**Status: EXECUTED 2026-09-17.** All steps applied via the two macros, validated, and saved.
Audit clean; parametric flex verified. Kept as the record of what was changed and why.

Remediation of an existing model — not an initial build.

### Result

| Check | Result |
|---|---|
| Audit | ✅ 0 issues |
| Bores | 12 (`2 × NumHoles`), both flanges, uniform 32.5 mm pitch, **equal 16.25 mm end margins** |
| Screws | 12 M3×10 seated, max axis offset **0.000 mm**, heads at z = 48, all in `Assembly.Group` |
| Solid | valid, 1 solid, 146 686.19 mm³, 225.5 × 195 × 52 mm |
| Flex test | `NumHoles` 6→8 gives 16 symmetric bores; 8→6 restores 146 686.19 mm³ **exactly** |

---

## Intake audit — what was measured

Read via the FreeCAD MCP bridge (1.1.3, GUI, xmlrpc). Measured, not inferred.

**Healthy:** one valid manifold solid, 147 019.56 mm³, 225.5 × 195 × 52 mm, no object in an
error state, all 4 sketches `FullyConstrained = True`, all attached to **origin planes**,
13 expressions already bound including both `AttachmentOffset.Base.z` offsets.

**Defects:**

| # | Object | Property | Value | Problem |
|---|---|---|---|---|
| 1 | `Fillet` | `Radius` | 8.0 mm | Hard-coded. Flagged by audit. |
| 2 | `Chamfer` | `Size` | 0.4 mm | Hard-coded. Flagged by audit. |
| 3 | `Hole` | `Diameter` | 3.4 mm | ~~Hard-coded~~ **WITHDRAWN.** Property is `ReadOnly` — derived from `ThreadSize = M3x0.5` + `ThreadFit = Medium` (ISO 273 medium clearance → 3.4 mm). Cannot be bound, and should not be: the standard is the driver. |
| 4 | `Hole` | `HoleCutDiameter` / `HoleCutCountersinkAngle` | 6.7 mm / 90° | ~~Hard-coded~~ **WITHDRAWN.** Auto-computed from the ISO countersink table while `HoleCutCustomValues = False`. Binding them would require forcing custom values, freezing the countersink off-standard. |
| 5 | `VarSet.HoleDiameter` | — | 4.0 mm | **Dead knob** — drives only the `Sketch003` marker circle, which `PartDesign::Hole` ignores (the sketch supplies positions; the bore comes from `ThreadSize`). Proof: marker Ø4.0, measured bore Ø3.4. |
| 6 | `Sketch003` + `LinearPattern` | spacing | — | **Three divisors for one concept** (below). |
| 7 | `Sketch003` / `Hole` | — | — | **Holes on one flange only** (all 6 bores at x = +97.75). |
| 8 | 6 × `Screw00n` | `Placement` | — | **Screws not seated.** Five are stacked at (−3.25, 0, 0); one at (54.12, −108.204, 0). None is at a bore. They are also **outside** `Assembly.Group`. |
| 9 | `Joint` ("Fixed") | `Reference1/2` | `None` | Empty, non-functional joint. |

### Defect 6 in detail

```
Sketch003.Constraints[2] = Depth / NumHoles            = 32.5     (first-hole offset)
LinearPattern.Length     = Depth - Depth/(NumHoles-2)  = 139.286  (span)
LinearPattern.Offset     = Depth / (NumHoles + 1)      = 27.857   (inert in Length mode)
```

Measured bores at y = −65, −37.143, −9.286, 18.571, 46.429, 74.286: uniform 27.857 mm pitch
but **end margins of 32.5 vs 23.21 mm**. `NumHoles - 2` also divides by zero at `NumHoles = 2`.

### Additional fragility (not remediated — documented only)

Three sketches use **external geometry from feature faces**: `Sketch001`→`Pad.Face1`,
`Sketch002`→`Pocket.Face6/Face12`, `Sketch003`→`Pad001.Face13`. Attachment is clean, so this
is a topological-naming risk, not a DAG cycle. Left alone deliberately: re-cutting these
references would disturb working, fully-constrained sketches for no functional gain.

---

## PARAMETERS

`VarSet` stays **in-document** (deliberate deviation — see `CLAUDE.md`), local `VarSet.Name` form.

### Existing (unchanged)

`Width` 169.5 · `Depth` 195.0 · `WallHeight` 48.0 · `WallThickness` 2.0 · `CornerRadius` 7.0 ·
`FlangWidth` 30.0 · `FlangThickness` 4.0 · `FlangRadius` 2.0 · `NumHoles` 6

### To add

| Name | Type | Value | Drives |
|---|---|---|---|
| `OuterFilletRadius` | `App::PropertyLength` | 8.0 mm | `Fillet.Radius` — distinct from `CornerRadius` (7.0); separate concept, separate knob (global rule #5) |
| `ChamferSize` | `App::PropertyLength` | 0.4 mm | `Chamfer.Size` |
| `ScrewLength` | `App::PropertyLength` | 10.0 mm | screw generation (M3×10) |

No countersink Params: those dimensions are standard-derived (defects 3–4, withdrawn).

### To rename

`HoleDiameter` → **`HoleMarkerDia`** (stays 4.0 mm). The variable is real but its name lies:
it sizes the `Sketch003` marker circle, which only supplies hole *positions*. The actual bore
is set by `Hole.ThreadSize = M3x0.5` with `ThreadFit = Medium`. Renaming it is the honest fix —
"one knob, one concern" (global rule #5) — and stops a future session from "correcting" it to
3.4 mm in the belief that it drives the bore. `Sketch003.Constraints[0]` is rebound to the new
name; the old property is removed.

**Bolt-hole clearance** therefore has no Param of its own by design: it is delegated to the ISO
fit class on the `Hole` feature. Recorded in `CLAUDE.md` so rule #4 is not read as violated.

---

## FEATURE TREE

No features added, removed, or reordered. Tip stays `LinearPattern`.

```
Body
├── Sketch         (XY_Plane)   footprint          → Width, Depth
├── Pad            48                              → WallHeight
├── Fillet         2 edges                         → OuterFilletRadius        [BIND]
├── Sketch001      (XZ_Plane)   wall section       → WallThickness ×3, CornerRadius
├── Pocket         UpToFace Fillet.Face1
├── Sketch002      (XY_Plane @ z=WallHeight)       → FlangWidth, FlangRadius
├── Pad001         4                               → FlangThickness
├── Mirrored       about Sketch002 V_Axis          (second flange)
├── Chamfer        12 edges                        → ChamferSize              [BIND]
├── Sketch003      (XY_Plane @ z=WallHeight)       → HoleDiameter, FlangWidth/2, spacing
│                  + SECOND CIRCLE mirrored about V_Axis                      [ADD]
├── Hole           M3 c'sunk, Depth=FlangThickness → HoleDiameter, HoleCsink* [BIND]
│                  Profile widened from Edge1 to the whole sketch             [CHANGE]
└── LinearPattern  → NumHoles, symmetric span                                 [REVISE]
```

## CONSTRAINT STRATEGY

1. **No existing sketch geometry is touched**; no coordinate edits (global rule #2). All four
   sketches stay fully constrained.

2. **Bindings via `setExpression`** on feature properties: `Fillet.Radius` →
   `OuterFilletRadius`, `Chamfer.Size` → `ChamferSize`. The `Hole` feature's dimensions are
   left to the ISO standard (defects 3–4 withdrawn) and recorded as audit exemptions.

3. **Symmetric spacing** — one concept, one formula.

   `LinearPattern.Mode` is **`Spacing`**, so `Offset` is the live driver and `Length` is
   inert. (Both happened to evaluate to 27.857, which is why the two are easy to confuse —
   the mode had to be read to tell them apart.)

   ```
   Sketch003.Constraints[2] = Depth / (2 * NumHoles)   = 16.25   (half-pitch end margin)
   LinearPattern.Offset     = Depth / NumHoles         = 32.5    (pitch)     [LIVE]
   LinearPattern.Occurrences= NumHoles                           (unchanged)
   LinearPattern.Length     — expression cleared                 (inert; contradictory)
   ```

   First bore at 16.25 from the near edge, last at 16.25 + 5 × 32.5 = 178.75, leaving
   195 − 178.75 = **16.25 at the far edge — equal margins**. No `NumHoles - 2` term, so it
   stays valid down to `NumHoles = 1`.

4. **Mirror the holes onto the second flange** by adding a second circle to `Sketch003`,
   constrained `Symmetric` about the sketch V_Axis and `Equal` to the first — rather than by
   adding a `Mirrored`/`MultiTransform` feature. Reasons: the flanges are already symmetric
   about x = 0, a sketch-level symmetry constraint is inherently parametric, it keeps the
   feature tree flat, and it avoids chaining one transform feature onto another (which
   PartDesign does not support without `MultiTransform`). `Hole.Profile` is widened from
   `Edge1` to the whole sketch so both circles are drilled; `LinearPattern` then patterns both.
   Result: **2 × NumHoles = 12 bores.**

5. **Screws seated in the bores.** The head of an ISO14582 screw sits at its placement origin
   with the shank running local −Z (verified from the shape: local z ∈ [−10, 0]). With a 180°
   rotation about X the shank points +Z. So each screw gets
   `Placement = (x_bore, y_bore, WallHeight)`, rotation 180° about X — head top flush with the
   flange underside at z = 48, thread running up through the 4 mm flange and ~6 mm into the desk.
   The screw set is regenerated to `2 × NumHoles`, and the screws are moved **into**
   `Assembly.Group`.

6. **Delivery:** two reviewable, idempotent macros under `macros/` (global rule #7 — no typed
   MCP tool covers `setExpression`, sketch symmetry constraints, or Fasteners-WB object
   creation):
   - `remediate_parametric.FCMacro` — steps 1–4
   - `place_screws.FCMacro` — step 5; re-run after any change to `NumHoles`

---

## TOOLING FIX

`scripts/audit_parametric.py` checks only `Length/Length2/Radius/Offset/Size`, so
`PartDesign::Hole` is effectively unaudited — defects 3 and 4 passed a "clean" run. This
project's copy gains `Diameter` and `Depth` coverage. The root canonical copy is **not**
edited (standing instruction). This is a third bug in that script, after the two in memory.

---

## VALIDATION

1. `python3 scripts/audit_parametric.py` → 0 issues.
2. Recompute → no object in an error or touched state.
3. `isValid() == True`, exactly 1 solid in the Body.
4. **Bore census:** exactly `2 × NumHoles` = 12 bores of Ø3.4, at x = ±97.75, with **equal
   end margins** (16.25 mm) and uniform 32.5 mm pitch — re-measured from `Shape.Faces`.
5. **Screw census:** 12 screws, each within 0.01 mm of a bore axis, head top at z = 48.
6. **Parametric proof:** set `NumHoles` 6 → 8, recompute, confirm 16 symmetric bores; flex
   `OuterFilletRadius` 8 → 6; restore both and confirm volume returns to the pre-change
   baseline of 147 019.56 mm³ **exactly**.
7. Steps 1–2 and the bindings must not move material; only the spacing fix and the mirrored
   flange may change volume, and only by relocating/adding bores.

Nothing is saved until validation passes (`PROJECT_BOOTSTRAP.md` Step 6).
