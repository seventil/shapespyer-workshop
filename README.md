# Shapespyer workshop notebooks

Four notebooks, in `notebooks/`, for a one-hour hands-on session with the `shapes.project` API. They are made for Google Colab. The same files run locally, in the uv environment of this folder.

| Notebook | GROMACS | Time | Content |
|---|---|---|---|
| `01_components.ipynb` | no | 10 min | library, `.show()`, a component from your own file, from SMILES; save both to the library (one new, one replacing a library asset) |
| `02_structures.ipynb` | no | 10 min | ball, rod, ring, vesicle, lattice: assemble, export, view, download |
| `03_sample_model.ipynb` | yes | 10 min | asymmetric DOPC/POPC vesicle in TIP3 + NaCl, assembly with `gmx insert-molecules` / `gmx solvate`, export |
| `04_equilibration.ipynb` | yes | 15 min | SDS micelle project, stage 0 (minimisation), stage 1 in the background: status, log, live files, cancel; optional full stage 1 with trajectory view and micelle Rg (mdtraj); save, load, download for VMD |

## Where the files go

The first cell of each notebook is the setup cell. It hands `shapespyer.yaml` config file to Shapespyer's file manager before anything else runs. Then the whole session uses that file, and your own Shapespyer data and config are neither used nor changed.

- **Locally** `notebooks/shapespyer.yaml` says `app_path: ../.data`, which is `.data/` in the repo root. The library goes to `.data/LocalAssets/`, the working files to `.data/Temp/` and the projects to `.data/Projects/`. Keep `shapespyer.yaml` next to the notebooks: if the setup cell can't find it in the notebook's folder, it stops with a message.
- **On Colab** the setup cell writes its own `shapespyer.yaml`, with everything in `/content/shapespyer`, so it shows in the Files panel. In `04`, tick `SAVE_PROJECTS_TO_DRIVE` to save the projects to `MyDrive/shapespyer/Projects` instead. Colab then asks once for Google sign-in. The library and Temp stay on the local disk, because GROMACS runs much slower on Drive.
- For a GROMACS that isn't on `PATH`, set `gmx_executable` in `notebooks/shapespyer.yaml`.

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

## On Colab

1. Open <https://colab.research.google.com>, then **File → Upload notebook**. Or put the notebooks in Google Drive and share the folder.
2. Each notebook runs in its own runtime, so each one needs its setup cell. They run in parallel, about 1 min without GROMACS and about 3 min with it.
3. Files in `/content` are lost when the runtime disconnects. Download the zips before that, or tick `SAVE_PROJECTS_TO_DRIVE` in `04`.

## Troubleshooting

| Message or symptom | Fix |
|---|---|
| `No shapespyer.yaml in ...; run the notebook in its own folder, next to shapespyer.yaml` | The notebook runs in another folder than its own. In VS Code, check the setting `jupyter.notebookFileRoot`; its default `${fileDirname}` is right. |
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
