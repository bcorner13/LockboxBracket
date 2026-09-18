# Lockbox Bracket

A parametric FreeCAD bracket that cradles a lockbox: an open-ended U-channel the box slides
into lengthwise, with an outward mounting flange along the top of each side wall drilled for
countersunk M3 fasteners.

| | |
|---|---|
| **Envelope** | 225.5 × 195 × 52 mm |
| **Interior cavity** | 165.5 × 195 × 46 mm |
| **Wall / floor** | 2.0 mm |
| **Flanges** | 30 mm wide × 4 mm thick, full depth, both sides |
| **Fasteners** | 6 × M3 countersunk per flange (3.4 mm bore, 6.7 mm × 90° c'sink) |
| **Volume** | 147 019.56 mm³ |
| **CAD** | FreeCAD 1.1.3, PartDesign, single body |

## Status

Model is geometrically complete and valid — one watertight solid, all sketches fully
constrained and attached to origin planes. **Parametric remediation is pending:** five
hard-coded feature dimensions and an inconsistent hole-spacing expression are documented in
[`plan.md`](plan.md) and await approval before execution.

No successful test print yet.

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
