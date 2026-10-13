# MAA → Fairbeam JSON Converter

**Batch converter for MMANA-GAL / MMANA antenna models (`.maa`) to Fairbeam designs (`.design.json`)**

**Version:** 1.2.2 — Port Topology Guard  
**Author:** UA1CFM · © 2026  
**Platforms:** Windows 10/11 (GUI / portable EXE); Python 3.9+ (CLI, where supported)  
**Languages:** [English](README.md) · [Русский](README_RU.md)

## Overview

MAA → Fairbeam JSON Converter transfers antenna geometry and supported electrical elements from MMANA-format files into Fairbeam's `fairbeam.design/1` JSON format. It can process entire directory trees in one run, preserving subfolder structure and producing a detailed conversion report.

It is designed for radio amateurs and antenna developers who maintain collections of MMANA models and want to inspect and further develop them in Fairbeam/openEMS.

**Important:** A valid JSON file is **not** necessarily an electromagnetically equivalent MMANA model. The converter explicitly flags sources and loads that cannot be exported safely. Always check the generated design and validate its port coupling in Fairbeam before trusting S11, impedance, efficiency, or radiation patterns.

## Main features

- **Recursive batch conversion:** `.maa` → `.design.json`, preserving the original input directory hierarchy.
- **Desktop GUI:** choose source and output folders, monitor progress, review warnings, and stop after the current file.
- **Command-line mode** for scripting and repeated conversion jobs.
- **Wire geometry:** exports conductor coordinates and individual wire radii; retains conductor thickness differences present in MMANA models.
- **Frequency setup:** converts MHz to GHz and generates a frequency sweep around the source model's design frequency.
- **Ports:** exports supported axis-aligned sources as real gaps with line-type lumped ports; adjustable reference impedance (50 Ω by default).
- **Port safety checks:** detects a number of cases where another conductor overlaps or approaches the proposed feed gap; refuses unsafe mappings instead of silently generating shorted ports.
- **Loads:** supports eligible R, L, C and RLC representations, omitting zero-valued component fields to satisfy Fairbeam validation. Finite-Q handling uses a design-frequency approximation, not a verified broadband loss model.
- **Diagnostics:** preserves original sources, loads, conversion warnings and readiness flags in `_maa_conversion` metadata; writes `conversion_report.csv`.
- **Windows build script:** `BUILD_EXE_WINDOWS.bat` builds a standalone executable with PyInstaller.

## Quick start — GUI on Windows

1. Download and extract the release ZIP.
2. To run from source, install **Python 3.9+ with Tkinter** and launch `maa_to_fairbeam.py`.
3. Or run **`BUILD_EXE_WINDOWS.bat`** to install the build dependencies, execute tests, and create `Windows_EXE/MAA_to_Fairbeam_JSON.exe`.
4. Select the folder containing your `.maa` files and a **different** output folder.
5. Select `.design.json`, choose overwrite behavior, then click **Convert all MAA → JSON**.
6. Inspect `conversion_report.csv`; open selected JSON projects in Fairbeam and check port/load placement, mesh, S11 and the run quality status.

The EXE build requires Windows, a working Python installation (or supported automatic setup), and internet access to install PyInstaller. **The source ZIP is not a precompiled EXE.**

## Command-line usage

```powershell
python maa_to_fairbeam.py --input "C:\Antennas\MAA" --output "C:\Antennas\Fairbeam"
```

Optional arguments:

```text
--suffix .design.json       Output extension (default)
--overwrite                Replace existing JSON files
--port-r 50                Port reference impedance in ohms
--radius-zero-mm 0.5       Fallback radius for zero-radius model wires
--sweep-percent 5          Sweep range ±5% around model frequency
```

Run `python maa_to_fairbeam.py --help` for the full option list. The program preserves relative subfolder paths and records conversion outcomes in a CSV report.

## Output layout

```text
MAA_collection/
  VHF/
    Yagi.maa
  Loops/
    Loop.maa

Fairbeam_collection/
  VHF/
    Yagi.design.json
  Loops/
    Loop.design.json
  conversion_report.csv
```

Each generated design contains Fairbeam geometry and simulation settings plus an `_maa_conversion` section containing the source model's original electrical records and export diagnostics.

## Current limitations — please read

- MMANA pulse references (`WnB`, `WnC`, `WnE` and offsets), particularly with automatic segmentation, **are not universally mapped** to an equivalent Fairbeam component location.
- Diagonal, embedded, intersecting and multiply connected feed structures may require **manual reconstruction**; the converter intentionally rejects unsafe port placement.
- Multi-source/phased excitation, ground models, stack arrangements and some matching networks are **not fully equivalent** in Fairbeam.
- A `simulation_ready` value or clean JSON validation **does not prove** that a port couples after openEMS meshing. `Port not coupled` must be checked in actual Fairbeam simulation results.
- Small conductor details can require very fine FDTD meshes; inspect mesh warnings and convergence.
- When a model cannot be transferred electrically, its geometry may still be exported with explanatory warnings. **Do not present geometry-only results as a validated antenna simulation.**

### Regression testing

Version 1.2.2 was batch-tested on a corpus of **1,713 MAA files**, with **1,713 JSON outputs generated** and **30 automated converter tests passed** in the development run. These numbers measure conversion completion and local checks, **not** 1,713 successful Fairbeam/openEMS simulations. The bundled `REGRESSION_1713_V122.csv` contains individual outcomes; models with missing feeds or loads remain identified.

## Repository contents

| File | Purpose |
|---|---|
| `maa_to_fairbeam.py` | Converter, GUI, CLI and batch processing |
| `BUILD_EXE_WINDOWS.bat` | Build the portable Windows executable |
| `test_converter.py`, `test_pulse_positions.py`, `test_zero_omission.py` | Automated tests |
| `REGRESSION_1713_V122.csv`, `SUMMARY_V122.json` | Batch regression results |
| Sample `.MAA` and `.design.json` files | Geometry and conversion reference cases |

## Development direction

The priority is **topology-aware MMANA pulse reconstruction**, reliable source/load mapping (including automatic segmentation and complex feed networks), and automated validation against Fairbeam's actual openEMS mesh and excitation outcomes. Fixes must preserve the antenna's electrical network, not merely make a project pass static checks.

## Credits and attribution

Developed by **UA1CFM © 2026**. Independent conversion utility; not an official MMANA-GAL or Fairbeam component. Fairbeam and MMANA-GAL belong to their respective authors and maintainers. See [Fairbeam on GitHub](https://github.com/ismailakdag/fairbeam).

**License:** No license is specified in this package. Add an explicit `LICENSE` file before publishing if you intend to grant others rights to use, modify or redistribute the source.
