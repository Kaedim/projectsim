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
  - warehouse
task_categories:
  - robotics
size_categories:
  - n<1K
pretty_name: Warehouse Manipulation & States
---

# Warehouse Manipulation & States

> **Hosted on Hugging Face:** [projectsim/warehouse-manipulation-and-states](https://huggingface.co/datasets/projectsim/warehouse-manipulation-and-states/tree/main)

Procedurally generated, sim-ready 3D warehouse assets - cardboard boxes,
pallet-and-box stacks, and wire logistics cages - shipped as OpenUSD (`.usda`)
and MuJoCo XML (`.xml`, MJCF) with per-zone PBR maps, plus a static glTF
(`.glb`) for tooling that doesn't read either. Every variant carries real
per-link mass, inertia, colliders, and, where present, articulation with joint
limits and friction latches.

## Motivation

Warehouse robotics work - pick-and-place of boxes, container manipulation, roll-cage
navigation - needs assets that (a) look like the real thing, (b) sim-correctly
in Isaac Sim / MuJoCo, and (c) come in enough parametric variation that a policy
sees the _distribution_ of shapes and materials, not one shape re-textured.
Public warehouse assets are either scanned singletons (no variation, no reliable
physics) or category-generic props (no articulation, guessed masses). This
dataset is our procedural pipeline's output: every variant has a spec that
produces watertight geometry, sim-authored physics, and PBR textures.

## What's included

| Class            | Count | Articulation                                                     | Materials                                                                                |
| ---------------- | ----- | ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `cardboard_box`  | 75    | 4 top flaps, each on a revolute joint with a 0.01 N·m light hold | body + four flap zones                                                                   |
| `pallet_boxes`   | 75    | static (pallet + stacked boxes prop)                             | pallet, boxes, tape, plus a printed shipping-label zone                                  |
| `logistics_cage` | 75    | 4 swivel casters + 4 wheel axles (8 revolute joints) per variant | painted-metal deck (industrial finish palette), brushed-steel wire, black-rubber casters |

**Total: 225 variants across 3 classes, ~12 GB.**

### Per-variant contents

```
assets/<class>/<vid>/
├── <vid>.usda            OpenUSD, textures referenced as ./textures/*.png
├── <vid>.glb             glTF binary, embedded textures, static pose
├── textures/*.png        PBR maps (base_color, normal, roughness, metallic; +labels on pallet_boxes)
└── mjcf/
    ├── <vid>.xml         MJCF, drop-in for MuJoCo `simulate` (`simulate <vid>.xml`)
    ├── visuals/*.obj     per-zone visual meshes, textured
    ├── visuals/textures/ PBR maps referenced by the MJCF
    └── collision/*.obj   per-component convex hulls (or primitives, per authored collision type)
```

The MJCF mirrors the USD authoring: primitives (`box`, `sphere`) where the rig
authored them, per-component convex hulls elsewhere; intra-asset self-collision
excluded so flaps and moving parts return to their home pose; visual geoms in
group 2 (shown by default), collision geoms in group 3 (press `3` in `simulate`
to reveal).

## Parametric variation covered

Every class is authored from a `spec.json` sampled deterministically from a
`(count, seed)` pair; anchor rows are the class's presets and dimension extremes.

- **`cardboard_box`** - 3 mm single-wall corrugated RSC, always articulated
  (the `has_top_tape` param is pinned to `false` at generation). Body:
  250–500 mm × 156–364 mm × 200–350 mm, bevel-edged panels, optional stadium
  hand-hole cutouts.
- **`pallet_boxes`** - wooden EUR pallet loaded with 3–10 stacked cardboard
  boxes; pallet 1200 × 800 mm ±10 %, per-box dims sampled independently, tape
  strips on selected boxes, printed shipping-label region on one visible face.
- **`logistics_cage`** - 4-sided wire cage with swivel casters; two presets are
  gated (nestable-frame, drop-gate). Base dims 720–1500 × 900–1700 × 700–1000 mm.
  All 75 variants roll (`base_style` is pinned to `swivel_casters` - the sampler
  cannot draw the "welded rigid" alternative).

The exact per-variant parameter values are not shipped in v1.

## Intended use

- Training and evaluating manipulation policies (grasp, place, stack, open flap)
- Perception (segmentation, category ID, pose estimation) from synthetic renders
- Domain-randomization sources for warehouse pipelines
- Sim-based experimentation before scanning real assets
- Prototyping in Isaac Sim / MuJoCo / any USD-aware DCC

## Tested simulators

Verified end-to-end:

- **NVIDIA Isaac Sim 6.0.1** - loads `<vid>.usda` headless and GUI. Doors/flaps
  open under applied effort, casters roll, taped boxes stay one rigid body,
  latches hold under gravity.
- **MuJoCo 3.11.0** - loads `mjcf/<vid>.xml` directly (`simulate mjcf/<vid>.xml`).
  Joint dynamics, mass/inertia, and per-component collision hulls all match the
  authored rig.
- **Blender 5.0** - USD and glTF import both work.

## Physical properties - authored vs estimated

| Property                 | How                                                                                                                | Notes                                                                                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Volume, COM, inertia     | Computed from watertight mesh                                                                                      | Falls back to the convex hull for open-shell parts (cage frame)                                                                                       |
| Mass                     | Volume × density                                                                                                   | Density per zone, classified by a material-analysis VLM. The VLM inspects the object's renders together with the zone names in context — e.g. reading pallet slats as pine, box walls as kraft cardboard, cage wire as galvanised steel, casters as rubber — and looks each zone's density up in a shared material table (wood ~600, kraft ~180, steel ~7850, rubber ~1200 kg/m³). Per-link mass = mesh volume × classified density. |
| Collision shape          | Auto-classified per link                                                                                           | Author-time collision types per link, taken directly from the rig:<br/>• **primitives** (`box`, `sphere`) where the rig is geometric — wheels, casters, simple frames<br/>• **`convexHull`** for sealed volumes with a single graspable shape (cardboard flaps, cage forks)<br/>• **`convexDecomposition`** with PhysX shrink-wrap for multi-part rigid props whose interior matters (pallet_boxes: pallet blocks + individual boxes each become their own hull at load)<br/>• **`sdf`** (`sdfResolution 256`) for thin, non-convex shells where a decomposition would seal openings (logistics_cage frame, cardboard_box body)<br/><br/>MJCF ships the same intent: primitives verbatim, per-connected-component convex hulls where the USD uses `convexHull` / `convexDecomposition`, and a single mesh geom for `sdf` shells (MuJoCo auto-hulls it). |
| Joint limits             | Authored in the rig                                                                                                | Explicit degrees / meters per joint                                                                                                                   |
| Joint friction / latches | Authored                                                                                                           | Cardboard flaps: 0.01 N·m soft latch (opens by hand, doesn't self-open). No other latches in v1                                                       |
| Textures                 | 4K PBR maps (`cardboard_box`, `pallet_boxes`); procedural PBR from an industrial finish palette (`logistics_cage`) | Zone-mapped so a downstream engine can rebind materials per zone                                                                                      |
| `max_mass_kg`            | Cavity volume × water density                                                                                      | Meta-only hint for pick-and-place - **not** the physics mass                                                                                          |

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
@dataset{kaedim_warehouse_manipulation_and_states_2026,
  author = {{Kaedim} and {ProjectSim}},
  title = {Warehouse Manipulation \& States},
  year = {2026},
  publisher = {Hugging Face},
  version = {1.0},
  url = {https://huggingface.co/datasets/projectsim/warehouse-manipulation-and-states}
}
```

## Maintenance

New classes and variants are added as separate top-level `assets/<class>/`
folders; the on-disk `<vid>` prefix is stable across future commits - a
`v03_pallet_boxes_6c1b6d` today will be the same asset next release. Regressions
are republished as an additive commit; assets are only ever removed with a
version bump.

Issues / questions: open a Discussion on this dataset repo, or email
`hello@kaedim3d.com`.
