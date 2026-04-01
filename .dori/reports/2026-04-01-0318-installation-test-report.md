# Test Report: Automodel Installation Guide

**Date**: 2026-04-01
**Branch**: `main`
**Doc page**: `docs/guides/installation.md`
**Published URL**: https://docs.nvidia.com/nemo/automodel/latest/guides/installation.html
**Instance**: `automodel-install-test` (MassedCompute 1x A6000 48GB, $0.68/hr)
**Org**: `docs-andrewch`

---

## Scope

All installation methods documented in `docs/guides/installation.md`:
- T1: Install via PyPI (Recommended)
- T2: Install in Developer Mode (Editable Install)
- T3: Install via GitHub (Source)
- T4: Install via NeMo Docker Container
- T5: Install Extras (CLI, VLM)

## Environment

| Component | Value |
|---|---|
| **OS** | Ubuntu 22.04 (6.8.0-90-generic) |
| **GPU** | NVIDIA RTX A6000 (49140 MiB) |
| **Driver** | 580.126.09 |
| **CUDA** | 13.0 |
| **Python** | 3.10.12 |
| **Docker** | 29.1.5 |
| **RAM** | 47 GB |
| **Disk** | 251 GB (6% used) |
| **nvcc** | Not installed (CUDA toolkit headers absent) |

## Test Plan

| # | Area | Method | Verdict |
|---|---|---|---|
| T1 | PyPI install | `pip3 install nemo-automodel` | PASS |
| T2 | Editable install | `pip3 install -e .` | FAIL |
| T3 | GitHub source install | `pip3 install git+https://github.com/NVIDIA-NeMo/Automodel.git` | PASS |
| T4 | Docker container | `docker pull` + `docker run` | PASS (with finding) |
| T5 | Install extras | `pip3 install nemo-automodel[cli]` + `[vlm]` | PASS |

---

## T1: Install via PyPI (Recommended)

**Objective**: Verify `pip3 install nemo-automodel` installs correctly.

**Checks performed**:
1. Ran `pip3 install nemo-automodel`
2. Verified import: `python3 -c "import nemo_automodel; print(nemo_automodel.__version__)"`

**Results**:
- Installed `nemo-automodel-0.3.0` with all dependencies (torch-2.10.0, transformers-5.4.0)
- Import verification returned `0.3.0`
- Installation took 269s (large dependency tree including PyTorch)

**Observations**:
- Multiple PATH warnings: scripts installed to `~/.local/bin` which is not on PATH
- Doc does not mention adding `~/.local/bin` to PATH

**Verdict**: PASS

---

## T2: Install in Developer Mode (Editable Install)

**Objective**: Verify `pip3 install -e .` works for editable development.

**Checks performed**:
1. `rsync`'d Automodel repo to instance
2. Ran `cd ~/Automodel && pip3 install -e .`

**Results**:
- ERROR: `Project file:///home/shadeform/Automodel has a 'pyproject.toml' and its build backend is missing the 'build_editable' hook. Since it does not have a 'setup.py' nor a 'setup.cfg', it cannot be installed in editable mode. Consider using a build backend that supports PEP 660.`

**Root cause**: The project's `pyproject.toml` build backend does not support PEP 660 editable installs. No `setup.py` or `setup.cfg` fallback exists.

**Verdict**: FAIL

---

## T3: Install via GitHub (Source)

**Objective**: Verify `pip3 install git+https://github.com/NVIDIA-NeMo/Automodel.git` works.

**Checks performed**:
1. Ran the exact command from the doc
2. Verified import and version

**Results**:
- Cloned and built from commit `fedc9d2b`
- Installed `nemo-automodel-0.3.0`
- Import verification passed (19s total)

**Verdict**: PASS

---

## T4: Install via NeMo Docker Container

**Objective**: Verify Docker container pull and GPU-enabled execution.

**Checks performed**:
1. `docker pull nvcr.io/nvidia/nemo-automodel:25.11.00`
2. `docker run --gpus all --rm --shm-size=8g ... python3 -c "import nemo_automodel; print(nemo_automodel.__version__)"`

**Results**:
- Pull succeeded (315s)
- Container runs with GPU access
- Version inside container: `0.2.0rc0`

**Observations**:
- Container `25.11.00` ships `nemo_automodel==0.2.0rc0` while PyPI has `0.3.0`
- `pynvml` deprecation warning displayed during import (cosmetic)
- Doc does not mention the version discrepancy between container and PyPI

**Verdict**: PASS (with finding)

---

## T5: Install Extras (CLI, VLM)

**Objective**: Verify extras install additional dependencies.

**Checks performed**:
1. `pip3 install nemo-automodel[cli]`
2. `pip3 install nemo-automodel[vlm]`

**Results**:
- CLI: already satisfied (deps overlap with base install)
- VLM: installed 33 additional packages (torchvision-0.25.0, timm-1.0.22, open-clip-torch-3.3.0, librosa-0.11.0, decord-0.6.0, etc.)
- Both completed successfully (53s total)

**Verdict**: PASS

---

## Issues Found and Fixed

| # | Issue | Severity | Scope | Fix | Verified |
|---|---|---|---|---|---|
| 1 | Editable install (`pip3 install -e .`) fails: build backend missing PEP 660 `build_editable` hook | High | T2 | Needs code fix (update build backend) or doc fix (document workaround) | No |
| 2 | Docker container version mismatch: `25.11.00` has `0.2.0rc0`, PyPI has `0.3.0` | Medium | T4 | Doc should note container version may lag PyPI, or container needs update | No |
| 3 | PATH warnings not documented: scripts install to `~/.local/bin` which is not on PATH | Low | T1, T5 | Add note about adding `~/.local/bin` to PATH for CLI tools | No |
| 4 | `nvcc` prerequisite: doc says "CUDA 11.8+" but instance has CUDA 13.0 driver without nvcc toolkit | Low | Prerequisites | Clarify whether CUDA runtime (driver) or toolkit (nvcc) is required | No |

## Summary

**Overall**: PASS (with issues) | **Confidence**: 85% :large_orange_diamond:

4/5 installation methods work as documented. The editable install (T2) is broken due to missing PEP 660 support in the build backend. The Docker container ships an older version than PyPI, which is undocumented. PATH warnings on bare-metal pip installs are a usability gap the doc should address. All other methods (PyPI, GitHub source, Docker, extras) install and import correctly.

**Residual items**:
- Docker + Mount method (T6) was not tested (requires interactive shell + long-running example script)
- `uv pip install` (Option B of GitHub source) was not tested
- No functional testing beyond import verification was performed
