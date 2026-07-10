# Dash Cluster Surround — Type-65 Daytona Coupe Gen 3

A 3D-printed, fully-parametric **replacement for the stock flat aluminum gauge
binnacle** on the FFR Type-65 Daytona Coupe. The redesign reclaims
**steering-wheel clearance** — the stock surround protrudes far enough that the
wheel rim clearance is tight — by making the front face parametrically
**curvable to match the steering-wheel arc** and/or **set back to reduce
protrusion**, both exposed as VarSet knobs.

It keeps the stock functions: framing the full cluster (2 large gauges +
4 minor), hooding the gauges from **sun glare**, and carrying the **3 dash
indicators** (LEFT / HIGH BEAM / RIGHT) along the top radius per
[`../InstrumentCluster.md`](../InstrumentCluster.md).

## Status

Fresh project — scaffolded, **no geometry yet**. Next steps:
1. 3D-scan the stock aluminum surround → `cad_files/` (reference underlay only).
2. Measure from the scan + calipers to fill the `[measure]` placeholders in
   [`plan.md`](plan.md).
3. Approve `plan.md`, then build `Params.FCStd` + `Dash Cluster Surround.FCStd`.

## Key parameters

Full table in [`plan.md`](plan.md). The headline clearance knobs:

- `FaceArcRadius` — huge = flat; ≈ wheel-rim radius = concave, hugs the wheel.
- `ProtrusionSetback` — uniform rearward shift to gain clearance.
- `HoodProjection` / `HoodRake` — glare-hood depth and rake.
- `WheelClearanceGap` — target min gap to the wheel rim (design reference).

## Layout

```
Dash Cluster Surround/
├── CLAUDE.md            project rules + topology (read first)
├── intent.md            the goal
├── plan.md              parameters, feature tree, constraint strategy, validation
├── CAD_STANDARDS.md     workspace standard (copied, read-only)
├── MCP_TOOLS_REFERENCE.md / tools.md   FreeCAD MCP reference (copied, read-only)
├── Params.FCStd         VarSet — all parametric variables (to be created)
├── Dash Cluster Surround.FCStd   the surround Body (to be created)
├── .claude/settings.json  FreeCAD-MCP guard hook (committed)
├── 3mf/ stl/            print deliverables (tracked)
├── gcode/               slicer output (gitignored)
├── images/              renders / screenshots
├── macros/              project .FCMacro files
├── scripts/             audit_parametric.py
└── cad_files/           imported reference geometry (the 3D scan)
```

## Constraints

- Millimeters, watertight/manifold, centered at origin (CAD_STANDARDS §1).
- Single piece ≤ 330 mm wide → one print on the Creality K2 Plus.
- PLA for test-fit first; ASA or PETG for the production in-cabin part.
- Everything parametric via `<<Params>>#VarSet.VarName`; sketches on datum
  planes; clearances decoupled. See [`CLAUDE.md`](CLAUDE.md).

## Verify

```bash
python3 scripts/audit_parametric.py   # must be clean before any commit
```

Part of the [`Daytona Coupe`](../) build. Commercial License workspace.
