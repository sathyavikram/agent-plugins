# Skill: FreeCAD Threading

This skill covers implementing 3D-print-ready threaded features in FreeCAD Python scripts using `Part.makeHelix` and `makePipeShell`. Use this skill whenever a part requires male (external) or female (internal) threads.

> **Standalone**: this skill runs independently. Code examples reference `params.SCALE`, `params.THREAD_CLEARANCE`, and `params.GENERAL_CLEARANCE` — if the project uses a `params.py` (see `freecad-project` agent) these are already defined. If not, add these local constants at the top of your part file:
> ```python
> SCALE             = 1.0
> THREAD_CLEARANCE  = 0.6 * SCALE
> GENERAL_CLEARANCE = 0.4 * SCALE
> ```
> then substitute `params.SCALE` → `SCALE`, `params.THREAD_CLEARANCE` → `THREAD_CLEARANCE`, etc.

---

## Clearance rules for 3D-printed threads

Threads printed on FDM printers need extra clearance beyond nominal dimensions to rotate and engage freely after printing. Always apply these clearances:

| Parameter | Value | Notes |
|-----------|-------|-------|
| `THREAD_CLEARANCE` | `0.6 * SCALE` | Shrink male (external) thread outer radius by this amount |
| `GENERAL_CLEARANCE` | `0.4 * SCALE` | Used for hex prisms, sliding fits, pin holes |

Define both in `params.py`:

```python
THREAD_CLEARANCE  = 0.6 * SCALE   # shrink male thread OD for easy rotation after printing
GENERAL_CLEARANCE = 0.4 * SCALE   # sliding fits, hex holes, pins
```

---

## Thread geometry — key relationships

```
t_pitch    = pitch in mm (e.g. 5.0 * SCALE)
t_radius   = nominal outer radius − THREAD_CLEARANCE   ← apply only to MALE thread
t_r_inner  = t_radius − (t_pitch * 0.45)               ← root radius
```

The female (internal) thread is always cut at **nominal dimensions** — clearance comes entirely from shrinking the male outer radius.

---

## Trapezoidal thread profile (4-point polygon)

```python
inner_X = t_r_inner - 2.0 * SCALE
p1 = App.Vector(inner_X,   0, -t_pitch * 0.35)   # tooth base leading edge
p2 = App.Vector(t_radius,  0, -t_pitch * 0.10)   # tooth crest leading edge
p3 = App.Vector(t_radius,  0,  t_pitch * 0.10)   # tooth crest trailing edge
p4 = App.Vector(inner_X,   0,  t_pitch * 0.35)   # tooth base trailing edge
t_wire = Part.Wire(Part.makePolygon([p1, p2, p3, p4, p1]))
```

---

## Male (external) thread construction

```python
t_pitch   = 5.0 * params.SCALE
t_radius  = 12.0 * params.SCALE - params.THREAD_CLEARANCE  # shrink for clearance
t_r_inner = t_radius - (t_pitch * 0.45)
t_length  = 30.0 * params.SCALE

# Helix path
t_helix = Part.makeHelix(t_pitch, t_length, t_r_inner, 0)

# Thread profile
inner_X = t_r_inner - 2.0 * params.SCALE
p1 = App.Vector(inner_X,  0, -t_pitch * 0.35)
p2 = App.Vector(t_radius, 0, -t_pitch * 0.10)
p3 = App.Vector(t_radius, 0,  t_pitch * 0.10)
p4 = App.Vector(inner_X,  0,  t_pitch * 0.35)
t_wire = Part.Wire(Part.makePolygon([p1, p2, p3, p4, p1]))

# Sweep
t_sweep = Part.Wire(t_helix).makePipeShell([t_wire], True, True)
t_sweep.Placement = App.Placement(App.Vector(0, 0, thread_start_z), App.Rotation(0,0,0,1))

# Core cylinder
t_core = Part.makeCylinder(t_r_inner, t_length, App.Vector(0, 0, thread_start_z))

# IMPORTANT: use makeCompound, NOT fuse(), to avoid OpenCASCADE boolean hangs.
# Overlapping geometry exports and slices correctly in Bambu/PrusaSlicer.
full_part = Part.makeCompound([shaft, t_core, t_sweep])
```

---

## Female (internal) thread construction

Female threads are cut into a body at **nominal dimensions** (no clearance reduction):

```python
t_pitch   = 5.0 * params.SCALE
t_radius  = 12.0 * params.SCALE          # nominal — NO clearance reduction
t_r_inner = t_radius - (t_pitch * 0.45)
t_length  = 30.0 * params.SCALE

t_helix = Part.makeHelix(t_pitch, t_length, t_r_inner, 0)

inner_X = t_r_inner - 2.0 * params.SCALE
p1 = App.Vector(inner_X,  0, -t_pitch * 0.35)
p2 = App.Vector(t_radius, 0, -t_pitch * 0.10)
p3 = App.Vector(t_radius, 0,  t_pitch * 0.10)
p4 = App.Vector(inner_X,  0,  t_pitch * 0.35)
t_wire = Part.Wire(Part.makePolygon([p1, p2, p3, p4, p1]))

t_sweep = Part.Wire(t_helix).makePipeShell([t_wire], True, True)
t_sweep.Placement = App.Placement(App.Vector(0, 0, hole_start_z), App.Rotation(0,0,0,1))
t_core  = Part.makeCylinder(t_r_inner, t_length + 2.0, App.Vector(0, 0, hole_start_z - 1.0))

thread_cutter = t_core.fuse(t_sweep).removeSplitter()
body = body.cut(thread_cutter)
```

---

## Thread tip chamfer (entry bevel)

Always add a chamfer at the tip of the male thread so it self-guides into the female socket:

```python
chamfer = Part.makeCone(
    t_radius + 2.0, t_r_inner,
    t_pitch / 2 + 1,
    App.Vector(0, 0, thread_start_z + t_length - t_pitch / 2 - 1)
)
end_cutter = Part.makeCylinder(
    t_radius + 5.0, t_pitch + 2.0,
    App.Vector(0, 0, thread_start_z + t_length - 1)
)
thread_part = thread_part.cut(end_cutter.cut(chamfer))
```

---

## Rules summary

- **Always shrink the male outer radius** by `THREAD_CLEARANCE` (0.6 × SCALE). Never adjust the female.
- **Root radius** = `t_radius − (t_pitch × 0.45)` for both male and female.
- **Use `makeCompound`** (not `.fuse()`) when combining a thread sweep with a body for export — avoids OpenCASCADE boolean hangs and slices correctly in all major slicers.
- **Use `.fuse().removeSplitter()`** only when the thread is a cutter (female), not when it is part of the exported geometry.
- **Add a tip chamfer** on all male threads to aid thread engagement.
- **Define all thread params in `params.py`**: `THREAD_CLEARANCE`, `GENERAL_CLEARANCE`, nominal radii, and pitches.
- **Pitch guideline**: for small parts use `pitch = 2.0–4.0 × SCALE`; for larger structural threads use `pitch = 5.0–8.0 × SCALE`.
