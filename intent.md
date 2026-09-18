# Intent — Lockbox Bracket

Stated by Bradley 2026-09-17. Supersedes the reverse-engineered draft.

## Goal

A bracket that holds a lockbox **under a desk**, fixed with **M3×0.5 screws (M3×10,
ISO14582 countersunk)**.

The box sits in an open-ended U-channel; a mounting flange runs along the top of each side
wall. The flanges go up against the underside of the desk and the screws pass **up through
the flanges into the desk** — heads countersunk flush on the flange underside, ~6 mm of
thread into the desk through the 4 mm flange.

## Requirements

1. **Hole placement must be symmetric** along the depth of the flange. The current model's
   spacing math is wrong — three different divisors describe one concept, leaving uneven
   end margins (32.5 mm vs 23.21 mm).
2. **The number of holes is a parameter** (`NumHoles`), and everything downstream — spacing,
   the mirrored flange, and the screws — follows from it.
3. **Both flanges are drilled.** The holes currently exist on the +X flange only; they must
   be mirrored onto the −X flange.
4. **The screws seat in the holes** — heads in the countersinks, threads pointing up into
   the desk. Two per hole position, i.e. `2 × NumHoles` screws.

## Form (measured)

- Open-ended **U-channel**: 2 mm floor, two 2 mm side walls, open at both ends along the
  depth axis, so the box slides in lengthwise.
- Interior cavity **165.5 × 195 × 46 mm**; overall envelope **225.5 × 195 × 52 mm**.
- **Flanges** 30 mm wide × 4 mm thick, full depth, at the top of each wall.
- **Countersunk M3 clearance holes**: 3.4 mm bore, 6.7 mm × 90° countersink opening downward.

## Constraints

- Must follow `CAD_STANDARDS.md` and the parametric rules in `CLAUDE.md`.
- All dimensions in mm; single watertight manifold solid.
- Printable **without supports** — channel open-side-up, flanges flat on the bed.
- Fits the Creality K2 Plus bed (350 × 350 mm) flat. Not a resin part; the Saturn 4 is not a
  target at this size.
- **[CONFIRM] Material.** PLA for test fit, ASA for production is the house default. Holding a
  metal lockbox overhead argues for ASA or PETG in production — an under-desk bracket that
  fails drops the box.
- **[CONFIRM] Load case.** Whether the twelve M3 fasteners carry the box's full weight in
  shear decides if 2 mm walls and a 2 mm floor are adequate.
- **[CONFIRM] Lockbox model** the 165.5 × 195 mm interior is sized around.
- Target price point $36–$45 per `CAD_STANDARDS.md`. **[CONFIRM]** whether that band applies
  to a functional bracket at ~147 cm³.

## Non-goals

- No lid, latch, or locking mechanism — this is the cradle only.
- No print-in-place moving parts.
- The desk itself is not modelled.
