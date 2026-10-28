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

## Where the files go

The first cell of each notebook is the setup cell. It hands `shapespyer.yaml` to Shapespyer's file manager before anything else runs. Then the whole session uses that file, and your own Shapespyer data and config are neither used nor changed.

- **Locally** the file says `app_path: .data`. The library goes to `.data/LocalAssets/`, the working files to `.data/Temp/` and the projects to `.data/Projects/`. Open the notebooks from this folder: if the setup cell can't find `shapespyer.yaml`, it stops with a message.
- **On Colab** the setup cell writes its own `shapespyer.yaml`, with everything in `/content/shapespyer`, so it shows in the Files panel. In `04`, tick `SAVE_PROJECTS_TO_DRIVE` to save the projects to `MyDrive/shapespyer/Projects` instead. Colab then asks once for Google sign-in. The library and Temp stay on the local disk, because GROMACS runs much slower on Drive.
- For a GROMACS that isn't on `PATH`, set `gmx_executable` in `shapespyer.yaml`.

## Shapespyer version

The setup cell of every notebook and `pyproject.toml` install the same pin, the release on PyPI:

```
shapespyer==0.3.0
```

To use a workshop branch instead, change it in all five places to `shapespyer @ git+https://gitlab.com/simnavi/shapespyer.git@<branch>`, then run `uv lock --upgrade-package shapespyer`. On Windows, a git install also needs `git config --global core.longpaths true`, because the shapespyer repo has paths longer than Windows allows by default.

## Run locally

| | 01, 02 | 03, 04 |
|---|---|---|
| Windows | yes | only with a Windows GROMACS set as `gmx_executable`; otherwise the setup cell stops and says so |
| WSL / Linux | yes | yes, after `sudo apt install gromacs` (Colab installs GROMACS from apt too) |

```bash
cd shapespyer-workshop
uv sync           # Python 3.12, from .python-version, the same as Colab
uv run jupyter lab
```

In VS Code, open the folder (from WSL: **WSL: Open Folder in WSL**) and pick the `.venv` kernel. For the molecule views, allow VS Code to load widget scripts from a CDN when it asks (setting `jupyter.widgetScriptSources`).

`.venv` belongs to the system that made it. A `uv sync` from Windows replaces a `.venv` made in WSL, and the other way round. To use both, clone the repo twice. For WSL, clone it in the Linux home, such as `~/shapespyer-workshop`, because file access on `/mnt/c` is slow from WSL.

To run the notebooks without a browser, about 8 min for all four. The executed copies go to `.runs/`:

```bash
for nb in 0*.ipynb; do uv run jupyter nbconvert --to notebook --execute --output-dir .runs "$nb"; done
```

```powershell
foreach ($nb in "01_components.ipynb", "02_structures.ipynb") { uv run jupyter nbconvert --to notebook --execute --output-dir .runs $nb }
```

Commit the notebooks without outputs: `uv run jupyter nbconvert --clear-output --inplace 0*.ipynb`.

## On Colab

1. Open <https://colab.research.google.com>, then **File → Upload notebook**. Or put the notebooks in Google Drive and share the folder.
2. Each notebook runs in its own runtime, so each one needs its setup cell. At the start of the session, ask everyone to open all four notebooks and run every setup cell. They run in parallel, about 1 min without GROMACS and about 3 min with it.
3. Files in `/content` are lost when the runtime disconnects. Download the zips before that, or tick `SAVE_PROJECTS_TO_DRIVE` in `04`.

### Check before the workshop (could not be tested locally)

- `.show()` and the `nglview` views draw. nglview 4.0 replaces Colab's `notebook` and `ipywidgets` with newer versions, and pip prints warnings about `google-colab` pins. The views need `output.enable_custom_widget_manager()`, which the setup cell runs.
- 03 and 04 print `GROMACS: 2021.x`. Colab is Ubuntu 22.04, so its apt GROMACS is older than the 2023.3 of Ubuntu 24.04.
- `SAVE_PROJECTS_TO_DRIVE`: the project folder appears in `MyDrive/shapespyer/Projects`, and `Project.load` opens it in a new runtime.
- How long stage 0 takes.
- `files.download` works. The browser may block it the first time.

## Troubleshooting

| Message or symptom | Fix |
|---|---|
| `No shapespyer.yaml in ...; open the notebook from the shapespyer-workshop folder` | The notebook runs from another folder. Open it from this folder. |
| `This notebook needs GROMACS: gmx not found ...` | On Colab, run the setup cell again (apt failed). On Windows, use WSL or Colab, or set `gmx_executable`. |
| `Shapespyer already runs with other folders: Runtime > Restart session ...` | You changed `SAVE_PROJECTS_TO_DRIVE` after Shapespyer started. Restart the session, then run the setup cell. |
| A molecule view stays empty | Run the setup cell again (widget manager), then the view cell. If that fails: **Runtime → Restart session**, then run all. |
| `Files of ... are being written by a run; wait for it to end or cancel it` | A background run is still going: `equilibration.job.cancel()`, or wait. |
| `Stage 0 hasn't finished, so stage 1 can't start` | Run stage 0 first. |
| `... was assembled again since stage 0 ran; run from stage 0` | The model changed. Set `first_stage = 0`. |
| `... is in use by this session ... Pass force=True to delete it anyway` | Assets of this session still use the files. Use `force=True`. |
| `Only N of M ... fit into the box` | Too much salt for the box. Lower the concentration. |
| `Asset with the name ... could not be downloaded`, with `429 Too Many Requests` in the warning above | GitLab limits downloads of the library files. Wait a minute, then run the cell again. |
| Windows: `FileNotFoundError` for a file in `.data\Temp\...\assembly.tmp\` (the lattice makes the deepest paths) | The path is longer than 260 characters. Move the folder closer to the drive root, or turn on long paths in an admin PowerShell: `New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name LongPathsEnabled -Value 1 -PropertyType DWORD -Force` |
