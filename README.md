# TensorFlow 2.15 Deployment Guide
## Environment: Python 3.8 Server + CUDA 12.2 Driver

> [!IMPORTANT]
> **Environment Status:**
> - **System Python:** If 3.8 (Incompatible with TF 2.15)
> - **CUDA Driver:** Ensure version is ≥ 12.0 (Compatible)
> - **Goal:** Deploy TensorFlow 2.15 (Requires Python 3.9-3.11)

---

## 1. Constraints & Prerequisites

**Why we must not upgrade the system Python:**

| Risk | Impact |
|------|--------|
| **System stability** | Core OS utilities depend on the specific system Python version |
| **Permissions** | Requires root access which is often restricted |
| **Dependencies** | Shared infrastructure may break for other users |

**Solution:** Use user-space virtualization (Miniconda/Virtualenv) to install Python 3.11 independently.

---

## 2. Recommended: Miniconda Installation

This method requires **no root access** and creates an isolated environment.

### Online Installation (Standard)

```bash
# 1. Download installer
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# 2. Install to home directory
bash Miniconda3-latest-Linux-x86_64.sh -b -p $HOME/miniconda3

# 3. Initialize conda
$HOME/miniconda3/bin/conda init bash
source ~/.bashrc

# 4. Create Python 3.11 environment
conda create -n <your_env_name> python=3.11 -y
conda activate <your_env_name>
python --version  # Confirms Python 3.11.x
```

### Offline / Air-Gapped Installation

If the server has no internet access, download the pre-packaged offline kit from this repository's **[Releases](https://github.com/fathalwi02/TensorFlow-2.15-Deployment-to-Server/releases)** page.

#### A. Download Artifacts (On Internet-Connected Machine)

1.  Go to the **Releases** tab.
2.  Download the following core files:
    *   `Miniconda3-latest-Linux-x86_64.sh` (Installer)
    *   `tf_offline.tar.gz` (TensorFlow 2.15 Wheels + Dependencies)

#### B. Transfer
Transfer `Miniconda3-latest-Linux-x86_64.sh` and `tf_offline.tar.gz` to the server via **FileZilla (SFTP)** or USB drive.

#### C. Installation (On Server)

```bash
# 1. Install Miniconda
bash ~/Miniconda3-latest-Linux-x86_64.sh -b -p $HOME/miniconda3
source ~/.bashrc

# 2. Unpack Wheels
tar -xzf tf_offline.tar.gz

# 3. Create Environment & Install
conda create -n <your_env_name> python=3.11 -y
conda activate <your_env_name>

# 4. Install Packages from Local Directory
pip install --no-index --find-links=tf_offline/ tensorflow
```

---

## 3. GPU Configuration

**Verification:**
Run `nvidia-smi` to confirm the driver version is **≥ 525.60.13** (required for CUDA 12). Ensure your system shows a compatible version.

### Installation Check
TensorFlow 2.15 will automatically use the bundled CUDA libraries if installed via `pip install tensorflow[and-cuda]`.

```python
import tensorflow as tf
print(f"TF Version: {tf.__version__}")
print(f"GPU: {tf.config.list_physical_devices('GPU')}")
```

**Troubleshooting:**
If `libcudart.so` errors occur, verify `LD_LIBRARY_PATH` includes the conda environment's lib directory:
```bash
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH
```

---

## 4. Alternative: Container Deployment

If conda is not an option, use a container runtime.

### Docker
```bash
docker run --gpus all -it -v <PROJECT_DIR>:/workspace tensorflow/tensorflow:2.15.0-gpu bash
```

### Apptainer / Singularity
Common in research environments without root Docker access.
```bash
apptainer build tf215.sif docker://tensorflow/tensorflow:2.15.0-gpu
apptainer exec --nv tf215.sif python <YOUR_SCRIPT.py>
```

---

## Quick Reference

| Action | Command |
|--------|---------|
| New Env | `conda create -n <your_env_name> python=3.11` |
| Activate | `conda activate <your_env_name>` |
| GPU Check | `nvidia-smi` |
| TF Install | `pip install tensorflow[and-cuda]==2.15.0` |
