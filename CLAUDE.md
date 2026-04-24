# CLAUDE.md — Metashape Scripts

## Project Overview

A collection of Python utility scripts for **Agisoft Metashape Pro**, a photogrammetry application. Scripts extend Metashape functionality and can run automatically on application launch.

**Metashape API version target:** 2.2+

## Repository Structure

```
metashape-scripts/
├── src/                    # Main scripts (run directly in Metashape)
│   ├── contrib/            # Community-contributed scripts
│   └── samples/            # Example workflow scripts
├── misc/                   # Utilities and dependency links
│   ├── generate_metashape_stub_file.py   # Generates type stubs for IDE support
│   └── links.txt           # URLs for required libraries (GDAL, rasterio, etc.)
├── README.md
└── LICENSE                 # MIT
```

## How Scripts Work

Scripts are loaded by Metashape at startup or run manually via **Tools → Run Script**. They use the `Metashape` Python module (bundled with the application).

**Version guard pattern** — every script must check compatibility:
```python
import Metashape

compatible_major_version = "2.2"
found_major_version = ".".join(Metashape.app.version.split(".")[:2])
if found_major_version != compatible_major_version:
    raise Exception("Incompatible Metashape version: {} != {}".format(
        found_major_version, compatible_major_version))
```

**Menu registration pattern** — scripts that add GUI items register a label and callback:
```python
def someFunction():
    ...

Metashape.app.addMenuItem("Custom/Do Something", someFunction)
```

**Dialog scripts** use PySide2 for UI widgets and follow the `_dialog.py` naming suffix convention.

## Development Guidelines

### Adding a New Script

1. Place general-purpose scripts in `src/`.
2. Place community contributions in `src/contrib/`.
3. Always include the version guard at the top.
4. Register a menu item via `Metashape.app.addMenuItem` if the script is interactive.
5. Keep scripts self-contained — no shared utility modules.

### Code Style

- Python 3, no type hints required.
- No external dependencies unless documented in `misc/links.txt`.
- ML/vision scripts may use `numpy`, `cv2`, `deepforest`, `rasterio` — check `misc/links.txt` for install sources.
- PySide2 for any GUI dialogs.

### Key Metashape API Objects

| Object | Purpose |
|--------|---------|
| `Metashape.app` | Application handle (menus, dialogs, version) |
| `Metashape.app.document` | Current project document |
| `doc.chunks` | List of processing chunks |
| `chunk.cameras` | Camera list with reference/pose data |
| `chunk.point_cloud` | Sparse point cloud (tie points) |
| `chunk.model` | 3D mesh model |
| `chunk.orthomosaic` | Orthomosaic raster |
| `chunk.shapes` | Vector shape layers |

### Running / Testing

Metashape Pro must be installed to run scripts — there is no standalone test harness. To test a script:

```bash
# Launch Metashape and run script from console
metashape.exe -r src/my_script.py
```

Or place the script in the Metashape auto-start folder (see [Agisoft KB article](https://agisoft.freshdesk.com/support/solutions/articles/31000133123)).

Type stubs for IDE autocompletion can be generated with:
```bash
python misc/generate_metashape_stub_file.py
```

## Contributing

- New scripts go in `src/contrib/` via pull request.
- Each script must be standalone and include the version guard.
- No modifications to existing scripts needed for new contributions.
