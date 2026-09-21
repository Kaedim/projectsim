# Try a sample asset

Explore an articulated desk, download its files, and load it into your simulator.
**No projectsim API token is required for this guide.**

## 1. Preview the assets

Open the [interactive demo](https://huggingface.co/spaces/projectsim/lab).
Choose an articulated object, orbit it, and inspect its drawers, doors, or other
moving parts. This is a visual preview; use a simulator to evaluate physics.

## 2. Download one complete asset

Start with desk `v01_desk_with_drawers_147755` from
[Articulated Storage & Appliances](https://huggingface.co/datasets/projectsim/articulated-storage-and-appliances/tree/main/assets/desk_with_drawers/v01_desk_with_drawers_147755).
The example downloads only that asset's folder, including meshes and textures,
rather than the full dataset. The folder is approximately 86 MB at the time of writing.

Install the Hugging Face download client in your Python environment:

```bash
python -m pip install huggingface_hub
```

Save this as `download_sample.py` and run `python download_sample.py`:

```python
from pathlib import Path

from huggingface_hub import snapshot_download

asset_id = "v01_desk_with_drawers_147755"
asset_folder = f"assets/desk_with_drawers/{asset_id}"

download_dir = snapshot_download(
    repo_id="projectsim/articulated-storage-and-appliances",
    repo_type="dataset",
    allow_patterns=[f"{asset_folder}/**"],
    local_dir="projectsim-sample",
    token=False,
)

asset_dir = Path(download_dir).resolve() / asset_folder
print(f"MuJoCo: {asset_dir / 'mjcf' / (asset_id + '.xml')}")
print(f"Isaac Sim: {asset_dir / (asset_id + '.usda')}")
```

Keep the downloaded folder structure intact. The simulator files reference
meshes and textures within it. See the
[Hugging Face download documentation](https://huggingface.co/docs/huggingface_hub/guides/download)
for the download client's options.

## 3. Load it in your simulator

### MuJoCo

Install MuJoCo, then launch its viewer from the same directory where you ran
the download script:

```bash
python -m pip install mujoco
python -m mujoco.viewer --mjcf=projectsim-sample/assets/desk_with_drawers/v01_desk_with_drawers_147755/mjcf/v01_desk_with_drawers_147755.xml
```

Inspect the desk and use the viewer's joint controls to move its drawers.
Check the geometry, range of motion, and collisions against the needs of your
manipulation task. See [MuJoCo's viewer documentation](https://mujoco.readthedocs.io/en/stable/python.html#standalone-app)
for viewer usage.

### Isaac Sim

In an existing Isaac Sim installation, open the `.usda` file printed by the
download script. Inspect its articulation, collision geometry, and materials
before adding it to your robot's test scene. Keep the accompanying textures
alongside the USD file.

The `.glb` is a static visual preview. Use the `.xml` or `.usda` for the
simulation's articulation and physics. See the
[dataset guide](../articulated-storage-and-appliances/README.md) for the
authored physics, limitations, license, and attribution requirements.

## 4. Generate assets for your own task

Need different objects or more variations? The projectsim API lets you create
assets from your own inputs. For rigid or articulated object families, explore
[Procedural variation sets](https://docs.projectsim.ai/creating-assets/procedural-variation-set).
Use [Choosing an endpoint](https://docs.projectsim.ai/creating-assets/choosing-an-endpoint)
to compare the other creation workflows.

**[Request API access →](https://www.projectsim.ai/access)**

Tell us what you are simulating, which simulator you use, and the assets you
need. Our team will help you get started and provide your API token.
Self-serve signup is not available today.

Once you receive a token, follow the
[API quickstart](https://docs.projectsim.ai/getting-started/quickstart).
You can continue using the public datasets without API access.
