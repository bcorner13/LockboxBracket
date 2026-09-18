# Intent — Lockbox Bracket

Stated by Bradley 2026-09-17/18. Test-print validated 2026-09-18.

## Goal

A bracket that holds a lockbox **under a desk**, fixed with **M3×0.5 screws (M3×10,
ISO14582 countersunk)**.

The box sits in an open-ended U-channel; a mounting flange runs along the top of each side
wall. The flanges go up against the underside of the desk and the screws pass **up through
the flanges into the desk** — heads countersunk flush on the flange underside, ~6 mm of
thread into the desk through the 4 mm flange.

## How the box is retained — the key design decision

**The bracket is deliberately SHORTER than the box.** The box slides in from one open end,
and the **combination dial / lock knob on the box's front face bottoms out against the end of
the channel**, stopping it before it can slide all the way through. There is no catch, lip, or
stop feature in the model — the knob *is* the stop.

This is why `VarSet.Depth` is **185 mm** rather than matching the box's length. That number
came off the physical box (2026-09-18) and is not arbitrary:

> **Do not "tidy" `Depth` to match the box length, and do not add a back wall.** Either would
> defeat the retention mechanism. If the bracket were as long as the box, the knob would clear
> the end and the box would slide straight through.

The far end stays open so the box can be pushed back out.

## Requirements

1. **Hole placement symmetric** along the flange. ✅ done — one formula, equal end margins.
2. **`NumHoles` is a parameter**; spacing, both flanges and the screws all follow from it.
   ✅ done. Currently 4 per flange, **8 total** (12 was judged overkill).
3. **Both flanges drilled.** ✅ done via a sketch symmetry constraint.
4. **Screws seat in the holes**, heads in the countersinks, threads up into the desk. ✅ done.

## Form (measured)

- Open-ended **U-channel**: 2 mm floor, two 2 mm side walls, open at both ends.
- Interior cavity **165.5 × 185 × 46 mm**; overall envelope **225.5 × 185 × 52 mm**.
- **Flanges** 30 mm wide × 4 mm thick, full length, at the top of each wall.
- **Countersunk M3 clearance holes**: 3.4 mm bore, 6.7 mm × 90° countersink, opening downward.

## Validated by test print (2026-09-18)

A full-width coupon (225.5 × 32.4 × 52 mm, 16.6% of the part, 48m49s, 29.4 g PETG) was printed
in the production orientation and checked against the real box:

- **Fit is snug without being over-tight** — the interior width of 165.5 mm is correct.
- **The corner radius fits well** — estimated value confirmed, no change needed.
- **2 mm walls and floor are adequate.** Bradley's call: *"plenty strong as is."* The earlier
  open question about thickening for the load case is **closed** — no change.

## Constraints

- Must follow `CAD_STANDARDS.md` and the parametric rules in `CLAUDE.md`.
- All dimensions in mm; single watertight manifold solid.
- Printable **without supports** — printed on end, depth axis vertical.
- Fits the Creality K2 Plus bed (350 × 350 mm).
- **Material: PETG** (CR-PETG, CFS slot 4), 4 perimeters. Chosen over PLA/PLA-CF because the
  bracket carries a load continuously and PLA creeps under sustained stress.
- Target price point $36–$45 per `CAD_STANDARDS.md`. Material cost is a few dollars; the full
  part is ~139 cm³.

## Non-goals

- **No lid, latch, catch or back stop** — the box's own lock knob is the stop (see above).
- No print-in-place moving parts.
- The desk itself is not modelled.
