# Shapespyer workshop notebooks

Four notebooks for a one-hour hands-on session with the `shapes.project` API. They are made for Google Colab. The same files run locally, in the uv environment of this folder.

| Notebook | GROMACS | Time | Content |
|---|---|---|---|
| `01_components.ipynb` | no | 5 min | library, `.show()`, a component from your own file, from SMILES |
| `02_structures.ipynb` | no | 10 min | ball, rod, ring, vesicle, lattice: assemble, export, view, download |
| `03_sample_model.ipynb` | yes | 10 min | asymmetric DOPC/POPC vesicle in TIP3 + NaCl, assembly with `gmx insert-molecules` / `gmx solvate`, export |
| `04_equilibration.ipynb` | yes | 15 min | SDS micelle project, stage 0 (minimisation), stage 1 in the background: status, log, live files, cancel; optional full stage 1 with trajectory view and micelle Rg (mdtraj); save, load, download for VMD |

Stage 1 of the equilibration takes about 12 min on 2 local threads and about 20 min on Colab. It is cancelled in the demo, and run to the end only if there is time (`RUN_STAGE_1`).

The notebooks come from `shapespyer/examples/projects/`. Repo-relative paths were replaced with library assets (`Lipid.load`, `AnalysisConfiguration.load("CHARMM36")`) or a download, so no clone is needed.

## Shapespyer version

The setup cell of every notebook and `pyproject.toml` install the same pin:

```
shapespyer @ git+https://gitlab.com/simnavi/shapespyer.git@v0.3.0
```

To use a workshop branch, change it in all five places to `@<branch>`, then run `uv lock --upgrade-package shapespyer`.

## Local test environment (WSL / Linux)

You need uv and GROMACS on `PATH`. On Ubuntu, `sudo apt install gromacs` installs it from the same source that Colab uses.

```bash
cd shapespyer-workshop
uv sync
# keep the library, Temp and Projects in .data/, away from ~/.local/share/Shapespyer
export XDG_DATA_HOME=$PWD/.data XDG_CONFIG_HOME=$PWD/.data
uv run jupyter lab
```

To run all notebooks without a browser (about 8 min). The executed copies go to `.runs/`:

```bash
for nb in 0*.ipynb; do uv run jupyter nbconvert --to notebook --execute --output-dir .runs "$nb"; done
```

## On Colab

1. Open <https://colab.research.google.com>, then **File → Upload notebook**. Or put the notebooks in Google Drive and share the folder.
2. Each notebook runs in its own runtime, so each one needs its setup cell. At the start of the session, ask everyone to open all four notebooks and run every setup cell. They run in parallel, about 1 min without GROMACS and about 3 min with it.
3. Files in `/content` are lost when the runtime disconnects. Download the zips before that.

On Colab, the setup cell sets the application directory to `/content/shapespyer`. Then `LocalAssets/`, `Temp/` and `Projects/` show in the Files panel. To keep them between sessions, mount Drive first and use a Drive folder instead:

```python
from google.colab import drive
drive.mount("/content/drive")
config.app_path = "/content/drive/MyDrive/shapespyer"
```

### Check before the workshop (could not be tested locally)

- `.show()` and the `nglview` views draw. nglview 4.0 replaces Colab's `notebook` and `ipywidgets` with newer versions, and pip prints warnings about `google-colab` pins. The views need `output.enable_custom_widget_manager()`, which the setup cell runs.
- 03 and 04 print `GROMACS: 2021.x ... | problem: None`. Colab is Ubuntu 22.04, so its apt GROMACS is older than the 2023.3 of Ubuntu 24.04.
- How long stage 0 takes.
- `files.download` works. The browser may block it the first time.

## Troubleshooting

| Message or symptom | Fix |
|---|---|
| `GROMACS: None \| problem: gmx not found` | The setup cell was skipped or apt failed. Run it again. |
| A molecule view stays empty | Run the setup cell again (widget manager), then the view cell. If that fails: **Runtime → Restart session**, then run all. |
| `Files of ... are being written by a run; wait for it to end or cancel it` | A background run is still going: `equilibration.job.cancel()`, or wait. |
| `Stage 0 hasn't finished, so stage 1 can't start` | Run stage 0 first. |
| `... was assembled again since stage 0 ran; run from stage 0` | The model changed. Set `first_stage = 0`. |
| `... is in use by this session ... Pass force=True to delete it anyway` | Assets of this session still use the files. Use `force=True`. |
| `Only N of M ... fit into the box` | Too much salt for the box. Lower the concentration. |
