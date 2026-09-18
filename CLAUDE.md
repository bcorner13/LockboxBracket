# Project rules — Lockbox Bracket

This project's parametric risk is concentrated in **one place: the hole pattern**. Three
different expressions (`Sketch003.Constraints[2]`, `LinearPattern.Length`,
`LinearPattern.Offset`) currently describe the same spacing concept with three different
divisors, one of which divides by `NumHoles - 2`. The rest of the model is clean and fully
bound — do not go looking for trouble elsewhere, and do not "tidy" the hole expressions
without reading `plan.md` first. The second risk is quieter: `VarSet.HoleDiameter` was a
**dead knob** that drove nothing in the solid, and the audit script did not catch it.

---

## Hard rules (this project)

These restate the global rules in `~/.claude/CLAUDE.md` with project-specific context.

1. **Everything parametric.** Intake audit (2026-09-17) found the model largely healthy —
   13 bindings, all 4 sketches fully constrained — with hard-coded values isolated to the
   *dress-up and hole features*: `Fillet.Radius` (8.0), `Chamfer.Size` (0.4), `Hole.Diameter`
   (3.4), and the two countersink dimensions. Those are the worst offenders, and they are the
   offenders precisely because `scripts/audit_parametric.py` only flagged two of the five.
   Feature properties outside `Length/Length2/Radius/Offset/Size` are this project's blind spot.

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

4. **Clearance concepts stay decoupled.** This project has exactly one mating interface, so it
   has exactly one clearance Param:
   - `HoleDiameter` (3.4 mm) — **bolt-hole clearance** for M3 countersunk fasteners passing
     through the mounting flanges into whatever the bracket is fixed to.

   There is no press fit, rail fit, or lid fit here. If a lid or retainer is ever added, it
   gets its **own** clearance Param — never reuse `HoleDiameter`.

---

## Assembly architecture

Single-body part. There is no multi-part assembly, so there is no cross-document link graph to
reason about — but the *physical* relationship still matters and cannot be read off the FCStd:

- The part is an **open-ended U-channel** (a trough), not a closed tray: a 2 mm floor plus two
  2 mm side walls, **open at both ends** along the depth (Y) axis. Verified by point-inside
  probing, not assumed: material exists at the centreline only from z≈0 to z≈2.
- **The lockbox slides in lengthwise** through either open end and rests on the floor. Interior
  cavity is **165.5 (X) × 195 (Y) × 46 (Z) mm**.
- **Mounting flanges** run the full depth along the top outer edge of each side wall, 30 mm wide
  × 4 mm thick, generated as `Pad001` and `Mirrored` about `Sketch002`'s V_Axis. The flanges are
  the only fixing interface — the bracket hangs or bolts by these two strips.
- **Six countersunk M3 clearance holes per flange** (3.4 mm bore, 6.7 mm × 90° countersink,
  depth bound to `FlangThickness`). Countersinks open **upward**, so heads sit flush with the
  flange top.
- Print orientation follows from this: channel open-side-up, flanges flat on the bed,
  **no supports needed**.
- **[UNCONFIRMED]** what the bracket mounts to and which lockbox it is sized around — see the
  `[CONFIRM]` markers in `intent.md`. Do not treat the 169.5 × 195 mm interior as validated
  against a real box until Bradley says so.

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `Lockbox Bracket.FCStd` | The entire part: `Body` + in-document `VarSet` | — (self-contained) | ⚠️ 5 unbound literals + 1 dead Param + inconsistent hole spacing — see `plan.md` |
| `3mf/Lockbox Bracket.3mf` | Sliced print file (2026-09-17) | the FCStd | ⚠️ sliced from the **pre-remediation** model; re-slice after the hole pattern is fixed |

**There is no `Params.FCStd`, and that is deliberate.**

> **Deliberate deviation from `PROJECT_BOOTSTRAP.md`:** this is a single-body, single-document
> project, so the VarSet lives **inside** `Lockbox Bracket.FCStd` and expressions use the local
> `VarSet.Name` form, not `<<Params>>#VarSet.Name`. Confirmed by Bradley on 2026-09-17.
> **Do not migrate this to a separate `Params.FCStd`** — a future session "fixing" the
> deviation would rewrite all 13 bindings for no benefit. The cross-document form only earns
> its keep when more than one document shares the variables.

### Known debt

- `Hole.Diameter = 3.4 mm` is hard-coded while `VarSet.HoleDiameter = 4.0 mm` drives only the
  `Sketch003` circle. `PartDesign::Hole` uses its profile sketch as a **position reference** and
  cuts at its own `Diameter`, so that circle's size is inert and the variable is dead. Do not
  "fix" this by changing `Hole.Diameter` to 4.0 — 3.4 mm is correct M3 clearance and the bore is
  right; the *binding* is what's missing.
- The internal document name is **`Unnamed`** (label is `Lockbox Bracket`). Scripts must use
  `FreeCAD.getDocument("Unnamed")` or resolve by label — `getDocument("Lockbox Bracket")` fails.

---

## Params variables (summary)

In-document `VarSet`, 9 live variables, all `App::PropertyLength` except `NumHoles`
(`App::PropertyInteger`):

- **Shell geometry:** `Width` (169.5), `Depth` (195.0), `WallHeight` (48.0),
  `WallThickness` (2.0), `CornerRadius` (7.0)
- **Flange:** `FlangWidth` (30.0), `FlangThickness` (4.0), `FlangRadius` (2.0)
  *(note the spelling — `Flang`, not `Flange`; it is consistent across all bindings, so leave it)*
- **Fastener:** `NumHoles` (6), `HoleDiameter` (4.0 — **currently dead**, see above)

`plan.md` adds `OuterFilletRadius`, `ChamferSize`, `HoleCsinkDiameter`, `HoleCsinkAngle`.

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
- **Third bug, found here 2026-09-17:** the feature-dimension check covers only
  `Length/Length2/Radius/Offset/Size`, so `Hole.Diameter`, `Hole.Depth`, and the countersink
  dimensions are never checked. Three real defects in this project passed a "clean" audit.
- The script also does not check `TaperAngle`, `Placement`, sketch `AttachmentOffset`, datum
  attachment, or external geometry. This project relies on two `AttachmentOffset.Base.z`
  bindings that the audit cannot see — verify those by hand via the MCP bridge.

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
- Everything lives at the project root; there is no `cad/` subdirectory.

---

## Print profile

**No successful test print yet — profile TBD.**

`3mf/Lockbox Bracket.3mf` was sliced on 2026-09-17 from the pre-remediation model, but there is
no confirmed print result. Do not treat it as a validated profile, and re-slice after the hole
pattern change in `plan.md` lands. Fill this section from the first actual successful print —
material, layer height, walls, infill, orientation — not from slicer defaults.
