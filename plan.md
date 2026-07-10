# Plan — Dash Cluster Surround

> AI-generated modeling plan per `PROJECT_BOOTSTRAP.md`. **Execution of CAD is
> forbidden until this plan is approved.** All defaults below are placeholders
> flagged **[measure]** — they get replaced with values derived from the 3D scan
> + caliper checks of the stock aluminum surround before/as geometry is built.
> Nothing here is a load-bearing literal; every value becomes a VarSet variable
> bound via `<<Params>>#VarSet.VarName`.

## Coordinate convention

- Origin at the **center of the gauge cluster front face**, model centered at
  (0,0,0) per CAD_STANDARDS §1.
- **X** = across the dash (driver's left→right, +X to the right).
- **Z** = up.
- **Y** = fore/aft, **+Y toward the driver** (the direction the hood projects and
  the direction that eats steering-wheel clearance).
- Front gauge face lies roughly in the XZ plane, tilted back by `FaceTilt` about X.

## PARAMETERS (VarSet in `Params.FCStd`)

Types: `Length` = App::PropertyLength (mm) · `Angle` = App::PropertyAngle (deg) ·
`Int` = App::PropertyInteger · `Float` = App::PropertyFloat (unitless).

### Group: Face (overall front panel)
| Name | Type | Default | Notes |
|---|---|---|---|
| `PanelWidth` | Length | 300 mm | **[measure]** overall surround width; must stay ≤330 for K2 Plus |
| `PanelHeight` | Length | 110 mm | **[measure]** front face height |
| `FaceThick` | Length | 3.0 mm | face wall thickness (CAD_STANDARDS 2–3 mm) |
| `FaceTilt` | Angle | 12 deg | **[measure]** rearward tilt of face from vertical |
| `CornerFillet` | Length | 6 mm | outer corner rounding (silk-friendly) |

### Group: Clearance-redesign knobs (the whole point)
| Name | Type | Default | Notes |
|---|---|---|---|
| `FaceArcRadius` | Length | 4000 mm | **[measure]** concave arc radius of front face. Huge = effectively flat; set ≈ steering-wheel rim radius + `WheelClearanceGap` to hug the wheel |
| `ProtrusionSetback` | Length | 0 mm | uniform rearward (−Y) shift of the whole surround to gain clearance |
| `WheelClearanceGap` | Length | 8 mm | **[measure]** target minimum gap from surround to wheel rim at the tightest point; design reference used in arc/setback expressions |

### Group: Glare hood (top shelf)
| Name | Type | Default | Notes |
|---|---|---|---|
| `HoodProjection` | Length | 40 mm | **[measure]** forward (+Y) depth of the hood shelf |
| `HoodRake` | Angle | 10 deg | downward/rearward rake of the hood to shade + clear wheel |
| `HoodThick` | Length | 3.0 mm | hood wall thickness |
| `HoodSlotWidth` | Length | 0 mm | optional styling/defog slot width (0 = no slot; matches stock if measured) |
| `HoodSlotLength` | Length | 60 mm | slot length when enabled |

### Group: Gauge bores (clearance-only; gauges retained by own hardware)
| Name | Type | Default | Notes |
|---|---|---|---|
| `LargeGaugeDia` | Length | 100 mm | **[measure]** speedo/tach cutout diameter (before clearance) |
| `MinorGaugeDia` | Length | 52 mm | **[measure]** 2-1/16″ minor gauge cutout diameter |
| `GaugeClearance` | Length | 0.3 mm | print-fit clearance added to each gauge bore radius (PLA; retune for ASA) |
| `GaugeBoreChamfer` | Length | 1.0 mm | front chamfer around each bore (deburr look, no overhang) |
| Per-gauge centers `G1X…G6X`, `G1Z…G6Z` | Length | **[measure]** | one X/Z pair per gauge; positions read from scan. G1/G2 = large (speedo/tach), G3–G6 = minor |

### Group: Indicators (per ../InstrumentCluster.md)
| Name | Type | Default | Notes |
|---|---|---|---|
| `IndicatorHoleDia` | Length | 12.7 mm | **[measure/caliper]** chrome LED bezel bore (12–12.7 mm) |
| `IndicatorClearance` | Length | 0.3 mm | print-fit clearance for bezel bore (decoupled from gauges) |
| `IndicatorPitch` | Length | 30 mm | **[measure]** center-to-center of the 3 indicators |
| `IndicatorCenterX` | Length | 0 mm | X of the middle (HIGH BEAM) hole; centered between speedo & tach |
| `IndicatorRowZ` | Length | 40 mm | **[measure]** height of indicator row along the top radius |

### Group: Mounting (side return flanges → dash frame)
| Name | Type | Default | Notes |
|---|---|---|---|
| `FlangeDepth` | Length | 50 mm | **[measure]** how far the side flanges return rearward (−Y) |
| `FlangeThick` | Length | 3.0 mm | flange wall thickness |
| `FastenerHoleDia` | Length | 4.2 mm | **[measure]** rivet/bolt shank diameter |
| `RivetHoleClearance` | Length | 0.3 mm | print-fit clearance for fastener holes (decoupled) |
| `FastenerCount` | Int | 4 | **[measure]** holes per side flange |
| `FastenerPitch` | Length | 30 mm | **[measure]** spacing of fastener holes along flange |
| `FastenerInset` | Length | 8 mm | inset of holes from flange edge |

### Group: Steering-column notch (bottom center)
| Name | Type | Default | Notes |
|---|---|---|---|
| `ColumnNotchWidth` | Length | 60 mm | **[measure]** bottom center cutout width for the column |
| `ColumnNotchHeight` | Length | 30 mm | **[measure]** cutout height |

## FEATURE TREE (ordered)

All in one PartDesign Body `Body_Surround`. Every sketch attaches to a
`PartDesign::Plane` datum (never a feature face). Cross-doc expressions use
`<<Params>>#VarSet.VarName`.

1. **Datums** — create datum planes offset from origin planes:
   `DP_Face` (XZ, tilted `FaceTilt`), `DP_Hood` (offset up + raked `HoodRake`),
   `DP_FlangeL` / `DP_FlangeR` (YZ, offset ±`PanelWidth`/2). No face attachment.
2. **Sk_FaceProfile** on `DP_Face` — the front-panel outline (rounded rect,
   `PanelWidth`×`PanelHeight`, `CornerFillet`). If `FaceArcRadius` is finite, the
   face is realized as a **cylindrical arc** either by (a) padding the profile
   then wrapping via a large-radius revolve/loft, or (b) sketching the outline on
   a curved datum. Chosen sub-method finalized at build time from the scan; the
   arc radius stays a single VarSet knob so flat↔curved is one number.
3. **Pad_Face** — pad `Sk_FaceProfile` by `FaceThick`. Apply `ProtrusionSetback`
   via the datum offset expression.
4. **Sk_Hood** + **Pad_Hood** (or additive loft `Face→Hood`) — the forward glare
   shelf, `HoodProjection` deep, `HoodThick` thick, raked `HoodRake`.
5. **Sk_FlangeL/R** + pads — side return flanges, `FlangeDepth`×`FlangeThick`.
6. **Sk_GaugeBores** on `DP_Face` — 6 circles at `(GnX,GnZ)`; large = `LargeGaugeDia`,
   minor = `MinorGaugeDia`, each +`GaugeClearance`. **Pocket ThroughAll**.
7. **GaugeBoreChamfer** — chamfer front edges of the 6 bores.
8. **Sk_Indicators** on `DP_Face` — 3 circles at `IndicatorCenterX ± IndicatorPitch`,
   `IndicatorRowZ`, `IndicatorHoleDia`+`IndicatorClearance`. **Pocket ThroughAll**.
9. **Sk_ColumnNotch** — bottom center cutout, `ColumnNotchWidth`×`ColumnNotchHeight`,
   **Pocket ThroughAll**.
10. **Sk_Fasteners** on `DP_FlangeL/R` — `FastenerCount` holes per side at
    `FastenerPitch`, dia `FastenerHoleDia`+`RivetHoleClearance`. **Pocket ThroughAll**.
11. **Edge fillets** — cosmetic outer-edge rounding as needed (CAD_STANDARDS,
    silk finish). Radius via a VarSet var, no literals.

## CONSTRAINT STRATEGY

- Every dimensional constraint (Distance/DistanceX/DistanceY/Radius/Diameter/
  Angle) is created, then immediately `setExpression`-bound to its VarSet var.
  No literal numbers left in constraints or feature Length/Offset/Radius props.
- Each sketch is driven fully-constrained (green) with all driving dims bound.
- Gauge/indicator/fastener centers are located by `DistanceX`/`DistanceY` from the
  sketch origin, bound to the `GnX/GnZ`, `Indicator*`, `Fastener*` vars.
- Symmetry: use symmetric constraints about the face centerline where the layout
  is mirror-symmetric, to halve the number of position vars where the scan
  confirms symmetry.
- The flat↔curved behavior is a single expression path on `FaceArcRadius`; the
  build never hard-codes "flat" vs "curved" geometry.

## VALIDATION

1. `python3 scripts/audit_parametric.py` → must be **clean** (0 issues) before
   any save/commit. Flags unconstrained sketches, unbound dimensions, DAG-risk
   face attachments.
2. Visual check via MCP screenshots (Front / Right / Isometric) after recompute.
3. `part_check_shape` / watertight check before STL export (manifold per
   CAD_STANDARDS §1).
4. Bounding box ≤330 mm in X (fits K2 Plus) — assert via `part_get_bounding_box`.
5. Steering-wheel clearance: after scan alignment, measure min distance from
   surround to wheel-rim reference; confirm ≥ `WheelClearanceGap`.
6. PLA test print → fit check on the car → tune `FaceArcRadius`,
   `ProtrusionSetback`, `HoodProjection`, and clearance vars, then reprint.

## OPEN ITEMS (resolved from scan + caliper before/at build)

- Final gauge arrangement (which bore is speedo vs tach vs which 4 minors) and
  exact `GnX/GnZ` positions.
- Whether the stock hood slot is functional (defog) or styling → sets `HoodSlot*`.
- Rivet vs bolt (sets whether `FastenerHoleDia` is a rivet shank or a bolt
  clearance) and the true `FastenerCount`/`FastenerPitch`.
- Real steering-wheel rim radius to seed `FaceArcRadius`.
