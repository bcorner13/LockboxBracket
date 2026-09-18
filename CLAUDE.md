# Project rules — Lockbox Bracket

This project's parametric risk was concentrated in **one place: the hole pattern**, and it has
been remediated (2026-09-17). Three expressions once described the same spacing concept with
three different divisors, one of which divided by `NumHoles - 2`; the pattern is now driven by
a single formula and is symmetric. The lasting lesson is the quieter one: `VarSet.HoleDiameter`
was a **dead knob** — bound, but to a sketch circle the `Hole` feature ignores — and the audit
script reported the model clean the whole time.

**So: a bound expression is not proof that a Param drives geometry.** When a knob matters here,
flex it and re-measure the solid. Do not trust a clean audit alone.

---

## Hard rules (this project)

These restate the global rules in `~/.claude/CLAUDE.md` with project-specific context.

1. **Everything parametric.** Intake audit (2026-09-17) found the model largely healthy —
   13 bindings, all 4 sketches fully constrained. Two genuine literals (`Fillet.Radius` 8.0,
   `Chamfer.Size` 0.4) are now bound to `OuterFilletRadius` and `ChamferSize`. Three further
   "literals" turned out to be **standard-derived, not hard-coded** — see the audit exemptions
   below; chasing them would have frozen the fastener off-standard. Feature properties outside
   `Length/Length2/Radius/Offset/Size` were this project's blind spot until the audit script
   was extended here.

2. **No fixing geometry by editing raw sketch coordinates.** No prior incident in this project;
   rule applies preventively. Note the standing risk: all four sketches are already
   `FullyConstrained = True`, so any apparent need to "nudge" geometry is a signal that a
   driving Param is missing, not that a coordinate needs editing.

3. **Attach sketches to datum planes, not feature faces.** No prior DAG incident; rule applies
   preventively — and the model is **already compliant**: all four sketches attach to origin
   planes (`XY_Plane`, `XZ_Plane`) via `MapMode = FlatFace`, with `Sketch002`/`Sketch003`
   offset by an `AttachmentOffset.Base.z` bound to `WallHeight`. Preserve this.
   *One lesser fragility exists:* `Pocket.UpToFace` points at `Fillet.Face1`, a feature face.
   That is a topological-naming risk (a renumbered face silently changes the cut), not a DAG
   cycle. Leave it unless it actually breaks; if it does, replace it with `ThroughAll` rather
   than re-picking the face.

4. **Clearance concepts stay decoupled.** This project has exactly one mating interface —
   fastener-in-flange — and it deliberately has **no clearance Param of its own**:

   - **Bolt-hole clearance is delegated to the ISO fit class** on the `Hole` feature:
     `ThreadSize = M3x0.5` with `ThreadFit = Medium` yields the ReadOnly `Diameter = 3.4 mm`
     (ISO 273 medium). The standard is the knob. Adding a Param here would duplicate it and
     let the two drift apart.

   This is *not* a violation of the rule — it is the rule's goal reached by a better route.
   If a lid, retainer, or press fit is ever added, it gets its **own** clearance Param; never
   overload the fastener fit to express it.

---

## Assembly architecture

Single-body part. There is no multi-part assembly, so there is no cross-document link graph to
reason about — but the *physical* relationship still matters and cannot be read off the FCStd:

- The part is an **open-ended U-channel** (a trough), not a closed tray: a 2 mm floor plus two
  2 mm side walls, **open at both ends** along the depth (Y) axis. Verified by point-inside
  probing, not assumed: material exists at the centreline only from z≈0 to z≈2.
- **The lockbox slides in lengthwise** through either open end and rests on the floor. Interior
  cavity is **165.5 (X) × 185 (Y) × 46 (Z) mm**.
- **Mounting flanges** run the full depth along the top outer edge of each side wall, 30 mm wide
  × 4 mm thick, generated as `Pad001` and `Mirrored` about `Sketch002`'s V_Axis. The flanges are
  the only fixing interface — the bracket hangs or bolts by these two strips.
- **The bracket mounts under a desk.** The flanges go up against the desk underside and the
  screws pass **up through the flange into the desk**.
- 🔑 **RETENTION: the bracket is deliberately SHORTER than the box.** The box slides in and
  the **combination dial / lock knob on its front face bottoms against the end of the
  channel**, which is what stops it sliding through. There is no catch or stop feature — the
  knob *is* the stop. This is why `VarSet.Depth` is **185 mm**, measured off the real box on
  2026-09-18, rather than matching the box length. **Do not lengthen `Depth` to match the box
  and do not add a back wall** — either defeats the mechanism and the box slides straight
  through. The far end stays open so it can be pushed back out.
- **`NumHoles` countersunk M3 clearance holes per flange**, `2 × NumHoles` total (**8 at the
  current `NumHoles = 4`** — reduced from 6/flange on 2026-09-17 as overkill; pitch 46.25 mm,
  end margins 23.125 mm): 3.4 mm bore, 6.7 mm × 90° countersink, depth bound to `FlangThickness`.
  Countersinks open **downward** (Ø6.6 at z=48, closing to Ø3.4 by z≈49.7) so the heads seat
  flush on the flange *underside* — verified by measurement, not assumed.
- Both flanges are drilled from **one sketch**: `Sketch003` holds two circles held `Symmetric`
  about the sketch V axis and `Equal` to each other. `Hole` drills both; `LinearPattern` then
  repeats them along the depth. So the mirroring is a sketch constraint, not a feature.
- **Assembly** (`Assembly::AssemblyObject`) holds `Body001` (an `App::Link` to `Body`, grounded
  by `GroundedJoint`) and `2 × NumHoles` M3×10 ISO14582 screws, seated head-top at z=48 with a
  180° rotation about X. With a 4 mm flange, ~6 mm of thread stands proud into the desk.
- Print orientation follows from this: channel open-side-up, flanges flat on the bed,
  **no supports needed**.
- **[UNCONFIRMED]** which lockbox the 165.5 × 195 mm interior is sized around, the material,
  and the load case — see the `[CONFIRM]` markers in `intent.md`.

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `Lockbox Bracket.FCStd` | The entire part: `Body` + in-document `VarSet` + `Assembly` with the screws | — (self-contained) | ✅ audit clean; parametric flex verified |
| `macros/remediate_parametric.FCMacro` | Params, bindings, symmetric spacing, mirrored holes | the FCStd | ✅ idempotent |
| `macros/place_screws.FCMacro` | Seats `2 × NumHoles` screws in the bores | the FCStd | ✅ idempotent; **re-run after changing `NumHoles`** |
| `macros/export_print_files.FCMacro` | Exports the Body alone to `stl/` | the FCStd | ✅ idempotent |
| `macros/export_test_coupon.FCMacro` | Cuts two test pieces from the solid | the FCStd | ✅ idempotent; **does not modify the model** |
| `stl/Lockbox Bracket.stl` | Printable mesh, full part | the FCStd | ✅ watertight, manifold, no self-intersections |
| `stl/Lockbox Bracket-cornertest.stl` | **Quick fit test** — 52 × 31.13 × 52 mm, one wall + flange + 1 bore, 5.7% of the part | the FCStd | ✅ watertight |
| `stl/Lockbox Bracket-testcoupon.stl` | **Width test** — 225.5 × 31.13 × 52 mm, full cross-section, 2 bores, 16.8% of the part | the FCStd | ✅ **printed 2026-09-18, fit confirmed snug** |

Both test pieces are cut from the part's **real open end** up to 8 mm past the first bore, so
they carry the genuine end geometry and the true 24.375 mm end margin — not a slab milled out
of the middle. The box goes in through the same opening it will use for real. Regenerate with
`macros/export_test_coupon.FCMacro`; `PAST_HOLE` at the top controls how far past the bore
they stop.
| `3mf/Lockbox Bracket.3mf` | Creality Print slicer project — PETG, 4 walls | the FCStd | ⚠️ **geometry STALE** (12 holes, 195 mm). Settings are current; graft or re-import and re-slice. |
| `3mf/Lockbox Bracket-widthtest.3mf` | Sliced width-test coupon | the FCStd | ⚠️ stale (195 mm version) — the one that was printed |

**There is no `Params.FCStd`, and that is deliberate.**

> **Deliberate deviation from `PROJECT_BOOTSTRAP.md`:** this is a single-body, single-document
> project, so the VarSet lives **inside** `Lockbox Bracket.FCStd` and expressions use the local
> `VarSet.Name` form, not `<<Params>>#VarSet.Name`. Confirmed by Bradley on 2026-09-17.
> **Do not migrate this to a separate `Params.FCStd`** — a future session "fixing" the
> deviation would rewrite all 13 bindings for no benefit. The cross-document form only earns
> its keep when more than one document shares the variables.

### Known debt and traps

- ⚠️ **Two unrelated things are called "Depth". Do not conflate them:**
  - **`Hole.Depth`** = 4.0 mm, bound to `VarSet.FlangThickness` — how deep the bore drills.
    This one is **healthy**; leave it alone.
  - **`VarSet.Depth`** = 195 mm — the length of the channel along Y, i.e. how far the box
    slides in. This is the dangerous one, below.

- 🔴 **A LARGE JUMP in `VarSet.Depth` silently drops holes. The expression is fine — the
  constraint's SIGN is not.**

  `Sketch003.Constraints[2]` is an **unsigned `Distance`** (`VarSet.Depth / (2 * VarSet.NumHoles)`)
  measured to an edge. "16.25 mm from that edge" has **two** solutions — inboard and outboard —
  and the Sketcher solver converges to whichever is nearer the geometry's *current* position.
  After a big jump the old position is nearer the **outboard** branch, so the circle flips to
  the wrong side of the edge and falls outside the material, where `PartDesign::Hole` cuts
  nothing.

  Proven by path-dependence (2026-09-17). Same destination, two routes:

  | Route to `Depth = 100` | value | circle y | bores |
  |---|---|---|---|
  | jump 195 → 100 | 12.5 ✓ | **−62.5** (outside; part is −50..50) | **6** ✗ |
  | steps 195→180→160→140→120→100 | 12.5 ✓ | −37.5 (correct) | 8 ✓ |

  The arithmetic is identical in both. **It is not a maths problem and not a broken external
  reference** — an earlier note in this file claimed the `Pad001.Face13` reference fails to
  track `Depth`; that was wrong, it tracks fine.

  There is **no error of any kind** when it happens — sketch `FullyConstrained`, every feature
  `Up-to-date`, solid valid and single. And it is partial: at `Depth = 100` only the first
  pattern instance fell outside, giving 6 asymmetric holes instead of 8.

  - `NumHoles` **is** safe (verified at 1, 4, 6 and 8).
  - A **10 mm step is safe**: `Depth` 195 → 185 on 2026-09-18 held all 8 bores with
    symmetric 23.125 mm margins. 20 mm steps also held. The failure needs a big jump.
  - **Workaround today:** change `VarSet.Depth` in steps of ~20 mm rather than one jump, then
    **re-measure the bore count**. Never trust a clean recompute or a clean audit here.
  - **Proper fix, not yet done:** replace the unsigned `Distance` with a signed `DistanceY`
    from the sketch origin, `-Depth/2 + Depth/(2*NumHoles)`. A signed constraint has exactly
    one solution, so the branch ambiguity disappears — and referencing the origin drops the
    `Pad001.Face13` dependency as a bonus. Worth doing before any box-fit iteration.

- **`HoleMarkerDia` (4.0 mm) does not set the bore.** It sizes the `Sketch003` marker circles,
  which `PartDesign::Hole` uses only for *positions*; the bore comes from `ThreadSize` +
  `ThreadFit`. It was called `HoleDiameter` until 2026-09-17, which invited exactly the wrong
  fix. **Do not "correct" it to 3.4 mm** — it drives nothing but the marker.
- **Three sketches use external geometry from feature faces:** `Sketch001`→`Pad.Face1`,
  `Sketch002`→`Pocket.Face6/Face12`, `Sketch003`→`Pad001.Face13`. Attachment is clean (origin
  planes), so this is topological-naming fragility, not a DAG cycle — a renumbered face would
  silently move geometry. Left as-is deliberately: re-cutting these would disturb working,
  fully-constrained sketches. **If holes or walls ever jump after an upstream edit, look here
  first.**
- **`LinearPattern.Mode` is `Spacing`**, so `Offset` is the live driver and `Length` is inert.
  Both once evaluated to 27.857, which made them easy to confuse. Edit `Offset`.
- The internal document name is **`Unnamed`** (label is `Lockbox Bracket`). Scripts must use
  `FreeCAD.getDocument("Unnamed")` — `getDocument("Lockbox Bracket")` fails.
- The Fasteners workbench **re-labels its screws on recompute**, so labels set by
  `place_screws.FCMacro` do not stick. Cosmetic; identify screws by `Placement`, not `Label`.
- **EXPORT TRAP — the screws will end up in your print file.** The community macro
  `3D_Printer_3mf_Workflow.FCMacro` exports `FreeCADGui.Selection` (its own line 2402), not the
  printable body. Running it with the `Assembly` or the whole document selected meshes all
  twelve M3×10 screws into the 3MF. That happened on 2026-09-17 and produced a ~20 MB file
  that no slicer can use.

  **Always export via `macros/export_print_files.FCMacro`**, which pins the selection to
  `Body` and writes `stl/` + `3mf/` itself. It leaves `Body` selected afterwards, so if you
  then want the workflow macro's slicer-settings round-trip, run it immediately after and it
  will pick up the bracket alone. A correct export is ~75 KB and spans z = 0..52; if `ZMax`
  reads 58, the screws are in it.

---

## Params variables (summary)

In-document `VarSet`, 13 variables, all `App::PropertyLength` except `NumHoles`
(`App::PropertyInteger`):

- **Shell geometry:** `Width` (169.5), `Depth` (**185.0** — see retention note), `WallHeight` (48.0),
  `WallThickness` (2.0), `CornerRadius` (7.0)
- **Flange:** `FlangWidth` (30.0), `FlangThickness` (4.0), `FlangRadius` (2.0)
  *(note the spelling — `Flang`, not `Flange`; it is consistent across all bindings, so leave it)*
- **Dress-up:** `OuterFilletRadius` (8.0 → `Fillet.Radius`), `ChamferSize` (0.4 → `Chamfer.Size`)
- **Fastener:** `NumHoles` (**4** per flange, 8 total), `ScrewLength` (10.0, used by
  `place_screws.FCMacro`), `HoleMarkerDia` (4.0 — sketch marker only, **not** the bore)

Hole spacing derives entirely from `Depth` and `NumHoles`:
`Sketch003.Constraints[2] = Depth / (2 × NumHoles)` (end margin) and
`LinearPattern.Offset = Depth / NumHoles` (pitch) — equal margins at both ends by construction.

---

## How to verify your change didn't break parametric

After any FreeCAD edit, before considering the task done:

```bash
python3 scripts/audit_parametric.py
```

This script flags:
- Sketches with 0 constraints
- Sketches with dimensional constraints lacking expression bindings
- Sketches attached to feature faces (DAG risk)
- Params variables used nowhere (dead Params)

The script is authoritative. If it reports violations, fix them via a `macros/*.FCMacro` change
before saving or committing — never by direct coordinate edits or FCStd XML surgery.

**Audit exemptions and known blind spots for this project — a clean run here is necessary but
not sufficient:**

- This copy came from `MagicCardBox`, which carries fixes for **two** enum bugs in the canonical
  script (the `DIMENSIONAL_TYPES` constraint table and the Pad/Pocket `Type` enum). Do not
  replace it with the root canonical copy.
- **Third bug, found here 2026-09-17 and fixed in this copy:** the script never examined
  `PartDesign::Hole` at all. It now does, checking `Depth` and only when
  `DepthType = Dimension` (index 0; `ThroughAll` makes `Depth` inert). The canonical copy and
  every other project still lack this.
- **Deliberately exempt — standard-derived, not literals:**
  - `Hole.Diameter` — **ReadOnly**, computed from `ThreadSize = M3x0.5` + `ThreadFit = Medium`
    (ISO 273 → 3.4 mm).
  - `Hole.HoleCutDiameter` / `HoleCutCountersinkAngle` / `HoleCutDepth` — from the ISO
    countersink table while `HoleCutCustomValues = False`.

    Binding any of these would require forcing custom values and freezing the fastener
    off-standard. **Leave them unbound.** Change the fastener by changing `ThreadSize`.
- The script still does not check `TaperAngle`, `Placement`, sketch `AttachmentOffset`, datum
  attachment, or external geometry. This project relies on two `AttachmentOffset.Base.z`
  bindings and three feature-face external-geometry references the audit cannot see — verify
  those by hand via the MCP bridge.
- It also cannot see the `Assembly`: screw placement is validated by re-running
  `place_screws.FCMacro` and checking each screw against a bore axis, not by the audit.

---

## Memory files (deeper context)

`~/.claude/projects/-Users-bradleycorner/memory/MEMORY.md` indexes the persistent memories.
If you're unsure *why* a rule exists, read those files first.

Relevant to this project:
- `reference_audit_parametric_type_bug.md` — the two known enum bugs in the audit script and
  the list of things it does not check. Read before trusting a clean audit.
- `feedback_freecad_use_mcp.md` — why `.FCStd` is never touched with shell tools.
- `feedback_canonical_reference_only.md` — scaffolding comes from the Spade Connector reference.
- No project-scoped memory for Lockbox Bracket itself yet — this project was bootstrapped
  2026-09-17 from a pre-existing model.

---

## Workflow notes

**Invariant (apply to every FreeCAD project — do not edit):**

- **Inspect/edit FreeCAD models via the MCP bridge — never with shell tools.** Do **not**
  `unzip`/`grep`/`cat`/`sed`/`strings`/etc. a `.FCStd`. Use the FreeCAD Robust MCP server:
  `get_connection_status` first, then `open_document`, `list_objects`, `inspect_object`,
  `execute_python`, and macros. This is **enforced** by a PreToolUse hook in
  `.claude/settings.json` (copied from the root `settings.template.json` during bootstrap) —
  raw shell access to `.FCStd` is blocked. Only fall back to read-only `unzip` if the MCP
  bridge is genuinely unreachable, and ask the user first.
- **MCP server auto-starts with FreeCAD.** If an `mcp__freecad__*` call fails, the right
  interpretation is "FreeCAD isn't running" — ask whether to launch it. Do **not** silently
  fall back to `unzip` + XML parsing, and do **not** retry the same MCP call.
- **Write changes as `macros/*.FCMacro` files**, not direct XML edits. Reasons: reviewable,
  re-runnable, idempotent-friendly, uses FreeCAD's own serialization.
- **Cross-document expressions**: use the canonical form `<<Params>>#VarSet.VarName`. The
  shorter `<<Params>>.VarName` form sometimes fails with "Params not found."
- **Run `python3 scripts/audit_parametric.py` before committing.** If it reports violations,
  fix via macro, not by editing FCStd XML.

**Project-specific:**

- **Document name trap:** the internal name is `Unnamed`, the label is `Lockbox Bracket`. Use
  `FreeCAD.getDocument("Unnamed")` in macros and `execute_python`.
- **Expressions use the local `VarSet.Name` form, not `<<Params>>#VarSet.Name`** — see the
  deliberate-deviation note above. The invariant bullet on cross-document expressions applies
  only if a second document is ever added.
- **`inspect_object` is broken on shape-bearing objects** in FreeCAD 1.1.3 (global rule #8) —
  it returns a bare `"Failed to get object"` for sketches, pads, and bodies. Read those via
  `execute_python` (`sk.Geometry`, `sk.Constraints`, `sk.ExpressionEngine`, `obj.Shape`).
  It works fine on the `VarSet`.
- **`execute_python` returns over XML-RPC, so dict keys must be strings.** An integer-keyed
  dict in `_result_` fails with `dictionary key must be string`.
- **Changing `NumHoles` is a two-step operation:** recompute the Body, then re-run
  `place_screws.FCMacro` so the screw set matches the new bore count. The macro reads bore
  positions **from the solid**, so it cannot drift from the geometry.
- Both macros are symlinked into `~/Library/Application Support/FreeCAD/v1-1/Macro/`, so they
  appear under Macro → Macros… Neither saves the document.
- Everything lives at the project root; there is no `cad/` subdirectory.

---

## Print profile

**No successful test print yet — profile below is INTENDED, not validated.** Do not treat it
as the working profile until a part comes off the plate. Fill the real one from that print.

Held in `3mf/Lockbox Bracket.3mf` (Creality Print project). Needs a re-slice in Creality Print
— the settings were changed after the last slice.

| Setting | Value | Notes |
|---|---|---|
| Printer | Creality K2 Plus, 0.4 nozzle | |
| Material | **CR-PETG** (slot 4) | Switched from Hyper PLA-CF 2026-09-17. PLA creeps under sustained load; this bracket carries a box continuously. PLA-CF is also brittle against shock loading. |
| Layer height | 0.16 mm (first layer 0.2) | 1219 layers on end |
| Walls | **4** | Raised from 2. The 4 mm flange is built from perimeters, not shells, in this orientation — at 2 walls its fastener-bearing core was ~2.3 mm of 15% infill. |
| Infill | 15% grid | Adequate once walls carry the flange |
| Top / bottom layers | 5 / 4 | Applies to the open channel ends only; not load-critical |
| Supports | **None** | The 90° countersinks land at 45° |
| Brim | auto, 5 mm | Needed: 195 mm tall on a 225.5 × 52 mm footprint |
| Orientation | **On end — part Y vertical** | Deliberate. Puts the installed load axis (part Z) in the layer plane; interlayer tension falls along the channel, which carries almost nothing. **Do not lay it flat.** |
| Est. print | ~3h36m / 154 g / $4.93 at 2 walls | Re-slice for the 4-wall figure |

**Watch on the first print:** the M3 bores are *horizontal* in this orientation, so they print
slightly undersize (droop at the top of the bore) and PETG shrinks more than PLA. If the M3×10
screws bind, do **not** edit geometry — change `Hole.ThreadFit` from `Medium` to `Loose`
(ISO 273 coarse → Ø3.6 mm). That is the parametric knob for this fit; see hard rule #4.

`3mf/Lockbox Bracket.3mf` was sliced on 2026-09-17 from the pre-remediation model, but there is
no confirmed print result. Do not treat it as a validated profile, and re-slice after the hole
pattern change in `plan.md` lands. Fill this section from the first actual successful print —
material, layer height, walls, infill, orientation — not from slicer defaults.
