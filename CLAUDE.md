# Project rules — Dash Cluster Surround

Everything in this project must be parametric. The single highest-risk area is
the **front-face arc**: the whole reason this part exists is to reclaim
steering-wheel clearance by dialing `FaceArcRadius` (flat↔concave) and
`ProtrusionSetback`. If those become baked-in geometry instead of live VarSet
knobs, the part loses its entire purpose and a re-print can't be dialed in.
Read the full rules below before making any geometry change.

> **How to use this template:** Every `[FILL: …]` marker is a required edit. Leave none. A thin CLAUDE.md (one that just restates the global rules) is the failure mode this template exists to prevent — the point is to capture *this project's* specific topology, history, and current state, so the user does not have to re-explain it each session.

---

## Hard rules (this project)

These restate the global rules in `~/.claude/CLAUDE.md` with project-specific context. Cite the actual incident or constraint that motivates each one — generic restatements are useless because the global file already has them.

1. **Everything parametric.** The clearance knobs (`FaceArcRadius`,
   `ProtrusionSetback`, `HoodProjection`, `WheelClearanceGap`) and every gauge/
   indicator/fastener position (`GnX/GnZ`, `Indicator*`, `Fastener*`) must be
   VarSet variables bound via `<<Params>>#VarSet.VarName`. These are exactly the
   values that get retuned after a test print — if any is a literal, the reprint
   loop breaks. No prior incident in *this* project (it is brand new), but the
   Spade Connector sibling lost 3 days to a coordinate-edit; that is the standard
   we are avoiding.

2. **No fixing geometry by editing raw sketch coordinates.** No prior incident in
   this project; rule applies preventively. When a gauge bore or fastener hole is
   mislocated, fix the `GnX/GnZ` / `Fastener*` VarSet value or the bound
   constraint — never edit `Circle CenterX/CenterY` or `LineSegment *X/*Y`.

3. **Attach sketches to datum planes, not feature faces.** No prior DAG incident;
   rule applies preventively. This part has a genuinely layered feature tree
   (face → hood → flanges → many pockets), so face-attached sketches would be
   especially prone to "graph must be a DAG" breakage. Every sketch attaches to a
   `PartDesign::Plane` datum (`DP_Face`, `DP_Hood`, `DP_FlangeL/R`) — see
   `plan.md` feature tree. If you see an `AttachmentSupport` pointing at a
   `Pad_*/Pocket_*/Body_*` face, retarget it to a datum.

4. **Clearance concepts stay decoupled.** This project uses one print-fit
   clearance Param per mating interface — never reuse one across interfaces:
   - `GaugeClearance` — gauge cutout ↔ gauge body (printed bore vs gauge can)
   - `IndicatorClearance` — indicator bore ↔ 12–12.7 mm chrome LED bezel
   - `RivetHoleClearance` — fastener hole ↔ rivet/bolt shank
   - `WheelClearanceGap` — a *design-target* gap (surround ↔ steering-wheel rim),
     used in arc/setback expressions; not a print-fit clearance, kept separate.
   PLA test prints (~0.2% shrink) will want tighter clearances than the eventual
   ASA/PETG production part (~0.5–0.7%); retune per material, don't reuse values.

---

## Assembly architecture

This is a single printed part that mounts to the *existing* car, so "assembly"
here means how the surround relates to the surrounding hardware (this cannot be
recovered from the FCStd):

- The **front face** (tilted back by `FaceTilt`, optionally arced by
  `FaceArcRadius`) carries **6 gauge bores** (2 large speedo+tach + 4 minor) and
  the **3 indicator bores** (LEFT green / HIGH BEAM blue / RIGHT green) along the
  top radius, centered between speedo and tach. Gauges and indicators are
  retained by their *own* hardware/bezels; the surround bores are clearance-only.
- The **glare hood** projects forward (+Y, toward the driver) from the top of the
  face, raked down by `HoodRake`, shading the gauge faces.
- Two **side return flanges** extend rearward (−Y) and rivet/bolt to the dash
  frame — this is the only structural attachment.
- The **bottom center notch** clears the steering column.
- **+Y is toward the driver** and is the direction that consumes steering-wheel
  clearance — the redesign reduces intrusion there via the arc + setback.
- Replaces the **stock flat aluminum surround** (photographed in the intake
  conversation); gauges and cluster wiring are untouched. See `../InstrumentCluster.md`
  for the indicator wiring intent (parallels the exterior turn-signal outputs via
  the D1 Mini "Smart Blinker").

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `Params.FCStd` | VarSet — all parametric variables | — | ⏳ not yet created (bootstrap macro pending first FreeCAD open) |
| `Dash Cluster Surround.FCStd` | the surround Body (`Body_Surround`) | `Params.FCStd` | ⏳ not yet created — awaiting scan + approval to model |

No broken files — the project is fresh; no geometry exists yet. Modeling has not
started (plan.md must be approved and the stock part scanned first).

---

## Params variables (summary)

Defined in `plan.md` (PARAMETERS section) and instantiated into `Params.FCStd`.
Groups: **Face** (`PanelWidth/Height`, `FaceThick`, `FaceTilt`, `CornerFillet`),
**Clearance-redesign** (`FaceArcRadius`, `ProtrusionSetback`, `WheelClearanceGap`),
**Hood** (`HoodProjection/Rake/Thick`, `HoodSlot*`), **Gauge bores**
(`LargeGaugeDia`, `MinorGaugeDia`, `GaugeClearance`, `GaugeBoreChamfer`, `GnX/GnZ`),
**Indicators** (`IndicatorHoleDia/Clearance/Pitch/CenterX/RowZ`), **Mounting**
(`FlangeDepth/Thick`, `FastenerHoleDia`, `RivetHoleClearance`, `FastenerCount/Pitch/Inset`),
**Column notch** (`ColumnNotchWidth/Height`). All defaults are placeholders
flagged **[measure]** until the 3D scan + caliper pass replaces them.

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
- Features with literal Length/Offset/Radius values

The script is authoritative. If it reports violations, fix them via a
`macros/*.FCMacro` change before saving or committing — never by direct
coordinate edits or FCStd XML surgery.

**Note on this copy of the audit script:** its `DIMENSIONAL_TYPES` enum was
corrected to the true FreeCAD Sketcher `Constraint::Type` values
(Distance=6, DistanceX=7, DistanceY=8, Angle=9, Radius=11, Diameter=18,
Weight=19). The canonical Spade Connector copy shipped a shifted/wrong mapping
that produced false positives — do not "restore" it from Spade. See memory
`reference_audit_parametric_type_bug.md`.

No exemptions; audit must be clean.

---

## Memory files (deeper context)

`~/.claude/projects/-Users-bradleycorner/memory/MEMORY.md` indexes the persistent
memories. Most relevant to this project:

- `reference_audit_parametric_type_bug.md` — why this project's audit script
  enum differs from Spade's (bug fix).
- `feedback_canonical_reference_only.md` — scaffolding uses only the Spade
  reference structure.
- `project_print_project_dir_convention.md` — the folder layout this project follows.
- Global `~/.claude/CLAUDE.md` — the parametric hard rules and the Spade
  coordinate-edit incident that motivates them.

No project-scoped memory file exists for the Dash Cluster Surround yet — this is
a fresh project. Add one if a modeling incident or non-obvious decision arises.

---

## Workflow notes

**Invariant (apply to every FreeCAD project — do not edit):**

- **Inspect/edit FreeCAD models via the MCP bridge — never with shell tools.** Do **not** `unzip`/`grep`/`cat`/`sed`/`strings`/etc. a `.FCStd`. Use the FreeCAD Robust MCP server: `get_connection_status` first, then `open_document`, `list_objects`, `inspect_object`, `execute_python`, and macros. This is **enforced** by a PreToolUse hook in `.claude/settings.json` (copied from the root `settings.template.json` during bootstrap) — raw shell access to `.FCStd` is blocked. Only fall back to read-only `unzip` if the MCP bridge is genuinely unreachable, and ask the user first.
- **MCP server auto-starts with FreeCAD.** If an `mcp__freecad__*` call fails, the right interpretation is "FreeCAD isn't running" — ask whether to launch it. Do **not** silently fall back to `unzip` + XML parsing, and do **not** retry the same MCP call.
- **Write changes as `macros/*.FCMacro` files**, not direct XML edits. Reasons: reviewable, re-runnable, idempotent-friendly, uses FreeCAD's own serialization.
- **Cross-document expressions**: use the canonical form `<<Params>>#VarSet.VarName`. The shorter `<<Params>>.VarName` form sometimes fails with "Params not found."
- **Run `python3 scripts/audit_parametric.py` before committing.** If it reports violations, fix via macro, not by editing FCStd XML.

**Project-specific:**

- The stock part is captured by **3D scan → `cad_files/`**, used as a *reference
  underlay only* (align to origin, measure from it to populate the VarSet).
  **Never trace/convert the scan mesh into model geometry** — it is noisy and
  non-manifold and would violate the parametric-only rule. Confirm fit-critical
  diameters (gauge cutouts, 12–12.7 mm indicator bores, rivet spacing) with
  calipers, not the scan alone.
- **Flat↔curved is one knob.** Keep `FaceArcRadius` as the single control; do not
  fork the model into "flat" and "curved" variants.
- Single piece, keep bounding-box X ≤ 330 mm (Creality K2 Plus bed).
- Source FCStds live at the project root (`Dash Cluster Surround.FCStd`,
  `Params.FCStd`); per-variant files would be `Dash Cluster Surround-V1.FCStd` etc.

---

## Print profile

**No successful test print yet — profile TBD.** (First article will be PLA for
fit-check; production later in ASA or PETG. Fill this table from the actual
first successful print, not slicer defaults.)
