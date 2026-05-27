# Skill: FreeCAD Fits & Tolerances

This skill covers designing mating parts for FDM 3D printing — sliding fits, rotating fits, press/glue fits, snap tabs, and hinge pin/bore pairs. Use this skill whenever two printed parts need to fit together.

> **Standalone**: this skill runs independently. Code examples reference `params.SCALE` — if the project uses a `params.py` (see `freecad-project` agent) this is already defined. If not, define `SCALE = 1.0` locally and substitute `params.SCALE` → `SCALE`.

---

## Clearance reference table

| Fit type | Radial clearance | Diameter clearance | Feel |
|---|---|---|---|
| Sliding / insertion | 1.0–2.0 × SCALE per side | 2.0–4.0 × SCALE | Pushes in by hand, pulls out cleanly |
| Rotating (hinge pin) | 0.3 × SCALE per side | 0.6 × SCALE | Spins freely, no wobble |
| Press / glue fit | 0.2 × SCALE per side | 0.4 × SCALE | Tight push-fit, use adhesive to lock |
| Snap tab catch | 1.5 × SCALE depth | — | Clicks in, resists pull-out |

Define all clearance constants in `params.py`:

```python
SLIDING_CLEARANCE = 1.0 * SCALE   # per side — ring sliding into housing
ROTATING_CLEARANCE = 0.3 * SCALE  # per side — hinge pin in bore
PRESS_CLEARANCE    = 0.2 * SCALE  # per side — peg in socket (glue fit)
```

---

## 1. Sliding / insertion fit

Used for compression rings, lid lips, sleeves that insert by hand and can be removed.

```python
# Housing inner dimension: e.g. bin interior width = 100mm
# Part outer dimension: subtract clearance on each side
HOUSING_INNER_W = 100.0 * params.SCALE
PART_OUTER_W    = HOUSING_INNER_W - 2 * params.SLIDING_CLEARANCE

# Build the part at PART_OUTER_W — it will slide into the housing cleanly
ring = create_rounded_box(PART_OUTER_W, PART_OUTER_L, RING_HEIGHT, corner_radius)
```

**Rules:**
- Apply clearance to the **inserted part** (shrink it); leave the housing at nominal.
- Use `1.0 × SCALE` for precise fits (lid lip, guide slot). Use `2.0 × SCALE` for easy hand-insertion (compression ring).
- The fit should be hand-removable without tools.

---

## 2. Rotating fit (hinge pin in bore)

Used for hinge pins, axles, pivots that spin inside a bore.

```python
# Pin (male)
PIN_RADIUS = 4.5 * params.SCALE

# Bore (female) — add rotating clearance
BORE_RADIUS = PIN_RADIUS + params.ROTATING_CLEARANCE   # = 4.8 * SCALE

# Build pin
pin = Part.makeCylinder(PIN_RADIUS, PIN_LENGTH)

# Build bore in the knuckle/housing body
bore = Part.makeCylinder(BORE_RADIUS, BORE_DEPTH)
bore.translate(App.Vector(0, 0, z_start))
knuckle = knuckle.cut(bore)
```

**Rules:**
- Clearance goes on the **bore** (enlarge it); keep the pin at nominal.
- Add a **lead-in chamfer** at the bore entrance to guide pin insertion:
  ```python
  chamfer_depth = 1.5 * params.SCALE
  lead_in = Part.makeCone(BORE_RADIUS + chamfer_depth, BORE_RADIUS, chamfer_depth)
  lead_in.translate(App.Vector(0, 0, z_start))
  knuckle = knuckle.cut(lead_in)
  ```
- For long pins spanning multiple knuckles, ensure all bores share the same axis — misalignment of even 0.5mm will jam the pin.

---

## 3. Press / glue fit (peg into socket)

Used for alignment pegs, dowels, and pins that lock with adhesive after assembly.

```python
# Peg (male) — at nominal
PEG_RADIUS = 2.6 * params.SCALE
PEG_LENGTH = 8.5 * params.SCALE

peg = Part.makeCylinder(PEG_RADIUS, PEG_LENGTH - 0.5 * params.SCALE)

# Chamfered tip for easy entry
tip = Part.makeCone(PEG_RADIUS, PEG_RADIUS - 0.5 * params.SCALE, 0.5 * params.SCALE)
tip.translate(App.Vector(0, 0, PEG_LENGTH - 0.5 * params.SCALE))
peg = peg.fuse(tip).removeSplitter()

# Socket (female) — add press clearance
SOCKET_RADIUS = PEG_RADIUS + params.PRESS_CLEARANCE   # = 2.8 * SCALE
SOCKET_DEPTH  = PEG_LENGTH + 1.5 * params.SCALE       # slightly deeper than peg

socket = Part.makeCylinder(SOCKET_RADIUS, SOCKET_DEPTH)
body = body.cut(socket)
```

**Rules:**
- Apply clearance to the **socket** (enlarge it); keep the peg at nominal.
- Socket depth should be `PEG_LENGTH + 1.0–2.0 × SCALE` so the peg bottoms out with a small air gap.
- Always add a chamfered tip on the peg for easy alignment during assembly.
- Designed for permanent assembly with adhesive — not intended to be disassembled.

---

## 4. Snap tab / catch

Used for lid clips, compression ring retention, panel latches.

```python
SNAP_TAB_W     = 10.0 * params.SCALE   # tab width
SNAP_TAB_DEPTH = 1.5 * params.SCALE    # protrusion depth
SNAP_TAB_H     = 4.0 * params.SCALE    # tab height

# Tab on the inserted part (protrudes outward)
tab = Part.makeBox(SNAP_TAB_W, SNAP_TAB_DEPTH, SNAP_TAB_H)
tab.translate(App.Vector(-SNAP_TAB_W / 2, housing_wall_inner_y, z_tab))

# Ramp on leading face so the tab deflects on insertion
ramp = Part.makeWedge(SNAP_TAB_W, SNAP_TAB_DEPTH, SNAP_TAB_H, 0, 0, SNAP_TAB_W, SNAP_TAB_DEPTH, 0, SNAP_TAB_H, 0)
# ... orient and position ramp on the tab face

# Matching groove in the housing (same depth + 0.2mm clearance)
groove_depth = SNAP_TAB_DEPTH + 0.2 * params.SCALE
groove = Part.makeBox(SNAP_TAB_W + 1.0 * params.SCALE, groove_depth, SNAP_TAB_H + 0.5 * params.SCALE)
groove.translate(App.Vector(-(SNAP_TAB_W + 1.0 * params.SCALE) / 2, housing_wall_inner_y - groove_depth, z_tab - 0.25 * params.SCALE))
housing = housing.cut(groove)
```

**Rules:**
- Wall behind the tab must be thin enough to flex: `1.5–2.5 × SCALE` thick.
- Tab protrusion `1.5 × SCALE` works well for walls 2–3mm thick at `SCALE = 1.0`.
- Groove should be `0.2 × SCALE` wider and taller than the tab for reliable snap.
- Add a leading ramp (15–30° angle) on the tab so insertion doesn't require prying.
- Print tab-bearing walls vertically so layer lines run along the flex direction.

---

## 5. Hinge knuckle construction

Full knuckle + pin pattern as used in carry-bag-bin:

```python
def make_knuckle(knuckle_width, knuckle_ext, pin_radius, rotating_clearance):
    """Returns the knuckle body with bore already cut."""
    bore_radius = pin_radius + rotating_clearance
    k_height = bore_radius * 2 + 2.0 * params.SCALE  # outer radius = bore_radius + wall

    # Rectangular body + semicylinder end
    box = Part.makeBox(knuckle_width, knuckle_ext, k_height)
    cap = Part.makeCylinder(k_height / 2.0, knuckle_width)
    cap.rotate(App.Vector(0, 0, 0), App.Vector(0, 1, 0), 90)
    cap.translate(App.Vector(0, knuckle_ext, k_height / 2.0))
    knuckle = box.fuse(cap).removeSplitter()

    # Pin bore through knuckle (along X axis)
    bore = Part.makeCylinder(bore_radius, knuckle_width + 2.0)
    bore.rotate(App.Vector(0, 0, 0), App.Vector(0, 1, 0), 90)
    bore.translate(App.Vector(-1.0, knuckle_ext, k_height / 2.0))
    knuckle = knuckle.cut(bore).removeSplitter()

    return knuckle
```

---

## Rules summary

- **Clearance always goes on the receiving (female) part** — shrink the male, or enlarge the female hole/bore/groove.
- **Peg tips and pin ends always get a chamfer** — self-guiding on insertion.
- **`removeSplitter()`** after every fuse/cut chain involving fit features.
- **Define all clearance constants in `params.py`** — never hard-code 0.3 or 2.0 inside part files.
- **Bore depth > peg length** — always leave a small air gap at the bottom of any socket.
- For threading fits, use the **`freecad-threading` skill**.
