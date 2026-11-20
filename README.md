# Shapespyer workshop notebooks

The notebooks of the Shapespyer workshop. In `notebooks/`, each author has a folder and arranges it freely; copies of the folders also go to the examples of the Shapespyer repository. This repo collects them with one environment, so you can install them on their own and keep using them after the workshop.

| Path | Content |
|---|---|
| `notebooks/00_SHAPESPYER_INTRO.ipynb` | an introduction to Shapespyer, to read first |
| `notebooks/AB_GMX_LIPIDS_WORKFLOW/` | `GMX_LIPID_VESICLE.ipynb`: from SMILES to a multicomponent lipid vesicle, with GROMACS. With its inputs (`inp/`), topologies (`toppar/`), scripts and reference results (`usr-ref.tar.xz`) |
| `notebooks/AK_PROJECTS/` | four notebooks on the `shapes.project` API, see [below](#ak_projects-the-shapesproject-api) |

## Install and run locally

```bash
cd shapespyer-workshop
uv sync           # Python 3.12, from .python-version; Shapespyer 0.3.1
uv run jupyter lab
```

In VS Code, open the folder (from WSL: **WSL: Open Folder in WSL**) and pick the `.venv` kernel. For the molecule views, allow VS Code to load widget scripts from a CDN when it asks (setting `jupyter.widgetScriptSources`).

The notebooks with GROMACS need it installed: on WSL / Linux, `sudo apt install gromacs`. `GMX_LIPID_VESICLE.ipynb` says what else it needs.

## AK_PROJECTS: the `shapes.project` API

Four notebooks for a one-hour hands-on session with the `shapes.project` API. They run in the uv environment of this repo.

| Notebook | GROMACS | Time | Content |
|---|---|---|---|
| `01_components.ipynb` | no | 10 min | library, `.show()`, a component from your own file, from SMILES; save both to the library (one new, one replacing a library asset) |
| `02_structures.ipynb` | no | 10 min | ring, rod, bilayer, ball, vesicle, lattice: assemble, export, view |
| `03_sample_model.ipynb` | yes | 10 min | asymmetric DOPC/POPC vesicle in TIP3 + NaCl, assembly with `gmx insert-molecules` / `gmx solvate`, export |
| `04_equilibration.ipynb` | yes | 15 min | SDS micelle project, stage settings, stage 0 (minimisation); optional stage 1 (short MD, about 20 min on 2 CPU cores) with trajectory view and micelle Rg (mdtraj); save and load the project; export the stages for VMD |

Each notebook starts with a few setup cells:

1. the setup cell, which checks that `shapespyer.yaml` is next to the notebook;
2. the helpers cell: the `Helpers` class, with small functions that show results. The workflow cells always call them as `Helpers.<name>()`, so they can't be taken for part of Shapespyer;
3. the config cell, which passes `shapespyer.yaml` to Shapespyer and sets the log level and the work folder.

Then come the imports of the workflow, and the library cell, which gives the library its folder. Each import sits in the first cell that needs it, such as `shutil` in the clean up.

### Where the files go

The config cell of each notebook hands the `shapespyer.yaml` config file to Shapespyer's file manager before any other Shapespyer call. Then the whole session uses that file, and your own Shapespyer data and config are neither used nor changed. The library cell gives the library its folder explicitly, `LocalAssets/` in `app_path`, which is also the default.

- `notebooks/AK_PROJECTS/shapespyer.yaml` says `app_path: ../../.data`, which is `.data/` in the repo root. The library goes to `.data/LocalAssets/`, the working files to `.data/Temp/` and the projects to `.data/Projects/`. Exports of the notebooks go to `.data/work/`, which the clean up cell of each notebook deletes. Keep `shapespyer.yaml` next to the notebooks: if the setup cell can't find it in the notebook's folder, it stops with a message.
- For a GROMACS that isn't on `PATH`, set `gmx_executable` in `notebooks/AK_PROJECTS/shapespyer.yaml`.

### Which notebooks run where

| | 01, 02 | 03, 04 |
|---|---|---|
| Windows | yes | only with a Windows GROMACS set as `gmx_executable`; otherwise the setup cell stops and says so |
| WSL / Linux | yes | yes, after `sudo apt install gromacs` |

### Troubleshooting

| Message or symptom | Fix |
|---|---|
| `No shapespyer.yaml in ...; run the notebook in its own folder, next to shapespyer.yaml` | The notebook runs in another folder than its own. In VS Code, check the setting `jupyter.notebookFileRoot`; its default `${fileDirname}` is right. |
| `This notebook needs GROMACS: gmx not found ...` | Install GROMACS (on WSL / Linux: `sudo apt install gromacs`). On Windows, use WSL, or set `gmx_executable`. |
| A molecule view stays empty | In VS Code, allow widget scripts from a CDN (setting `jupyter.widgetScriptSources`), then run the view cell again. If that fails, restart the kernel and run all. |
| `Stage 0 hasn't finished, so stage 1 can't start` | Run stage 0 first. |
| `... was assembled again since stage 0 ran; run from stage 0` | The model changed. Set `first_stage = 0`. |
| `... is in use by this session ... Pass force=True to delete it anyway` | Assets of this session still use the files. Use `force=True`. |
| `Only N of M ... fit into the box` | Too much salt for the box. Lower the concentration. |
| `Layer 1 of ... needs an rmin of at least ...` (or `The inner leaflet of lamella 1 ...`) | A layer's or leaflet's `rmin` must be at least 1.6 times its `dmin`. Raise the `rmin` or lower the `dmin`. |
| `Asset with the name ... could not be downloaded`, with `429 Too Many Requests` in the warning above | GitLab limits downloads of the library files. Wait a minute, then run the cell again. |
| Windows: `FileNotFoundError` for a file in `.data\Temp\...\assembly.tmp\` (the lattice makes the deepest paths) | The path is longer than 260 characters. Move the folder closer to the drive root, or turn on long paths in an admin PowerShell: `New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name LongPathsEnabled -Value 1 -PropertyType DWORD -Force` |
