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
  - industrial
  - warehouse
task_categories:
  - robotics
size_categories:
  - n<1K
pretty_name: Industrial Parts & Packaging
---

# Industrial Parts & Packaging

> **Hosted on Hugging Face:** [projectsim/industrial-parts-and-packaging](https://huggingface.co/datasets/projectsim/industrial-parts-and-packaging/tree/main)

Procedurally generated, sim-ready 3D industrial infrastructure - warehouse
pallet racking bays and LPG / propane gas cylinders - shipped as OpenUSD
(`.usda`) with per-zone PBR maps, plus a static glTF (`.glb`) for tooling that
doesn't read USD. Every variant carries real per-link mass, inertia and
colliders authored from the class rig.

## Motivation

Industrial robotics pipelines - warehouse navigation, gas-bottle depot
handling, storage-shelf perception - need assets that (a) look real, (b)
sim-correctly in Isaac Sim / MuJoCo, and (c) come in enough parametric
variation that a policy sees the _distribution_ of shapes and materials, not
one shape re-textured. Public assets are either scanned singletons (no
variation, no reliable physics) or category-generic props (no PBR, guessed
masses). This dataset is our procedural pipeline's output: every variant has a
spec that produces watertight geometry, sim-authored physics, and PBR textures.

## What's included

| Class            | Count | Articulation                                  | Materials                                                                              |
| ---------------- | ----- | --------------------------------------------- | -------------------------------------------------------------------------------------- |
| `pallet_racking` | 50    | static (fixed rack bay between two uprights)  | uprights, beams, bracing, decking, footplates (painted-steel palette + bare-steel option) |
| `gas_cylinder`   | 50    | static (welded steel bottle, valve fixed)     | body (painted steel), collar_and_foot (painted steel), valve (brass), label (printed band) |

**Total: 100 variants across 2 classes, ~2.6 GB.**

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

- **`pallet_racking`** - one selective pallet-rack bay between two upright
  frames. Uprights 2.5-6 m tall, frame depth 1100 mm (EUR pallets) or 1200 mm,
  perforated cold-rolled 90 x 70 mm columns with horizontal + diagonal bracing;
  2-4 pairs of box-section step beams at 75 mm pitch with locking pins;
  optional wire-mesh or timber decking on beam levels; painted frames (blue
  uprights, orange beams typical) or a bare-steel option.
- **`gas_cylinder`** - welded steel LPG / propane bottle in real-world
  standard sizes driven by a size enum: 3 kg (Ø250 x 320 mm), 5 kg (Ø300 x
  400 mm), 7 kg (Ø300 x 540 mm), and 9 kg (Ø315 x 590 mm). Only surface
  details and paint colour vary within a size (grey, blue, red, green, orange
  typical). Fixed brass valve with handwheel.

The exact per-variant parameter values are not shipped in v1.

## Intended use

- Training and evaluating manipulation policies (gas-bottle grasp/carry/place,
  pallet loading onto racks)
- Perception (segmentation, category ID, pose estimation) from synthetic
  renders
- Domain-randomization sources for warehouse and depot robotics pipelines
- Sim-based experimentation before scanning real infrastructure
- Prototyping in Isaac Sim / MuJoCo / any USD-aware DCC

## Tested simulators

Verified end-to-end:

- **NVIDIA Isaac Sim 6.0.1** - loads `<vid>.usda` headless and GUI. Bottles
  stack and roll under gravity, rack bays hold a static-collider load.
- **MuJoCo 3.11.0** - USD is not native; use our sibling MJCF drop or the
  sim-ready `usd2mjcf` converter (matching physics for static props).
- **Blender 5.0** - USD and glTF import both work.

## Physical properties - authored vs estimated

| Property                 | How                                                                                                                | Notes                                                                                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Volume, COM, inertia     | Computed from watertight mesh                                                                                      | Falls back to the convex hull for open-shell parts                                                                                                    |
| Mass                     | Volume × density                                                                                                   | Density per zone, classified by a material-analysis VLM. The VLM inspects the object's renders together with the zone names in context — e.g. reading rack uprights as painted steel, cylinder bodies as painted steel, valves as brass, decking as timber — and looks each zone's density up in a shared material table (steel ~7850, brass ~8500, timber ~600, painted ABS ~1150 kg/m³). Per-link mass = mesh volume × classified density. |
| Collision shape          | Auto-classified per link                                                                                           | Author-time collision types per link, taken directly from the rig:<br/>• **primitives** (`box`, `sphere`, `cylinder`) where the rig is geometric — rack columns, beams, cylinder bodies<br/>• **`convexHull`** for sealed volumes with a single graspable shape (cylinder collar, valve body, footplates)<br/>• **`convexDecomposition`** with PhysX shrink-wrap for multi-part rigid props (rack braces + decking treated per member)<br/>• **`sdf`** (`sdfResolution 256`) for thin, non-convex shells where a decomposition would seal openings (wire-mesh decking) |
| Textures                 | 4K PBR maps per zone (photoreal retexturing on one half of the run; a curated painted-steel palette on the shader half) | Zone-mapped so a downstream engine can rebind materials per zone                                                                                      |
| `max_mass_kg`            | Cavity volume × water density                                                                                      | Meta-only hint for pick-and-place - **not** the physics mass. Gas cylinders report their advertised capacity by size enum. Rack bays report the pallet-payload rating.                                           |

## Limitations

- **Physics is in the `.usda`, not the `.glb`.** glTF has no physics
  equivalent - the `.glb` is a _visual_ copy (static pose, single mesh per
  link, no colliders). Use it for previews, tooling that can't read USD,
  Blender/Unity/three.js/model-viewer; use the `.usda` for simulation.
- Both classes are static props - no articulation is authored (valves are
  welded rigid, rack beams are locked into their columns).

## License

Files: **CC BY 4.0** - free to use, share, and adapt for any purpose,
including commercial use; attribution required.

## Citation

```bibtex
@dataset{kaedim_industrial_parts_and_packaging_2026,
  author = {{Kaedim} and {ProjectSim}},
  title = {Industrial Parts \& Packaging},
  year = {2026},
  publisher = {Hugging Face},
  version = {1.0},
  url = {https://huggingface.co/datasets/projectsim/industrial-parts-and-packaging}
}
```

## Maintenance

New classes and variants are added as separate top-level `assets/<class>/`
folders; the on-disk `<vid>` prefix is stable across future commits - a
`v03_pallet_racking_983fda` today will be the same asset next release.
Regressions are republished as an additive commit; assets are only ever
removed with a version bump.

Issues / questions: open a Discussion on this dataset repo, or email
`hello@kaedim3d.com`.
