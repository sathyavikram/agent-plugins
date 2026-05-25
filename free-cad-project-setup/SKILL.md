# Skill: FreeCAD Project Setup

This skill defines how to structure, write, and organize Python-based FreeCAD 3D modeling projects.

> **Standalone**: this skill runs independently. Related skills that can be used alongside it:
> - `freecad-visual-validation` — visual validation procedure for parts and assemblies
> - `freecad-threading` — threaded feature construction with 3D-print clearances

---

## Project Structure

Every FreeCAD project must have this layout:

```
project-name/
├── spec/                  # Project specification files
│   ├── mission.md         # What the project builds and why
│   ├── requirements.md    # Dimensions, parts, tolerances, constraints
│   ├── roadmap.md         # Phased plan with [ ] checkboxes
│   └── reference-images/  # Example part and assembly images for visual reference
├── params.py              # All parameters, all scaled by SCALE
├── part_01_<name>.py      # First part
├── part_02_<name>.py      # Second part (if needed)
├── ...                    # Additional parts as required
├── assembly.py            # Assembles all parts into final model
├── export_all.py          # Runs all part scripts via freecadcmd
├── README.md              # Project description and requirements
├── .gitignore             # Excludes exports/, __pycache__/, *.FCBak, .DS_Store
├── exports/               # Generated STEP and STL files (gitignored)
├── 3d-print/              # Print-ready files and slicing notes
└── media/                 # Reference images, renders, screenshots
```

---

## Step 1 — Read requirements

- **Always check for a `spec/` folder first.** If it exists, read `spec/mission.md`, `spec/requirements.md`, and `spec/roadmap.md` before writing any code.
- If no `spec/` folder exists, create it and populate `spec/mission.md` and `spec/requirements.md` from the user's description or `README.md`. Also create an empty `spec/reference-images/` folder.
- If only a `README.md` or `requirements.md` exists at the project root, read that instead.
- Extract all dimensions, part names, tolerances, and functional requirements before writing any code.

### Reference images

- **Always check `spec/reference-images/`** before building any part or assembly.
- If images exist, view them using the image tool and use them as visual reference for:
  - Overall shape, proportions, and geometry of each part
  - How parts fit together and their relative positions in the assembly
  - Surface details, cutouts, ribs, or features not fully described in text
  - Correct orientation and assembly order
- Cross-reference image details against `spec/requirements.md` — images take precedence for shape; requirements take precedence for exact dimensions.
- If an image shows a feature not mentioned in requirements, include it and note it in a comment.

---

## Step 2 — params.py

All parameters live in `params.py`. Rules:

- `SCALE = 1.0` is always the **first line**.
- Every dimension is multiplied by `SCALE` so the entire model scales uniformly.
- Include `PROJECT_DIR` and `EXPORT_DIR` paths.
- Never hard-code dimensions inside part files; always import from params.
- **Derived parameters are encouraged**: compute dependent geometry values directly in `params.py` so part files never do arithmetic on raw params. For example:

```python
SCALE = 1.0
WIDTH  = 100.0 * SCALE
HEIGHT = 200.0 * SCALE
WALL   = 3.0 * SCALE
CORNER_RADIUS = WIDTH * 0.125        # derived from WIDTH
INNER_WIDTH   = WIDTH - 2 * WALL     # derived from WIDTH + WALL
```

### Build plate constraint

The default build volume is **175 × 175 × 175 mm** when `SCALE = 1.0`. Every individual part **must fit within this envelope** at `SCALE = 1.0`.

- If `spec/requirements.md` specifies different build plate dimensions, those take precedence — update `BUILD_PLATE_X/Y/Z` accordingly.
- Add a comment in `params.py` noting the target printer/build volume when overriding the default.

### Parts that exceed the build plate

If any dimension of a part exceeds the build plate at `SCALE = 1.0`, **split it into separate named parts**, each in its own file:

1. **Identify the split plane** — choose a natural seam (flat face, symmetry axis, mid-point) that minimises print supports and keeps each piece structurally sound.
2. **Give each piece its own descriptive name**: `part_NN_<name>.py` — e.g. a body split into a top and bottom becomes `part_03_body_top.py` and `part_04_body_bottom.py`, each with its own part number.
3. **Add alignment features** — dowel holes, alignment pins, or a tongue-and-groove joint so the pieces assemble accurately after printing. Define the pin/hole diameter and tolerance in `params.py`.
4. **Each part file must independently fit** within `BUILD_PLATE_X × BUILD_PLATE_Y × BUILD_PLATE_Z`.
5. **Update `assembly.py`** to import and position all pieces, placing them as if already joined together.
6. **Document the split** in `spec/requirements.md` — note which original part was split, the split plane, and the joining method.

### Template

```python
import os

SCALE = 1.0

# Build plate limits at SCALE = 1.0 (default: 175x175x175 mm)
# Override these if spec/requirements.md specifies a different build volume.
BUILD_PLATE_X = 175.0 * SCALE
BUILD_PLATE_Y = 175.0 * SCALE
BUILD_PLATE_Z = 175.0 * SCALE

# Primary dimensions (all scaled)
WIDTH  = 100.0 * SCALE
HEIGHT = 200.0 * SCALE
DEPTH  =  50.0 * SCALE

WALL_THICKNESS = 3.0 * SCALE
TOLERANCE      = 0.4 * SCALE

PROJECT_DIR = os.path.dirname(os.path.abspath(__file__))
EXPORT_DIR  = os.path.join(PROJECT_DIR, "exports")
```

---

## Step 3 — Part files (`part_NN_name.py`)

### Naming convention

Files must be named `part_NN_name.py` where `NN` is a zero-padded number:
- `part_01_body.py`
- `part_02_lid.py`
- `part_03_hinge_pin.py`

### Standard boilerplate

Every part file must follow this exact structure:

```python
import FreeCAD as App
import Part
import Import
import os
import sys

try:
    sys.path.append(os.path.dirname(os.path.abspath(__file__)))
except NameError:
    pass

import params
import importlib
importlib.reload(params)

CURRENT_DIR  = os.path.dirname(os.path.abspath(__file__))
EXPORT_BASE  = os.path.join(CURRENT_DIR, "exports")
EXPORT_STEP  = os.path.join(EXPORT_BASE, "part_NN_name.step")
EXPORT_STL   = os.path.join(EXPORT_BASE, "part_NN_name.stl")


def construct_<part_name>():
    """Build and return the Part shape, and export STEP + STL."""
    # --- geometry construction ---
    shape = ...  # build using Part primitives and boolean ops

    # --- export ---
    os.makedirs(EXPORT_BASE, exist_ok=True)

    # Delete existing files before writing
    for path in (EXPORT_STEP, EXPORT_STL):
        if os.path.exists(path):
            os.remove(path)

    shape.exportStep(EXPORT_STEP)
    shape.exportStl(EXPORT_STL)
    print(f"Exported to {EXPORT_STEP} and {EXPORT_STL}")

    return shape


def main():
    doc = App.newDocument("<PartName>")
    shape = construct_<part_name>()
    feature = doc.addObject("Part::Feature", "<PartName>")
    feature.Shape = shape


if __name__ == "__main__":
    main()
```

### Key rules for part files

- **Delete before export**: Always delete the existing STEP and STL files before writing new ones.
- **Return the shape**: `construct_*()` must return the `Part.Shape` so `assembly.py` can import it.
- **Export inside construct**: Both STEP and STL are exported inside `construct_*()`, not in `main()`.
- **Reload params**: Always call `importlib.reload(params)` after importing params.
- **sys.path guard**: Wrap `sys.path.append(...)` in `try/except NameError` to handle FreeCAD's `__file__` quirk.
- **`removeSplitter()` after complex booleans**: Call `.removeSplitter()` after fuse/cut chains with 3+ operands to clean up internal faces and prevent STEP export errors:
  ```python
  shape = a.fuse(b).fuse(c).removeSplitter()
  shape = body.cut(slot).cut(hole).removeSplitter()
  ```
- **Wrap `makeFillet()` in try/except**: Edge-based fillet operations can fail on complex geometry. Always guard them:
  ```python
  try:
      shape = shape.makeFillet(radius, edges_to_fillet)
  except Exception:
      pass  # proceed without fillet if it fails
  ```
- **Export in print orientation**: If a part's natural construction axis differs from its best print orientation, rotate it before exporting so the STL is print-ready. Document the rotation in a comment:
  ```python
  # Rotate 90° around Y so shaft lies flat on build plate
  shape.rotate(App.Vector(0, 0, 0), App.Vector(0, 1, 0), 90)
  shape.translate(App.Vector(offset_x, 0, radius))  # re-centre on plate
  ```

---

## Step 4 — assembly.py

The assembly file:
1. Clears the entire `exports/` directory.
2. Runs every part file (or imports `construct_*` functions) to regenerate STEP files.
3. Loads each part's STEP file into a `Part.Shape`.
4. Positions / transforms parts as needed.
5. Adds each part as a named `App::Part` + `Part::Feature` in the document.
6. Exports `assembly.step` and `assembly.stl` to `exports/`.

### Template

```python
import FreeCAD as App
import Part
import Import
import os
import sys
import shutil
import importlib

try:
    _dir = os.path.dirname(os.path.abspath(__file__))
except NameError:
    _dir = os.getcwd()

if _dir not in sys.path:
    sys.path.insert(0, _dir)

import params
importlib.reload(params)

from part_01_body   import construct_body
from part_02_lid    import construct_lid
# ... import all construct_* functions

CURRENT_DIR  = os.path.dirname(os.path.abspath(__file__))
EXPORT_BASE  = os.path.join(CURRENT_DIR, "exports")


def clear_exports():
    """Delete all files in the exports directory."""
    if os.path.exists(EXPORT_BASE):
        print("Clearing exports/ directory...")
        for item in os.listdir(EXPORT_BASE):
            item_path = os.path.join(EXPORT_BASE, item)
            try:
                if os.path.isfile(item_path) or os.path.islink(item_path):
                    os.unlink(item_path)
                elif os.path.isdir(item_path):
                    shutil.rmtree(item_path)
            except Exception as e:
                print(f"  Warning: could not delete {item_path}: {e}")
    else:
        os.makedirs(EXPORT_BASE)


def load_step(filename):
    """Load a STEP file from exports/ and return its Part.Shape."""
    path = os.path.join(EXPORT_BASE, filename)
    if not os.path.exists(path):
        raise FileNotFoundError(f"STEP file not found: {path}")
    shape = Part.Shape()
    shape.read(path)
    return shape


def main():
    # 1. Clear exports and regenerate all parts
    clear_exports()
    construct_body()
    construct_lid()
    # ... call all construct_* functions

    # 2. Load STEP files
    body = load_step("part_01_body.step")
    lid  = load_step("part_02_lid.step")
    # ...

    # 3. Position parts using App.Placement(translation, rotation)
    # Simple translate:
    lid.translate(App.Vector(0, 0, params.HEIGHT))
    # Translate + rotate:
    hinge.Placement = App.Placement(
        App.Vector(50, 0, params.HEIGHT),
        App.Rotation(App.Vector(0, 0, 1), 180)  # rotate 180° around Z
    )
    # Re-use the same STEP for multiple instances (e.g. 4 screws from one STEP file):
    screw_positions = [(10, 10), (10, -10), (-10, 10), (-10, -10)]
    screw_shapes = []
    for x, y in screw_positions:
        s = load_step("part_03_screw.step")
        s.Placement = App.Placement(App.Vector(x, y, 0), App.Rotation())
        screw_shapes.append(s)

    # 4. Build FreeCAD document
    doc = App.newDocument("Assembly")

    def add_part(doc, name, shape, color=None):
        body_obj = doc.addObject("App::Part", f"Part_{name}")
        feat     = doc.addObject("Part::Feature", f"Shape_{name}")
        feat.Shape = shape
        body_obj.addObject(feat)
        if color and feat.ViewObject:
            feat.ViewObject.ShapeColor = color
        return body_obj

    body_obj = add_part(doc, "Body", body, (0.7, 0.7, 0.7))
    lid_obj  = add_part(doc, "Lid",  lid,  (0.2, 0.6, 0.2))
    # ...

    # 5. Export assembly
    os.makedirs(EXPORT_BASE, exist_ok=True)
    Import.export(
        [body_obj, lid_obj],  # list all part objects
        os.path.join(EXPORT_BASE, "assembly.step")
    )
    compound = Part.makeCompound([body, lid])  # list all shapes
    compound.exportStl(os.path.join(EXPORT_BASE, "assembly.stl"))
    print("Assembly exported.")


if __name__ == "__main__" or __name__ == "assembly":
    main()
```

### Assembly rules

- **Always clear exports first** before regenerating parts.
- **Load from STEP files** (not by calling construct functions directly) so the assembly reflects exactly what was exported.
- **Name every part**: use `App::Part` + `Part::Feature` pairs with descriptive names.
- **Assign colours**: give each part a distinct `ShapeColor` RGB tuple for visual clarity.
- **Export both formats**: `Import.export(...)` for STEP (with part tree); `Part.makeCompound(...).exportStl(...)` for STL.
- **Re-use one STEP for multiple instances**: call `load_step()` once per unique part file, then call it again (or use `.copy()`) for each additional instance with a different `App.Placement`. This is common for fasteners, crossbars, brackets — any part that repeats.
- **Use `App.Placement(vector, rotation)` for combined transforms**: prefer this over chaining `.translate()` + `.rotate()` calls, especially when placing many instances.

---

## Step 5 — export_all.py

`export_all.py` is a standalone helper that regenerates every part using `freecadcmd`:

```python
import os
import glob
import subprocess
import sys
import shutil

FREECAD_CMD = "/Applications/FreeCAD.app/Contents/Resources/bin/freecadcmd"


def run_part_scripts():
    script_dir  = os.path.dirname(os.path.abspath(__file__))
    part_files  = sorted(glob.glob(os.path.join(script_dir, "part_*.py")))
    exports_dir = os.path.join(script_dir, "exports")

    if not part_files:
        print("No part_*.py files found.")
        return

    if not os.path.exists(FREECAD_CMD):
        print(f"Error: FreeCAD not found at {FREECAD_CMD}")
        return

    # Clear exports
    if os.path.exists(exports_dir):
        print("Clearing exports/ ...")
        for item in os.listdir(exports_dir):
            p = os.path.join(exports_dir, item)
            if os.path.isfile(p) or os.path.islink(p):
                os.unlink(p)
            elif os.path.isdir(p):
                shutil.rmtree(p)
    else:
        os.makedirs(exports_dir)

    success = 0
    for part_file in part_files:
        name = os.path.basename(part_file)
        print(f"Running {name}...")
        cmd = f"import runpy; runpy.run_path('{part_file}', run_name='__main__')"
        proc = subprocess.Popen(
            [FREECAD_CMD, "-c", cmd],
            stdout=subprocess.PIPE,
            stderr=subprocess.STDOUT,
            stdin=subprocess.DEVNULL,
            text=True,
        )
        for line in proc.stdout:
            if any(kw in line for kw in ("Export", "Error", "Warning")):
                print(f"  > {line.strip()}")
        rc = proc.wait()
        if rc == 0:
            print(f"  OK: {name}")
            success += 1
        else:
            print(f"  FAILED: {name} (exit {rc})")

    print(f"\nDone: {success}/{len(part_files)} parts exported.")


if __name__ == "__main__":
    run_part_scripts()
```

---

## Step 6 — Folder creation

Always create these three folders in the project root (they may be empty):

```
exports/    ← generated STEP and STL files
3d-print/   ← slicing files, print notes
media/      ← reference images, renders
```

Always create a `.gitignore` file in the project root. Generated files must never be committed.

```
# Generated CAD exports
exports/

# Python cache
__pycache__/
*.pyc
*.pyo

# FreeCAD backup files
*.FCBak

# macOS
.DS_Store
```

---

## Print orientation

When designing each part, consider its print orientation and build it geometry-first with that orientation in mind:

- **Largest flat face on the build plate** — maximises bed adhesion and minimises supports.
- **Avoid overhangs greater than 45°** without explicit support structures; design chamfers or self-supporting angles where possible.
- **Tall narrow parts** (e.g. pins, pegs) should be oriented vertically only if they fit within `BUILD_PLATE_Z`; otherwise lay them flat.
- **Threaded shafts** print best vertically so layer lines run perpendicular to the thread helix.
- **Functional surfaces** (mating faces, bearing surfaces) should face upward or sideways, not against the bed — bed-facing surfaces have rougher texture.
- Note the intended print orientation in a comment at the top of each part file.

---

## Step 7 — Visual Validation

After generating each part and the final assembly, always run visual validation using the **`freecad-visual-validation` skill**. Never skip this step. It covers the full MCP tool parameter reference, required per-part and assembly view sets, what to look for, and the defect reporting procedure.

---

## Checklist before finishing

- [ ] `params.py` has `SCALE = 1.0` as the first line; every dimension multiplied by `SCALE`.
- [ ] Each part file: deletes old STEP/STL before exporting, returns shape from `construct_*()`, has `main()`, has `if __name__ == "__main__": main()`.
- [ ] `assembly.py`: clears exports, regenerates all parts, loads from STEP, positions parts, exports assembly STEP and STL.
- [ ] `export_all.py` present and working.
- [ ] `exports/`, `3d-print/`, `media/`, `spec/`, `spec/reference-images/` folders exist.
- [ ] `spec/` contains at least `mission.md` and `requirements.md`.
- [ ] `spec/reference-images/` checked — any images used as visual reference for parts and assembly.
- [ ] `.gitignore` created with `exports/`, `__pycache__/`, `*.FCBak`, `.DS_Store`.
- [ ] After generating code, visually validate using the MCP FreeCAD tool (`mcp_freecad_execute_freecad_script`).
- [ ] Each part validated from 5 views: Isometric, Front, Top, Right, Isometric close-up (zoom 2.0).
- [ ] Assembly validated from 8 views including low-angle and close-up on joint/interface.
- [ ] No floating geometry, unexpected intersections, or missing parts found.

---

## Threading

For parts that require threaded features, use the **`freecad-threading` skill** — it covers clearance values, trapezoidal profile construction, male/female thread patterns, tip chamfers, and the `makeCompound` vs `fuse` export rules.

## Fits & Tolerances

For sliding fits, rotating fits (hinge pins), press/glue fits, snap tabs, and knuckle construction, use the **`freecad-fits-tolerances` skill**.
