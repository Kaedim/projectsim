---
license: cc-by-4.0
tags:
  - robotics
  - manipulation
  - simulation
  - synthetic-data
  - usd
  - mjcf
  - isaac-sim
  - mujoco
  - articulation
  - furniture
  - appliances
task_categories:
  - robotics
size_categories:
  - n<1K
pretty_name: Articulated Storage & Appliances
---

# Articulated Storage & Appliances

> **Hosted on Hugging Face:** [projectsim/articulated-storage-and-appliances](https://huggingface.co/datasets/projectsim/articulated-storage-and-appliances/tree/main)

Procedurally generated, sim-ready 3D storage furniture and appliances with real
articulation - drawers that slide, doors that swing on friction-latched hinges -
shipped as OpenUSD (`.usda`) and MuJoCo XML (`.xml`, MJCF) with per-zone PBR
maps, plus a static glTF (`.glb`) for tooling that doesn't read either. Every
variant carries real per-link mass, inertia, colliders, joint limits and
friction latches.

## Motivation

Learning-based manipulation policies for storage furniture and household
appliances need articulated assets that (a) look real, (b) hold closed under
gravity and open under applied effort (no self-opening doors at t=0), and (c)
come with enough parametric variation to expose the _distribution_ of shapes and
materials, not one shape re-textured. Public assets are either scanned
singletons (no articulation, no reliable physics) or category-generic props (no
PBR, no joints). This dataset is our procedural pipeline's output: every
variant has a spec that produces watertight geometry, sim-authored articulation
and physics, and PBR textures.

## What's included

| Class               | Count | Articulation                                                                            | Materials                                               |
| ------------------- | ----- | --------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `desk_with_drawers` | 75    | 1-6 prismatic drawer joints per variant, damped so a released drawer holds its position | top, carcass, drawer fronts, handles, runners           |
| `oven`              | 75    | one revolute door joint per variant with an authored gasket / magnetic-catch latch      | body, door, handle, control panel, glass window, cavity |
| `wardrobe`          | 75    | 1-2 revolute door joints per variant on outward-swinging hinges with an 8 N·m catch     | body, doors, handles, interior rail, base               |

**Total: 225 variants across 3 classes, ~12 GB.**

### Per-variant contents

```
assets/<class>/<vid>/
├── <vid>.usda            OpenUSD, textures referenced as ./textures/*.png
├── <vid>.glb             glTF binary, embedded textures, static pose
├── textures/*.png        PBR maps (base_color, normal, roughness, metallic)
└── mjcf/
    ├── <vid>.xml         MJCF, drop-in for MuJoCo `simulate` (`simulate <vid>.xml`)
    ├── visuals/*.obj     per-zone visual meshes, textured
    ├── visuals/textures/ PBR maps referenced by the MJCF
    └── collision/*.obj   per-component convex hulls (or primitives, per authored collision type)
```

The MJCF mirrors the USD authoring: primitives (`box`, `sphere`) where the rig
authored them, per-component convex hulls elsewhere; drawer slides and door
hinges carry the same limits, damping, and friction catches as the USD;
intra-asset self-collision is excluded so latched doors and stored drawers
return to their home pose. Visual geoms sit in group 2 (shown by default),
collision geoms in group 3 (press `3` in `simulate` to reveal).

## Parametric variation covered

Every class is authored from a `spec.json` sampled deterministically from a
`(count, seed)` pair; anchor rows are the class's presets and dimension extremes.

- **`desk_with_drawers`** - a rectangular writing desk with panel construction.
  Four canonical layouts sampled by the class:
  - _standard single pedestal_ - three-drawer pedestal on one side
  - _executive twin pedestal_ - pedestal columns on both sides
  - _slim writing / center drawer_ - single centre-mount drawer
  - _office file / pedestal panel end_ - file drawer at the top, letter drawers below
    Overall dimensions vary between 900 × 550 × 720 mm and 1800 × 900 × 820 mm.
- **`oven`** - a rectangular free-standing oven with a hinged front door.
  Body 600 × 600 × 600 mm to 900 × 750 × 620 mm, `sheet_thickness_mm` capped at 4
  and `door_thickness_mm` at 70 so body/door masses land in the 25-100 kg
  range.
- **`wardrobe`** - a freestanding closet with hinged doors and internal shelving.
  Body 600-1500 × 500-650 × 1800-2400 mm; the sampler draws only single and
  double-door layouts

The exact per-variant parameter values are not shipped in v1.

## Intended use

- Training and evaluating manipulation policies (grasp handle, open/close drawer or door)
- Perception (segmentation, pose estimation, articulation-state inference)
- Domain-randomization sources for household and appliance robotics pipelines
- Sim-based experimentation before scanning real furniture
- Prototyping in Isaac Sim / MuJoCo / any USD-aware DCC

## Tested simulators

Verified end-to-end:

- **NVIDIA Isaac Sim 6.0.1** - loads `<vid>.usda` headless and GUI. Drawers
  slide under applied effort, doors swing on their hinges and hold closed under
  gravity, latches break away under a hand push.
- **MuJoCo 3.11.0** - loads `mjcf/<vid>.xml` directly (`simulate mjcf/<vid>.xml`).
  Prismatic drawer joints and revolute door joints carry the authored limits,
  damping, and latch friction; move a drawer with the "Joint" slider in the
  simulate GUI or via `data.qpos`.
- **Blender 5.0** - USD and glTF import both work.

## Physical properties - authored vs estimated

| Property                 | How                                                                                                           | Notes                                                                                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Volume, COM, inertia     | Computed from watertight mesh                                                                                 | Falls back to the convex hull for open-shell parts                                                                                                    |
| Mass                     | Volume × density                                                                                              | Density per zone, classified by a material-analysis VLM. The VLM inspects the object's renders together with the zone names in context — e.g. reading a wardrobe carcass as MDF, an oven door as painted steel, drawer runners as bright steel, door glass as tempered glass — and looks each zone's density up in a shared material table (MDF ~750, steel ~7850, glass ~2500, plastic ~1200 kg/m³). Per-link mass = mesh volume × classified density. |
| Collision shape          | Auto-classified per link                                                                                      | Author-time collision types per link, taken directly from the rig:<br/>• **primitives** (`box`, `sphere`) where the rig is geometric<br/>• **`convexHull`** for sealed volumes with a single graspable shape (drawer boxes, wardrobe doors)<br/>• **`convexDecomposition`** with PhysX shrink-wrap for multi-part rigid props whose interior matters<br/>• **`sdf`** (`sdfResolution 256`) for thin panels where a decomposition would seal openings (oven cavity, wardrobe interior, drawer walls)<br/><br/>MJCF ships the same intent: primitives verbatim, per-connected-component convex hulls where the USD uses `convexHull` / `convexDecomposition`, and a single mesh geom for `sdf` shells (MuJoCo auto-hulls it). |
| Joint limits             | Authored in the rig                                                                                           | Explicit meters per drawer, degrees per door                                                                                                          |
| Joint friction / latches | Authored                                                                                                      | Drawer joints carry damping so a released drawer holds; oven and wardrobe doors carry an 8 N·m friction catch                                         |
| Textures                 | 4K PBR maps per zone (photoreal retexturing on most variants; a curated palette on the appliance shader half) | Zone-mapped so a downstream engine can rebind materials per zone                                                                                      |
| `max_mass_kg`            | Cavity volume × water density                                                                                 | Meta-only hint for pick-and-place - **not** the physics mass                                                                                          |

## Limitations

- **Physics lives in the `.usda` and `.xml`, not the `.glb`.** glTF has no
  physics equivalent - the `.glb` is a _visual_ copy (static pose, single mesh
  per link, no joints, no colliders). Use it for previews, tooling that reads
  neither USD nor MJCF, Blender/Unity/three.js/model-viewer; use `.usda` for
  Isaac Sim, `mjcf/<vid>.xml` for MuJoCo.

## License

Files: **CC BY 4.0** - free to use, share, and adapt for any purpose,
including commercial use; attribution required.

## Citation

```bibtex
@dataset{kaedim_articulated_storage_and_appliances_2026,
  author = {{Kaedim} and {ProjectSim}},
  title = {Articulated Storage \& Appliances},
  year = {2026},
  publisher = {Hugging Face},
  version = {1.0},
  url = {https://huggingface.co/datasets/projectsim/articulated-storage-and-appliances}
}
```

## Maintenance

New classes and variants are added as separate top-level `assets/<class>/`
folders; the on-disk `<vid>` prefix is stable across future commits - a
`v03_desk_with_drawers_f4f381` today will be the same asset next release.
Regressions are republished as an additive commit; assets are only ever removed
with a version bump.

Issues / questions: open a Discussion on this dataset repo, or email
`hello@kaedim3d.com`.
