# Skill: FreeCAD Visual Validation

This skill defines how to visually validate FreeCAD parts and assemblies using `mcp_freecad_execute_freecad_script`. Run validation after every part is created or modified, and after every assembly build. Never skip this step.

> **Standalone**: this skill runs independently of any project structure. All it needs is the absolute path to a `.py` part or assembly script.

---

## Tool parameters reference

| Parameter | Values | Notes |
|-----------|--------|-------|
| `script_path` | absolute path to `.py` file | required |
| `view_angle` | `Isometric`, `Top`, `Bottom`, `Front`, `Back`, `Left`, `Right` | ignored when `azimuth`+`elevation` are set |
| `azimuth` | 0–360° | custom horizontal camera angle |
| `elevation` | -90–90° | custom vertical camera angle |
| `zoom` | `1.0` = fit all, `2.0` = 2× closer, `0.5` = 2× farther | |
| `width` / `height` | pixels, defaults 800×600 | |
| `background` | hex string e.g. `'#ffffff'` | |

---

## Per-part validation — required views

Run these for **every part file** after it is created or modified:

| Call | `view_angle` | `zoom` | Purpose |
|------|-------------|--------|---------|
| 1 | `Isometric` | `1.0` | Overall shape and proportions |
| 2 | `Front` | `1.0` | Front profile, wall thickness |
| 3 | `Top` | `1.0` | Top-down footprint |
| 4 | `Right` | `1.0` | Side profile |
| 5 | `Isometric` | `2.0` | Close-up to inspect detail features (holes, ribs, grooves, fillets) |

If the part has small or complex features (threads, snap tabs, hinge knuckles), add:

| Call | `azimuth` | `elevation` | `zoom` | Purpose |
|------|-----------|-------------|--------|---------|
| 6 | 45 | 30 | `3.0` | Tight close-up from a custom angle on the feature |
| 7 | 225 | 20 | `3.0` | Opposite side of the same feature |

---

## Assembly validation — required views

Run these after `assembly.py` completes:

| Call | `view_angle` / angles | `zoom` | Purpose |
|------|-----------------------|--------|---------|
| 1 | `Isometric` | `1.0` | Full assembly, all parts visible |
| 2 | `Front` | `1.0` | Front elevation, vertical alignment |
| 3 | `Top` | `1.0` | Top-down, horizontal layout |
| 4 | `Right` | `1.0` | Side elevation |
| 5 | `Back` | `1.0` | Rear — check nothing is missing or protruding |
| 6 | azimuth=45, elevation=15 | `1.2` | Low-angle 3/4 view — good for spotting z-offset errors |
| 7 | azimuth=225, elevation=15 | `1.2` | Opposite low-angle 3/4 |
| 8 | `Isometric` | `2.5` | Close-up on the primary joint/interface between parts |

---

## What to check at each view

- **No floating geometry**: every part sits at its intended position; nothing is hanging in mid-air.
- **No unexpected intersections**: parts that should not overlap do not penetrate each other.
- **Correct proportions**: compare against `spec/reference-images/` if available — overall silhouette should match.
- **Wall thickness visible**: thin walls should appear as distinct surfaces, not paper-thin slivers.
- **Features present**: holes, ribs, grooves, tabs, and knuckles must be visible and correctly placed.
- **Assembly completeness**: all parts listed in `assembly.py` are visible in the render; none are missing.
- **Colour coding**: each part has a distinct colour — if two adjacent parts look identical in colour, check the `ShapeColor` assignments.

---

## Defect report

If any view reveals a defect, stop and fix it before proceeding. Document the defect and fix in a comment in the relevant part file. Re-run all required views for that part before continuing.
