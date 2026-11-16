---
name: mechanical-cad
description: Design mechanical parts (brackets, housings, shafts, plates, flanges) as parametric CAD code using CadQuery (Python), then export to STEP/STL for CAD tools or 3D printing. Use when the user asks to model, draft, or generate a mechanical part, bracket, enclosure, or any CAD geometry.
---

# Mechanical CAD with CadQuery

Build mechanical parts as Python scripts with CadQuery. Keep dimensions as named
parameters at the top of the script so the design is easy to change.

## Setup

```bash
pip install cadquery
```

Requires Python 3.9+. Some environments cannot install packages; if the install is
blocked, say so and do not work around it.

## Workflow

1. Clarify the part only if a key dimension is missing. Otherwise pick sensible
   defaults and list them in the final message.
2. Write the script in a project folder, for example `cad/bracket.py`.
3. Run it and export:
   - `.step` for editing in SolidWorks, Fusion, FreeCAD, or Onshape.
   - `.stl` for 3D printing or mesh previews.
4. Check the result: bounding box, volume, and that the solid is valid.

## Example: plate with bolt holes and a fillet

```python
import cadquery as cq

# Parameters (mm)
L, W, T = 60.0, 40.0, 6.0      # plate length, width, thickness
HOLE_D = 5.5                    # clearance hole for M5
HOLE_INSET = 8.0                # distance from the plate edge to hole centers
FILLET_R = 3.0                  # corner radius on vertical edges

part = (
    cq.Workplane("XY")
    .box(L, W, T)
    .edges("|Z").fillet(FILLET_R)
    .faces(">Z").workplane()
    .pushPoints([
        (-L / 2 + HOLE_INSET, -W / 2 + HOLE_INSET),
        ( L / 2 - HOLE_INSET, -W / 2 + HOLE_INSET),
        (-L / 2 + HOLE_INSET,  W / 2 - HOLE_INSET),
        ( L / 2 - HOLE_INSET,  W / 2 - HOLE_INSET),
    ])
    .hole(HOLE_D)
)

cq.exporters.export(part, "plate.step")
cq.exporters.export(part, "plate.stl")

bb = part.val().BoundingBox()
print(f"bbox: {bb.xlen:.2f} x {bb.ylen:.2f} x {bb.zlen:.2f} mm")
print(f"volume: {part.val().Volume():.1f} mm^3")
print(f"valid: {part.val().isValid()}")
```

## Common operations

| Goal | CadQuery |
|------|----------|
| Block | `cq.Workplane("XY").box(x, y, z)` |
| Cylinder | `cq.Workplane("XY").cylinder(h, r)` |
| Extrude a 2D profile | `.rect(w, h).extrude(d)` or `.circle(r).extrude(d)` |
| Hole on top face | `.faces(">Z").workplane().hole(d)` |
| Hole pattern | `.pushPoints([...]).hole(d)` |
| Pocket | `.faces(">Z").workplane().rect(w, h).cutBlind(-depth)` |
| Fillet / chamfer | `.edges("|Z").fillet(r)`, `.edges(">Z").chamfer(c)` |
| Hollow housing | `.faces(">Z").shell(-wall)` |
| Boolean | `a.union(b)`, `a.cut(b)`, `a.intersect(b)` |
| Translate | `.translate((x, y, z))` |

## Design rules

- Use parameters, not magic numbers.
- Put clearance holes about 0.5 mm larger than the bolt for M3 to M8.
- Avoid sharp internal corners in pockets; add a fillet of at least the tool radius.
- Keep wall thickness at least 2 mm for 3D printed parts, and check against the
  printer's minimum feature size.
- Check tolerances for fits: for a sliding fit, add about 0.1 to 0.2 mm to the
  nominal diameter.

## Verification

- `isValid()` must be `True`.
- The bounding box must match the requested size.
- For a printed part, the STL should open without errors in a slicer.
- Report the parameters and the bounding box to the user.

## Status

The example above was written without being executed in the environment where
this skill was created, because installing CadQuery was denied. Run it once
before relying on it, and fix any API differences for the installed version.
