# Lockbox Bracket

A parametric FreeCAD bracket that holds a lockbox **under a desk**: an open-ended U-channel the
box slides into lengthwise, with an outward mounting flange along the top of each side wall
drilled for countersunk M3×10 fasteners that run up into the desk.

![Lockbox Bracket assembly](images/lockbox-bracket-assembly.png)

| | |
|---|---|
| **Envelope** | 225.5 × 195 × 52 mm |
| **Interior cavity** | 165.5 × 195 × 46 mm |
| **Wall / floor** | 2.0 mm |
| **Flanges** | 30 mm wide × 4 mm thick, full depth, both sides |
| **Fasteners** | `NumHoles` × M3×10 countersunk per flange, both flanges (3.4 mm bore, 6.7 mm × 90° c'sink) — 12 at the default `NumHoles = 6` |
| **Volume** | 146 686.19 mm³ |
| **CAD** | FreeCAD 1.1.3, PartDesign, single body + Assembly |

## Status

Model is complete, valid, and **passes the parametric audit**. One watertight solid, all
sketches fully constrained and attached to origin planes, every live dimension bound to the
`VarSet`.

Remediated 2026-09-17: symmetric hole spacing driven by one formula, holes mirrored onto both
flanges via a sketch symmetry constraint, and `2 × NumHoles` screws seated in the bores.
Flexing `NumHoles` 6 → 8 → 6 returns the volume to 146 686.19 mm³ exactly.

**No successful test print yet.** `stl/` and `3mf/` hold current exports of the bracket alone —
watertight, manifold, 146 683.72 mm³. The 3MF is geometry only; it carries no slicer settings.

## Layout

```
Lockbox Bracket.FCStd     the part — Body plus an in-document VarSet
intent.md                 design goal and constraints (draft — needs confirmation)
plan.md                   remediation plan; execution waits on approval
CLAUDE.md                 project rules, assembly architecture, known traps
3mf/                      slicer-ready print files
stl/                      STL exports
gcode/                    slicer output (gitignored, regenerable)
images/                   renders and screenshots
macros/                   project .FCMacro files
scripts/audit_parametric.py   parametric compliance audit
```

## Exporting print files

Run `macros/export_print_files.FCMacro`. It writes `stl/Lockbox Bracket.stl` and
`3mf/Lockbox Bracket.3mf` from the bracket **Body only**.

> **Do not export with the Assembly selected.** The community
> `3D_Printer_3mf_Workflow.FCMacro` exports the current selection, so it will mesh the twelve
> M3×10 screws into the print file. `export_print_files` pins the selection to `Body` and
> leaves it selected, so you can run the workflow macro straight afterwards for its
> slicer-settings round-trip.

## Changing the hole count

`NumHoles` drives the spacing, both flanges, and the screws. After changing it:

1. Recompute the Body — the bores re-space symmetrically on their own.
2. Re-run `macros/place_screws.FCMacro` so the screw set matches the new bore count. It reads
   bore positions from the solid, so it cannot drift from the geometry.

There is **no `Params.FCStd`** — this single-document project deliberately keeps its VarSet
in-document. See the deviation note in [`CLAUDE.md`](CLAUDE.md) before "fixing" that.

## Working on this model

Open the FCStd in FreeCAD and drive everything from the `VarSet`. Never edit sketch
coordinates directly — adjust the driving variable or add a constraint bound to one.

Before committing any change:

```bash
python3 scripts/audit_parametric.py
```

A clean audit is necessary but not sufficient — the script has documented blind spots listed
in `CLAUDE.md`.

Conforms to `PROJECT_BOOTSTRAP.md` and `CAD_STANDARDS.md` in the parent workspace.
