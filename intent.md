# Intent — Lockbox Bracket

> **STATUS: DRAFT — reverse-engineered from the existing model, needs Bradley's confirmation.**
> `intent.md` is human input per `PROJECT_BOOTSTRAP.md`. The model existed before this file, so
> the goal below was inferred by measuring `Lockbox Bracket.FCStd`, not stated by the user.
> Correct anything wrong here before it gets treated as the spec. The lines marked
> **[CONFIRM]** are the ones I could not derive from geometry.

## Goal

A parametric mounting bracket that cradles a lockbox: an open-ended U-channel that the box
sits down into, with an outward mounting flange along the top of each side wall, drilled for
countersunk M3 fasteners.

**[CONFIRM]** What the bracket actually mounts *to* (wall, vehicle panel, safe interior, shelf
underside) and what lockbox model it is sized around. The 169.5 × 195 mm interior footprint
looks like it was taken off a specific box.

## Form (measured from the current model)

- Open-ended **U-channel / trough**: 2 mm floor, two 2 mm side walls, **open at both ends**
  along the depth axis, so the box slides in lengthwise.
- **Outward flanges** at the top of each wall, 30 mm wide × 4 mm thick, running the full depth.
- **6 countersunk M3 clearance holes per flange** (3.4 mm bore, 6.7 mm × 90° countersink),
  evenly spaced along the depth.
- Overall envelope **225.5 × 195 × 52 mm**. Interior cavity **165.5 × 195 × 46 mm**.

## Constraints

- Must follow `CAD_STANDARDS.md` and the parametric rules in `CLAUDE.md`.
- All dimensions in mm; single watertight manifold solid.
- Printable **without supports** — the U-channel prints open-side-up, flanges sit flat on the
  bed at full width. Countersinks face upward, so they self-support.
- Fits the **Creality K2 Plus** bed (350 × 350 mm). At 225.5 × 195 mm the part fits flat with
  room to spare. **[CONFIRM]** whether ELEGOO Saturn 4 (resin) is also a target — it is not,
  at this size.
- **[CONFIRM] Material.** PLA for test fit, ASA for production is the house default; a load-
  bearing bracket holding a metal lockbox argues for ASA or PETG in production.
- **[CONFIRM] Load case.** Does the bracket carry the box's full weight in shear on the six
  fasteners per side, or does the box rest on something else? This decides whether 2 mm walls
  and a 2 mm floor are adequate or need thickening.
- Target price point $36–$45 per `CAD_STANDARDS.md`. **[CONFIRM]** — at ~147 cm³ of solid
  volume this is a large print; verify the price band still applies to a functional bracket
  rather than a decorative piece.

## Non-goals

- No lid, latch, or locking mechanism — this is the cradle only.
- No print-in-place moving parts.
