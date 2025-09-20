# Using Pixi for Package Management on HiPerGator

Pixi is a modern package management tool that provides a faster, more user-friendly alternative to conda for managing project environments.
While pixi works out of the box, **please configure your cache location** as described in the Cache Configuration section below to avoid storage issues on HiPerGator.

**Advantages:**
- **Project-local environments**: Environment files are stored in your project directory (`.pixi/`). (pixi does use a global cache, which you [should configure](#cache-configuration).)
- **Reproducible environments**: Uses `pixi.toml` for declarative environment specification
- **Faster installations**: Pixi uses an efficient solver and caching system
- **Multi-language support**: Works seamlessly with Python, R, Rust, C++, and other languages
- **Cross-platform compatibility**: Works consistently across different operating systems

## Table of Contents

- [Installation](#installation)
- [Cache Configuration](#cache-configuration)
- [Basic Usage](#basic-usage)
- [Python-Specific Features](#python-specific-features)
- [Multi-Language Support](#multi-language-support)
- [Notebook Integration](#notebook-integration)
- [Environment Management](#environment-management)
- [Using Pixi in HPG Scripts](#using-pixi-in-hpg-scripts)
- [Getting Help](#getting-help)

## Installation

Install pixi using the official installer:

```bash
curl -fsSL https://pixi.sh/install.sh | bash
```

> **⚠️ Security Note**: Always be careful when running bash scripts from the internet.
This one is from the official pixi site, but in general, review scripts before executing them since a malicious script could compromise your system.

After installation, restart your shell or source your `~/.bashrc`:

```bash
source ~/.bashrc
```

Verify installation:

```bash
pixi --version
```

## Cache Configuration

HiPerGator home directories have limited storage (40GB), so configure pixi to use your allocation storage for caching.

**Why does pixi need a cache?** While pixi keeps environment-specific files in your project directory (`.pixi/`), it uses a global cache for downloaded packages, metadata, and solve results. This cache can be shared across projects, making subsequent installations much faster and reducing storage usage overall.

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

### Option 2: Set Pixi Cache Environment Variable

Add to your `~/.bashrc`:

```bash
export PIXI_CACHE_DIR="/blue/dr-florida/allie.gator/.pixi_cache"
```

> **Warning**: Never set `PIXI_CACHE_DIR` to your main allocation directory (`/blue/dr-florida/allie.gator`), as `pixi clean cache` could delete your entire folder.

## Basic Usage

### Initialize a Project

Using a terminal, create a new project directory and initialize pixi:

```bash
mkdir my-project
cd my-project
pixi init
```

This creates a `pixi.toml` file that defines your project configuration.
More information about the `pixi.toml` format can be found in the [official documentation](https://pixi.sh/latest/reference/pixi_manifest/).

### Activate the Environment

Activate the pixi shell environment:

```bash
pixi shell
```

This is similar to `conda activate` (conda) or `source venv/bin/activate` (Python's built-in venv module).
You'll see the environment name in your prompt.

Deactivate with:

```bash
exit
```

### Add Packages

Add packages to your environment:

```bash
# Add Python packages
pixi add python==3.10.* numpy pandas matplotlib

# Add packages from specific channels
pixi add pytorch --channel pytorch
pixi add cuda --channel nvidia/label/cuda-12.4.0

# Add development dependencies
pixi add --dev pytest black

# Add system-level packages
pixi add cmake gcc
```

### Install from Existing pixi.toml

If you have an existing `pixi.toml` file:

```bash
pixi install
```

## Python-Specific Features

### Using pyproject.toml

Pixi supports managing Python projects using `pyproject.toml` files, which is a popular standard for Python project configuration:

```bash
pixi init --format pyproject
pixi add python=3.10
```

### Installing PyPI Packages

Add Python packages from PyPI (AKA like pip):

```bash
pixi add --pypi requests flask[async] fastapi
```

You can also specify version constraints:

```bash
pixi add --pypi "numpy>=1.24.0" "pandas<2.0.0"
```

### Installing from Git Repositories

Install packages directly from Git:

```bash
pixi add --pypi "clip @ git+https://github.com/openai/CLIP.git"
```

### Using pixi for PyTorch AI Projects
To use PyTorch on NVIDIA GPUs, we need to make sure we compile PyTorch with the appropriate CUDA version. This requires adding the correct channels and dependencies in your `pixi.toml` file. This is discussed in pixi's [official documentation](https://pixi.sh/latest/python/pytorch/#installing-from-conda-forge). Here’s an example `pixi.toml` configuration that uses conda-forge:

```toml
[project]
channels = ["nvidia/label/cuda-12.4.0", "nvidia", "conda-forge", "pytorch", "main", "r", "msys2"]
name = "my-pytorch-repo"
platforms = ["linux-64"]
version = "0.1.0"

[dependencies]
cuda = {channel="nvidia/label/cuda-12.4.0"}
pytorch = {channel="pytorch"}
torchvision = {channel="pytorch"}
pytorch-cuda = {channel="pytorch"}
python = "3.10.*"
```

## Multi-Language Support

Pixi supports multiple programming languages in the same environment:

### R Support

```bash
pixi add r-base r-ggplot2 r-dplyr
```

### Rust Support

```bash
pixi add rust
```

### C/C++ Development

```bash
pixi add cmake gcc gxx make
```

## Notebook Integration

### Creating Jupyter Kernels for Jupyter Notebooks (Python)

1. Initialize your project and add Python:

```bash
pixi init
pixi add python ipykernel
```

2. Install the kernel:

```bash
pixi run python -m ipykernel install --user --name "my-project" --display-name "My Project"
```

### VS Code Integration

In VS Code:

1. Open your project directory and .ipynb notebook file
2. In the kernel selector (top right), choose "Select Another Kernel"
3. Select the Python interpreter from the pixi environment (it should be called 'default' and live at '.pixi/envs/default/bin/python')

### Installing Packages directly from Notebook Cells
Install packages directly from notebook cells:

```python
# Install packages from within the notebook
!pixi add numpy pandas matplotlib

# Now you can import them
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

This approach updates your `pixi.toml` file automatically.

## Environment Management

### List Environments

View all pixi environments:

```bash
pixi list
```

### Remove Packages

Remove packages from your environment:

```bash
pixi remove numpy pandas
```

### Clean Cache

Clean the pixi cache (use carefully; make sure you've done your [cache configuration](#cache-configuration) correctly):

```bash
pixi clean cache
```

### Update Packages

Update all packages:

```bash
pixi update
```

Update specific packages:

```bash
pixi update numpy pandas
```
## Using Pixi in HPG Scripts

### Command Line Usage

Run commands in your pixi environment without activating:

```bash
pixi run python my_script.py
pixi run jupyter notebook
```

If you activate your env with `pixi shell`, you can run commands directly:

```bash
python my_script.py
```

### SLURM Job Scripts

Use pixi in SLURM job scripts:

```bash
#!/bin/bash
#SBATCH --job-name=pixi-job
#SBATCH --output=./logs/job-%j.log
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16gb
#SBATCH --time=24:00:00
#SBATCH --account=your-account
#SBATCH --qos=your-qos

cd /full/path/to/your/project

# Run your script
pixi run python train_model.py --epochs 100
```

## Getting Help

- [Official Pixi Documentation](https://pixi.sh/latest/)
- Check `pixi --help` for command options