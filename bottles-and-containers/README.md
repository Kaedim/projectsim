---
license: cc-by-4.0
tags:
  - robotics
  - manipulation
  - simulation
  - synthetic-data
  - usd
  - isaac-sim
  - mujoco
  - tabletop
  - grasping
task_categories:
  - robotics
size_categories:
  - n<1K
pretty_name: Bottles & Containers
---

# Bottles & Containers

> **Hosted on Hugging Face:** [projectsim/bottles-and-containers](https://huggingface.co/datasets/projectsim/bottles-and-containers)

Procedurally generated, sim-ready 3D tabletop containers - aluminium beverage
cans, glass food jars, and ceramic coffee mugs - shipped as OpenUSD (`.usda`)
with per-zone PBR maps, plus a static glTF (`.glb`) for tooling that doesn't
read USD. Every variant carries real per-link mass, inertia and colliders
authored from the class rig, with realistic packaging formats and standard
sizes.

## Motivation

Tabletop manipulation policies need containers that (a) look real,
(b) sim-correctly in Isaac Sim / MuJoCo, and (c) span the _distribution_ of
sizes and materials a real robot encounters (cans in every common ml format,
jars in mason / jam / pickle proportions, mugs in the diner-to-espresso range),
not one shape re-textured. Public assets are either scanned singletons or
category-generic props (no PBR, guessed masses). This dataset is our
procedural pipeline's output: every variant has a spec that produces
watertight geometry, sim-authored physics, and PBR textures.

## What's included

| Class          | Count | Articulation                              | Materials                                                                          |
| -------------- | ----- | ----------------------------------------- | ---------------------------------------------------------------------------------- |
| `beverage_can` | 34    | static (rolled seam + stay-tab fixed)     | label (printed), lid, rim, bottom, tab                                             |
| `glass_jar`    | 40    | static (screw lid modelled fixed)         | body (transparent glass), lid (painted steel), label (printed band)                |
| `mug`          | 75    | static (rigid handle)                     | body (ceramic), handle (ceramic), base                                             |

**Total: 149 variants across 3 classes, ~3.1 GB.**

### Per-variant contents

```
assets/<class>/<vid>/
├── <vid>.usda            OpenUSD, textures referenced as ./textures/*.png
├── <vid>.glb             glTF binary, embedded textures, static pose
└── textures/*.png        PBR maps (base_color, normal, roughness, metallic)
```

## Parametric variation covered

Every class is authored from a `spec.json` sampled deterministically from a
`(count, seed)` pair; anchor rows are the class's presets and dimension
extremes.

- **`beverage_can`** - thin-walled aluminium can with a necked top, rolled
  double-seam rim, dome bottom, and stay-tab. A `can_format` enum drives
  real-world formats: 330 ml (Ø66 x 115 mm), 355 ml (Ø66 x 122 mm), 440 ml
  (Ø66 x 149 mm), 500 ml (Ø66 x 168 mm), 568 ml UK pint (Ø66 x 189 mm), and
  slim/sleek variants where applicable. Label print varies (soda, beer, energy
  drink styles).
- **`glass_jar`** - cylindrical jar with a threaded neck and a metal screw
  lid. Real jam / pickle / mason-jar formats: 200 ml, 370 ml, 500 ml, 720 ml,
  1000 ml. Slightly rounded shoulders, 2-3 shallow helical ridges at the
  thread. Painted-steel lid, printed label band.
- **`mug`** - ceramic coffee mug with a C-shaped handle. Body Ø65-95 mm,
  height 80-130 mm; capacity roughly 200-500 ml. Handle geometry sampled from
  a small library of C-shape parameters; base can be flat or slightly
  countersunk. Photoreal retexturing on one half, shader palette
  (cream/blue/red/black glaze) on the remainder.

The exact per-variant parameter values are not shipped in v1.

## Intended use

- Training and evaluating manipulation policies (grasp handle, pick can from
  tray, uncap-simulate on jars via a scripted lid replacement)
- Perception (segmentation, category ID, pose estimation) from synthetic
  renders
- Domain-randomization sources for tabletop and kitchen robotics pipelines
- Sim-based experimentation before scanning real containers
- Prototyping in Isaac Sim / MuJoCo / any USD-aware DCC

## Tested simulators

Verified end-to-end:

- **NVIDIA Isaac Sim 6.0.1** - loads `<vid>.usda` headless and GUI. Cans /
  jars / mugs stack, roll, and grasp under gravity with authored mass and
  friction.
- **MuJoCo 3.11.0** - USD is not native; use our sibling MJCF drop or the
  sim-ready `usd2mjcf` converter (matching physics for static props).
- **Blender 5.0** - USD and glTF import both work.

## Physical properties - authored vs estimated

| Property                 | How                                                                                                                | Notes                                                                                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Volume, COM, inertia     | Computed from watertight mesh                                                                                      | Falls back to the convex hull for open-shell parts                                                                                                    |
| Mass                     | Volume × density                                                                                                   | Density per zone, classified by a material-analysis VLM. The VLM inspects the object's renders together with the zone names in context — e.g. reading a can body as aluminium, a jar body as soda-lime glass, a jar lid as painted tinplate, a mug body as glazed ceramic — and looks each zone's density up in a shared material table (aluminium ~2700, glass ~2500, tinplate ~7850, ceramic ~2400 kg/m³). Per-link mass = mesh volume × classified density. |
| Collision shape          | Auto-classified per link                                                                                           | Author-time collision types per link, taken directly from the rig:<br/>• **primitives** (`cylinder`, `box`) where the rig is geometric — can bodies, jar bodies, mug bodies<br/>• **`convexHull`** for sealed volumes with a single graspable shape (jar lids, mug handles)<br/>• **`convexDecomposition`** with PhysX shrink-wrap for multi-part rigid props (mug + handle authored as one link but decomposed at load)<br/>• **`sdf`** (`sdfResolution 256`) for thin non-convex shells (jar shoulder, can neck) |
| Textures                 | 4K PBR maps per zone (photoreal retexturing on one half; curated shader palette on the other)                       | Zone-mapped so a downstream engine can rebind materials per zone. Glass jar body renders with an authored OpenGlass MDL surface in Isaac; a translucent UsdPreviewSurface elsewhere. |
| `max_mass_kg`            | Cavity volume × water density                                                                                      | Meta-only hint for pick-and-place - **not** the physics mass. Cans and jars carry their advertised capacity by size enum.                             |

## Limitations

- **Physics is in the `.usda`, not the `.glb`.** glTF has no physics
  equivalent - the `.glb` is a _visual_ copy (static pose, single mesh per
  link, no colliders). Use it for previews, tooling that can't read USD,
  Blender/Unity/three.js/model-viewer; use the `.usda` for simulation.
- All three classes are static props - jar lids and can tabs are welded rigid,
  mug handles are rigid extensions of the body. Open / uncap behaviours are
  out of scope for v1.

## License

Files: **CC BY 4.0** - free to use, share, and adapt for any purpose,
including commercial use; attribution required.

## Citation

```bibtex
@dataset{kaedim_bottles_and_containers_2026,
  author = {{Kaedim} and {ProjectSim}},
  title = {Bottles \& Containers},
  year = {2026},
  publisher = {Hugging Face},
  version = {1.0},
  url = {https://huggingface.co/datasets/projectsim/bottles-and-containers}
}
```

## Maintenance

New classes and variants are added as separate top-level `assets/<class>/`
folders; the on-disk `<vid>` prefix is stable across future commits - a
`v03_glass_jar_...` today will be the same asset next release. Regressions
are republished as an additive commit; assets are only ever removed with a
version bump.

Issues / questions: open a Discussion on this dataset repo, or email
`hello@kaedim3d.com`.
