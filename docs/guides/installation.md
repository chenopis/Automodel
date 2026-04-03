# Install NeMo Automodel

This guide explains how to install NeMo Automodel for LLM, VLM, and OMNI models on various platforms and environments. Depending on your use case, there are several ways to install it:

| Method                  | Dev Mode | Use Case                                                          | Recommended For             |
| ----------------------- | ---------|----------------------------------------------------------------- | ---------------------------- |
| 📦 **PyPI**             | - | Install stable release with minimal setup                         | Most users, production usage |
| 🐳 **Docker**           | - | Use in isolated GPU environments, e.g., with NeMo container       | Multi-node deployments     |
| 🐍 **Git Repo**         | ✅ | Use the latest code without cloning or installing extras manually | Power users, testers         |
| 🧪 **Editable Install** | ✅ | Contribute to the codebase or make local modifications            | Contributors, researchers    |
| 🐳 **Docker + Mount**   | ✅ | Use in isolated GPU environments, e.g., with NeMo container       | Multi-node deployments     |

## Prerequisites

### System Requirements
- **Python**: 3.9 or higher
- **CUDA driver**: 11.8 or higher (for GPU support). You need the NVIDIA driver with CUDA runtime support; the CUDA toolkit (`nvcc`) is not required for inference or fine-tuning because PyTorch ships its own CUDA libraries. Verify with `nvidia-smi`.
- **Memory**: Minimum 16GB RAM, 32GB+ recommended
- **Storage**: At least 50GB free space for models and datasets

### Hardware Requirements

- **GPU**: NVIDIA GPU with 8GB+ VRAM (16GB+ recommended)
- **CPU**: Multi-core processor (8+ cores recommended)
- **Network**: Stable internet connection for downloading models

---
## Installation Options for Non-Developers
This section explains the easiest installation options for non-developers, including using pip3 via PyPI or leveraging a preconfigured NVIDIA NeMo Docker container. Both methods offer quick access to the latest stable release of NeMo Automodel with all required dependencies.
### Install via PyPI (Recommended)

For most users, the easiest way to get started is using `pip3`.

```bash
pip3 install nemo-automodel
```
:::{tip}
This installs the latest stable release of NeMo Automodel from PyPI.

To verify the install, run `python -c "import nemo_automodel; print(nemo_automodel.__version__)"`. See [nemo-automodel on PyPI](https://pypi.org/project/nemo-automodel/).
:::

:::{note}
If you see warnings about scripts installed to a directory that is not
on `PATH` (for example, `~/.local/bin`), add that directory to your
shell's `PATH`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Add this line to your `~/.bashrc` or `~/.zshrc` to make it permanent.
:::

### Install via NeMo Docker Container
You can use NeMo Automodel with the NeMo Docker container. Pull the container by running:
```bash
docker pull nvcr.io/nvidia/nemo-automodel:25.11.00
```
:::{note}
The above `docker` command uses the [`25.11.00`](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/nemo-automodel?version=25.11.00) container. Use the [most recent container](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/nemo-automodel) version to ensure you get the latest version of Automodel and its dependencies like torch, transformers, etc.

The container version of NeMo Automodel may differ from the latest PyPI
release because containers follow a separate release cadence. To check
the version inside the container, run:

```bash
python -c "import nemo_automodel; print(nemo_automodel.__version__)"
```
:::

Then you can enter the container using:
```bash
docker run --gpus all -it --rm \
  --shm-size=8g \
  nvcr.io/nvidia/nemo-automodel:25.11.00
```

---
## Installation Options for Developers
This section provides installation options for developers, including pulling the latest source from GitHub, using editable mode, or mounting the repo inside a NeMo Docker container.
### Install via GitHub (Source)

If you want the **latest features** from the `main` branch or want to contribute:

#### Option A - Use `pip` with git repo:
```bash
pip3 install git+https://github.com/NVIDIA-NeMo/Automodel.git
```
:::{note}
This installs the repo as a standard Python package (not editable).
:::

#### Option B - Use `uv` with git repo:
```bash
uv pip install git+https://github.com/NVIDIA-NeMo/Automodel.git
```
:::{note}
`uv` handles virtual environment transparently and enables more reproducible installs.
:::

### Install in Developer Mode (Editable Install)
To contribute or modify the code:
```bash
git clone https://github.com/NVIDIA-NeMo/Automodel.git
cd Automodel
pip3 install --upgrade pip setuptools
pip3 install -e .
```

:::{note}
This installs Automodel in editable mode, so changes to the code are
immediately reflected in Python.

The `pip3 install --upgrade pip setuptools` step is required because
editable installs use [PEP 660](https://peps.python.org/pep-0660/),
which needs `pip >= 21.3` and `setuptools >= 64.0`. If you skip this
step on a system with an older pip, you may see an error about a
missing `build_editable` hook.

Alternatively, you can use `uv` which handles this automatically:

```bash
uv pip install -e .
```
:::

### Mount the Repo into a NeMo Docker Container
To run `Automodel` inside a NeMo container while **mounting your local repo**, follow these steps:

**Step 1**: Clone the Automodel repository.

```bash
git clone https://github.com/NVIDIA-NeMo/Automodel.git
cd Automodel
```

**Step 2**: Pull the latest compatible NeMo container. Replace
`25.11.00` with the
[latest version](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/nemo-automodel)
if needed.

```bash
docker pull nvcr.io/nvidia/nemo-automodel:25.11.00
```

**Step 3**: Run the NeMo container with GPU support, shared memory,
and mount the repo.

```bash
docker run --gpus all -it --rm \
  -v $(pwd):/workspace/Automodel \
  --shm-size=8g \
  nvcr.io/nvidia/nemo-automodel:25.11.00 /bin/bash -c \
    "cd /workspace/Automodel && pip install -e . && python3 examples/llm/finetune.py"
```

:::{note}
The `-v $(pwd):/workspace/Automodel` option mounts your local clone
into the container so that code changes on the host are reflected
immediately. The `--shm-size=8g` flag increases shared memory, which
PyTorch data loaders require for multi-worker loading.
:::

## Bonus: Install Extras
Some functionality may require optional extras. You can install them like this:
```bash
pip3 install nemo-automodel[cli]    # Installs only the Automodel CLI
pip3 install nemo-automodel         # Installs the CLI and all LLM dependencies.
pip3 install nemo-automodel[vlm]    # Install all VLM-related dependencies.
```

## Summary
| Goal                        | Command or Method                                               |
| --------------------------- | --------------------------------------------------------------- |
| Stable install (PyPI)       | `pip3 install nemo-automodel`                                   |
| Latest from GitHub          | `pip3 install git+https://github.com/NVIDIA-NeMo/Automodel.git` |
| Editable install (dev mode) | `pip install -e .` after cloning                                |
| Run without installing      | Use `PYTHONPATH=$(pwd)` to run scripts                          |
| Use in Docker container     | Mount repo and `pip install -e .` inside container              |
| Fast install (via `uv`)     | `uv pip install ...`                                            |
