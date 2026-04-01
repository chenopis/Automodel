# Walkthrough Log: Install NeMo Automodel

**Date**: 2026-04-01
**Tester**: DORI agent
**Doc page**: `docs/guides/installation.md`
**Published URL**: https://docs.nvidia.com/nemo/automodel/latest/guides/installation.html
**Focus**: All installation methods (PyPI, Docker, GitHub source, editable, Docker mount, extras)
**Instance**: `automodel-install-test` (MassedCompute 1x A6000 48GB, $0.68/hr, Des Moines)
**Org**: `docs-andrewch`
**Model**: N/A (installation guide, no model under test)

---

## Entries

### Entry 1: T1 -- Install via PyPI

| Field | Value |
|---|---|
| **doc_section** | Install via PyPI (Recommended) |
| **doc_anchor** | `install-via-pypi-recommended` |
| **command** | `pip3 install nemo-automodel` |
| **expected** | Installs latest stable release |
| **actual** | Installed `nemo-automodel-0.3.0` with torch-2.10.0, transformers-5.4.0. PATH warnings for `/home/shadeform/.local/bin`. |
| **verdict** | PASS |
| **duration** | 269s |
| **finding** | PATH warnings: scripts (`automodel`, `transformers`, etc.) installed to `~/.local/bin` which is not on PATH. Doc does not mention this or suggest adding to PATH. |

**Verification** (doc tip):
```
$ python -c "import nemo_automodel; print(nemo_automodel.__version__)"
0.3.0
```
Verification command works as documented.

### Entry 2: T2 -- Editable Install (Developer Mode)

| Field | Value |
|---|---|
| **doc_section** | Install in Developer Mode (Editable Install) |
| **doc_anchor** | `install-in-developer-mode-editable-install` |
| **command** | `cd Automodel && pip3 install -e .` |
| **expected** | Installs in editable mode, changes reflected immediately |
| **actual** | ERROR: build backend missing `build_editable` hook. Cannot install in editable mode. |
| **verdict** | FAIL |
| **duration** | 10s |
| **finding** | **Doc bug**: `pip3 install -e .` does not work. The project's `pyproject.toml` build backend does not support PEP 660 editable installs. Either the build backend needs updating, or the doc should provide a workaround (e.g., `pip3 install -e . --no-build-isolation`, or use `uv pip install -e .`). |

### Entry 3: T3 -- Install from GitHub Source

| Field | Value |
|---|---|
| **doc_section** | Install via GitHub (Source), Option A |
| **doc_anchor** | `install-via-github-source` |
| **command** | `pip3 install git+https://github.com/NVIDIA-NeMo/Automodel.git` |
| **expected** | Installs latest code from main branch |
| **actual** | Installed `nemo-automodel-0.3.0` from commit `fedc9d2b`. Resolved, built, and installed successfully in 19s. |
| **verdict** | PASS |
| **duration** | 19s |
| **finding** | Works as documented. Clone + build is fast (~19s vs ~270s for PyPI) since heavy deps were already installed. |

### Entry 4: T5 -- Install Extras (CLI + VLM)

| Field | Value |
|---|---|
| **doc_section** | Bonus: Install Extras |
| **doc_anchor** | `bonus-install-extras` |
| **command** | `pip3 install nemo-automodel[cli]` then `pip3 install nemo-automodel[vlm]` |
| **expected** | CLI and VLM extras install additional dependencies |
| **actual** | Both installed successfully. CLI: already satisfied (deps overlap with base). VLM: installed 33 additional packages (torchvision, timm, open-clip-torch, librosa, decord, etc.). |
| **verdict** | PASS |
| **duration** | 53s total |
| **finding** | Works as documented. Same PATH warning pattern for `mistral_common`. VLM extra pulls significant additional dependencies. |

### Entry 5: T4 -- Install via NeMo Docker Container

| Field | Value |
|---|---|
| **doc_section** | Install via NeMo Docker Container |
| **doc_anchor** | `install-via-nemo-docker-container` |
| **command** | `docker pull nvcr.io/nvidia/nemo-automodel:25.11.00` then `docker run --gpus all --rm --shm-size=8g ... python3 -c "import nemo_automodel; print(...)"` |
| **expected** | Pulls container and runs with GPU support |
| **actual** | Pull succeeded (315s). Container runs and imports `nemo_automodel` successfully. Version inside container: `0.2.0rc0`. |
| **verdict** | PASS (with finding) |
| **duration** | 315s pull + 22s run |
| **finding** | **Version mismatch**: Container `25.11.00` ships `nemo_automodel==0.2.0rc0` while PyPI has `0.3.0`. Doc does not mention this discrepancy. Also: `pynvml` deprecation warning displayed during import (cosmetic). |
