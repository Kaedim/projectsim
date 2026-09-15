# ProjectSim Datasets

Index of Kaedim / ProjectSim's procedurally generated, sim-ready 3D asset
datasets. Content is hosted on Hugging Face; this repo aggregates the READMEs
and gives you one place to browse the scope.

**700 sim-ready variants across 11 classes**, every variant shipping
`.usda + .glb + .xml` (OpenUSD, glTF, MJCF), 4K PBR textures, sim-authored
physics (mass / inertia / colliders), and articulation where the class defines
it. All datasets are **CC BY 4.0** and publicly readable on Hugging Face.

## Datasets

| Dataset                                                                          | Classes                                     | Variants | Total size | HF link                                                                                                                        |
| -------------------------------------------------------------------------------- | ------------------------------------------- | -------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------ |
| [Warehouse Manipulation & States](warehouse-manipulation-and-states/README.md)   | cardboard_box, pallet_boxes, logistics_cage | 225      | ~12 GB     | [projectsim/warehouse-manipulation-and-states](https://huggingface.co/datasets/projectsim/warehouse-manipulation-and-states)   |
| [Articulated Storage & Appliances](articulated-storage-and-appliances/README.md) | desk_with_drawers, oven, wardrobe           | 225      | ~12 GB     | [projectsim/articulated-storage-and-appliances](https://huggingface.co/datasets/projectsim/articulated-storage-and-appliances) |
| [Industrial Parts & Packaging](industrial-parts-and-packaging/README.md)         | pallet_racking, gas_cylinder                | 100      | ~2.6 GB    | [projectsim/industrial-parts-and-packaging](https://huggingface.co/datasets/projectsim/industrial-parts-and-packaging)         |
| [Bottles & Containers](bottles-and-containers/README.md)                         | beverage_can, glass_jar, mug                | 149      | ~3.1 GB    | [projectsim/bottles-and-containers](https://huggingface.co/datasets/projectsim/bottles-and-containers)                         |

## Per-variant contents (uniform across all four datasets)

```
assets/<class>/<vid>/
├── <vid>.usda            OpenUSD, textures referenced as ./textures/*.png
├── <vid>.glb             glTF binary, embedded textures, static pose
├── textures/*.png        PBR maps (base_color, normal, roughness, metallic)
└── mjcf/
    ├── <vid>.xml         MJCF, drop-in for MuJoCo `simulate`
    ├── visuals/*.obj     per-zone visual meshes, textured
    ├── visuals/textures/ PBR maps referenced by the MJCF
    └── collision/*.obj   per-component convex hulls (or primitives, per authored collision type)
```

## Interactive viewer

An in-browser 3D playground for the articulated classes is live at
**[projectsim/articulated-lab](https://huggingface.co/spaces/projectsim/articulated-lab)** —
pick a class tab, pick an item, orbit it, and drag any flap / drawer / door /
wheel joint. Works from any modern browser, no build step, no backend.

## Sim-authored physics summary

- **Mass, COM, inertia** from watertight mesh volumes; density per zone
  classified by a material-analysis VLM (steel / wood / kraft / rubber /
  ceramic / glass / MDF / etc.).
- **Collision shape** per link matches the authoring intent:
  primitives (`box`, `sphere`) where the rig is geometric; `convexHull` for
  sealed graspable volumes; `convexDecomposition` (PhysX shrink-wrap) for
  multi-part rigid props; `sdf` (`sdfResolution 256`) for thin non-convex
  shells. MJCF ships the same shapes: primitives verbatim, per-connected-
  component convex hulls otherwise.
- **Joints** with explicit degrees / meters limits, damping, and friction
  latches (cardboard flaps 0.01 N·m soft latch, appliance doors 8 N·m catch).
- **`max_mass_kg`** = cavity volume × water density — a pick-and-place hint,
  **not** the physics mass.

## Tested simulators

- **NVIDIA Isaac Sim 6.0.1** — loads `<vid>.usda` headless and GUI.
- **MuJoCo 3.11.0** — loads `mjcf/<vid>.xml` directly (`simulate mjcf/<vid>.xml`).
- **Blender 5.0** — USD and glTF import both work.

## License

Files: **CC BY 4.0** — free to use, share, and adapt for any purpose,
including commercial use; attribution required.

## Citation

```bibtex
@dataset{kaedim_projectsim_datasets_2026,
  author = {{Kaedim} and {ProjectSim}},
  title = {ProjectSim Datasets},
  year = {2026},
  publisher = {Hugging Face},
  version = {1.0},
  url = {https://huggingface.co/projectsim}
}
```

## Maintenance

New classes and variants are added as additive commits on the Hugging Face
side; the on-disk `<vid>` prefix is stable across future commits — a
`v03_pallet_boxes_6c1b6d` today will be the same asset next release.
Regressions are republished as an additive commit; assets are only ever
removed with a version bump.

Issues / questions: open a discussion on any of the linked Hugging Face
datasets, or email `hello@kaedim3d.com`.
