# Using `uv` for Python Environment Management on HiPerGator

`uv` is a modern package management tool that provides a faster, more user-friendly alternative to pip/conda for managing Python project environments.
While `uv` works out of the box, **please configure your cache location** as described in the Cache Configuration section below to avoid storage issues on HiPerGator.

*Note that `uv` is only for Python, unlike `pixi` which supports multiple languages.*

**Advantages:**
- **Project-local environments**: All environment files are stored in your project directory (`.venv/`), not in your home directory. (uv does use a global cache, which you [should configure](#cache-configuration).)
- **Easy Notebook Integration**: Use `uv add package` in notebook cells to safely install packages directly into your environment
- **Reproducible environments**: Uses `uv.toml` for declarative environment specification
- **Native Python venv support**: Leverages Python's built-in venv module for environment management
- **Faster installations**: uv uses a more efficient solver and caching system

## Table of Contents
- [Installation](#installation)
- [Cache Configuration](#cache-configuration)
- [Basic Usage](#basic-usage)
- [Notebook Integration](#notebook-integration)
- [PyTorch](#pytorch)
- [Using uv in HPG Scripts](#using-uv-in-hpg-scripts)
- [Getting Help](#getting-help)

## Installation
Install uv using the command below:

```bash
curl -fsSL https://uv.sh/install.sh | bash
```
> **⚠️ Security Note**: Always be careful when running bash scripts from the internet.

## Cache Configuration
HiPerGator home directories have limited storage (40GB), so configure uv to use your allocation storage for caching.

**Why does uv need a cache?** While uv keeps environment-specific files in your project directory (`.venv/`), it uses a global cache for downloaded packages, metadata, and solve results. This cache can be shared across projects, making subsequent installations much faster and reducing storage usage overall.

> We recommend using a symlink to your allocation directory since this will also take care of other applications that use the `~/.cache` directory.

### Option 1: Symlink Cache Directory (Recommended)

Move your cache to your allocation and create a symlink:

```bash
# Replace with your allocation path
# Let's say that your allocation is /blue/dr-florida and your username is allie.gator

# Move your existing cache if it's there
mv ~/.cache/ /blue/dr-florida/allie.gator/.cache

# Create symlink
ln -s /blue/dr-florida/allie.gator/.cache ~/.cache
```

### Option 2: Set uv Cache Environment Variable

Add to your `~/.bashrc`:

```bash
export UV_CACHE_DIR="/blue/dr-florida/allie.gator/.pixi_cache"
```

## Basic Usage

### Initialize a Project

Create a new project directory and initialize uv:

```bash
mkdir my-project
cd my-project
uv init
```

This creates a `pyproject.toml` file and `.venv/` directory for your project.

### Add Dependencies

Add packages to your project:

```bash
# Add regular dependencies
uv add numpy pandas matplotlib

# Add development dependencies
uv add --dev pytest black flake8

# Add packages with version constraints
uv add "requests>=2.25.0" "fastapi<1.0.0"
```

### Install Dependencies

Install all dependencies from `pyproject.toml`:

```bash
uv sync
```

### Reproduce Environments
To recreate an environment in a new location, copy the `pyproject.toml` file and run:

```bash
uv sync
```

### Run Commands

Execute commands in your uv environment:

```bash
# Run Python scripts
uv run python my_script.py

# Run arbitrary commands
uv run jupyter notebook
uv run pytest
```

## Notebook Integration

### Quick Setup

1. Install ipykernel:
```bash
uv add --dev ipykernel
```

2. Create kernel (for JupyterLab):
```bash
uv run ipython kernel install --user --name "my-project" --display-name "my-kernel-name"
```

3. Use in notebooks:
   - VS Code: Select the kernel from `.venv/bin/python`
   - JupyterLab: Select your project kernel

### Adding Packages in Notebooks

Install packages directly from notebook cells:

```python
# This updates pyproject.toml and installs the package
!uv add seaborn
```

## PyTorch

For deep learning on HiPerGator, you need to install PyTorch with CUDA support for NVIDIA GPUs.

### Configure PyTorch with CUDA

After running `uv init`, edit your `pyproject.toml` to include PyTorch sources:

```toml
[project]
name = "my-pytorch-uv-project"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.10"
dependencies = [
]

[tool.uv.sources]
torch = [
  { index = "pytorch-cu128", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
]
torchvision = [
  { index = "pytorch-cu128", marker = "sys_platform == 'linux' or sys_platform == 'win32'" },
]

[[tool.uv.index]]
name = "pytorch-cu128"
url = "https://download.pytorch.org/whl/cu128"
explicit = true
```

### Install PyTorch

```bash
uv add torch
uv add torchvision
```

### Verify CUDA Installation

Test your PyTorch CUDA installation:

```bash
uv run python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}')"
```

See the [PyTorch guide](https://docs.astral.sh/uv/guides/integration/pytorch/) for more details.

## Using uv in HPG Scripts

### Command Line Usage

You can use `uv` commands directly in your terminal or scripts. For example, to run a Python script within your uv environment:

```bash
uv run my_script.py
```

### Job Scripts

Use uv in SLURM job scripts:

```bash
#!/bin/bash
#SBATCH --job-name=uv-job
#SBATCH --output=job-%j.log
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16gb
#SBATCH --time=24:00:00

cd /path/to/your/project
uv run python train_model.py
```


## Getting Help

- [Official Documentation](https://docs.astral.sh/uv/)
- `uv --help` for command help
- `uv [command] --help` for specific command help