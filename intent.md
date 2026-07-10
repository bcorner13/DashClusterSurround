# Intent — Dash Cluster Surround

**Project:** Type-65 Daytona Coupe Gen 3 → Dash Cluster Surround
**Parent build:** `Daytona Coupe/` (see `../InstrumentCluster.md`, `../TurnSignal/`)

## Goal

Design a 3D-printed **replacement** for the stock flat aluminum gauge binnacle
(surround) on the FFR Type-65 Daytona Coupe. The redesign's primary purpose is
to **reclaim steering-wheel clearance** — the stock surround protrudes far
enough that clearance to the wheel rim is tight. The new surround does this by
being **parametrically curvable to match the arc of the steering wheel** and/or
by **reducing forward protrusion**, both exposed as VarSet knobs.

Secondary purposes, carried over from the stock part:
- Frame the full gauge cluster: 2 large gauges (speedo + tach) + 4 minor gauges.
- Hood the gauge faces from sun glare via the forward top shelf.
- Carry the 3 dash indicators (LEFT green / HIGH BEAM blue / RIGHT green) along
  the top radius, centered between speedo and tach, per `../InstrumentCluster.md`.

## What it is (and is not)

- **Is:** a replacement for the aluminum *surround/binnacle* — the front gauge
  face + forward glare hood + side mounting return flanges + bottom center notch
  clearing the steering column.
- **Is not:** a replacement for the gauges or the cluster wiring. The gauges
  stay and mount through the surround's bores. It is not a stick-on trim bezel;
  it is the structural surround itself.

## Constraints

- Must follow `CAD_STANDARDS.md` (mm, manifold/watertight, centered at origin,
  Part/PartDesign workbenches) and the global parametric rules in
  `~/.claude/CLAUDE.md` (parametric-only, datum-plane attachment, decoupled
  clearances, no raw-coordinate edits).
- **Single piece**, ≤330 mm wide → fits the Creality K2 Plus bed in one print.
- **Material:** PLA for test-fit prototypes first (~0.2% shrink); ASA or PETG for
  the production in-cabin part later (heat/UV on a sunny dash). Print-fit
  clearances are tuned per material.
- **Mounting:** riveted or bolted to the existing dash frame via the side return
  flanges. Gauge bores are clearance-only (gauges retained by their own nuts /
  clamps); indicator bores are clearance for 12–12.7 mm chrome LED bezels.
- Printable without exotic supports; silk-filament-friendly surfaces (avoid
  sharp overhangs / glare artifacts) per CAD_STANDARDS §2.

## Dimensional source

Exact dimensions are **TBD**. The stock part will be **3D-scanned** and the scan
used as a *reference underlay only* — imported into `cad_files/`, aligned to
origin, and measured from to populate the VarSet. The scan is **never** traced
or converted directly into geometry (noisy/non-manifold; would violate the
parametric-only rule). Fit-critical diameters (gauge cutouts, 12–12.7 mm
indicator bores, rivet spacing) are confirmed with calipers. Until then, plan.md
carries placeholder defaults flagged "measure to confirm."

## Success criteria

1. Test print in PLA fits the dash frame (rivet/bolt pattern aligns) and clears
   the steering wheel with margin at the tightest point.
2. All six gauges and three indicators seat correctly in their bores.
3. Glare hood shades the gauge faces without obscuring driver sightlines.
4. `python3 scripts/audit_parametric.py` reports clean (fully parametric).
5. Single-piece print on the K2 Plus, watertight/manifold export.
