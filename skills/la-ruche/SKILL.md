---
name: la-ruche
description: HPC assistant for La Ruche cluster at Paris-Saclay / Mésocentre. Use when the user asks about connecting, submitting jobs, managing storage, loading modules, or running experiments on La Ruche.
argument-hint: "[task] e.g. submit-job, check-quota, setup-env, interactive-gpu"
disable-model-invocation: false
---

# La Ruche — HPC Cluster Assistant (Paris-Saclay Mésocentre)

You are an expert assistant for the La Ruche HPC cluster. Use the reference below to help the user with any cluster-related task. Always provide ready-to-run commands. Warn about login-node policies when relevant.

---

## Cluster Quick Facts

- **OS:** Rocky Linux 8.10
- **Interconnect:** OmniPath 100 Gbit/s
- **Filesystem:** IBM Spectrum Scale (GPFS), 480 TiB, 9 GB/s — shared across all nodes
- **Scheduler:** SLURM
- **Support:** ruche.support@universite-paris-saclay.fr

---

## 1. Connection

```bash
# SSH login
ssh username@ruche.mesocentre.universite-paris-saclay.fr

# With X forwarding
ssh -XY username@ruche.mesocentre.universite-paris-saclay.fr

# File transfer
scp -r my_dir username@ruche.mesocentre.universite-paris-saclay.fr:/workdir/username
rsync -P -r my_dir username@ruche.mesocentre.universite-paris-saclay.fr:/workdir/username
```

**VS Code Remote SSH config** (`~/.ssh/config` on local machine):
```
Host ruche
    HostName ruche.mesocentre.universite-paris-saclay.fr
    User your_username
```
Then use "Remote-SSH: Connect to Host" → `ruche`.
⚠️ Do NOT run heavy kernels or compilation through VS Code on the login node — it violates the no-computation policy. For GPU/Jupyter work, tunnel through an `srun` session.

**Login nodes:** `ruche01`, `ruche02` (round-robin DNS)

---

## 2. CRITICAL: Login Node Policy

**Running computations on login nodes is STRICTLY FORBIDDEN.**
Violations trigger automated process kills and warning emails.

Allowed on login nodes: editing files, submitting jobs (`sbatch`), light `git`/`scp` operations.
NOT allowed: Python scripts, MATLAB, compilation of large projects, Jupyter kernels.

All compute work → submit via SLURM.

---

## 3. SLURM Partitions

### CPU Partitions (216 nodes × 40 cores × 192 GB RAM)

| Partition | Max Walltime | Max CPUs/user | Max Mem/user | Notes |
|-----------|-------------|---------------|--------------|-------|
| `cpu_short` | 1 h | 1,000 | 4.5 TB | **Default** |
| `cpu_med` | 4 h | 1,000 | 4.5 TB | |
| `cpu_long` | 168 h (7 days) | 160 | 720 GB | |
| `cpu_prod` | 6 h | 2,000 | 9 TB | Nights & weekends only |
| `cpu_scale` | 1 h | 4,000 | 18 TB | Large-scale parallel |

### Memory-Optimized Partitions

| Partition | RAM/node | Max Walltime | Max CPUs/user |
|-----------|----------|-------------|---------------|
| `mem_short` | 1.5 TB | 1 h | 80 |
| `mem` | 1.5 TB | 72 h | 160 |
| `fusion_shm` | up to 1.5 TB | 168 h | 160 |
| `fusion_shm_big` | up to 1.5 TB | 168 h | 320 |

### GPU Partitions

| Partition | GPU | VRAM | Max Walltime | Max GPUs/user |
|-----------|-----|------|-------------|---------------|
| `gpu_test` | Tesla V100 | 32 GB | 1 h | 8 |
| `gpu` | Tesla V100 | 32 GB | 24 h | 8 |
| `gpua100` | HGX A100 | 40 GB | 24 h | 4 |
| `gpup100` | Tesla P100 | 12 GB | 168 h | — |

**GPU hardware totals:** 40× V100, 36× A100, 8× P100

---

## 4. SLURM Job Scripts

### Minimal script template
```bash
#!/bin/bash
#SBATCH --job-name=my_job
#SBATCH --output=%x.o%j          # stdout → jobname.oJOBID
#SBATCH --error=%x.e%j           # stderr
#SBATCH --partition=cpu_short    # MANDATORY — no default partition
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=00:30:00          # HH:MM:SS
#SBATCH --mail-type=ALL

module purge
module load anaconda3/2023.09-0/none-none

python my_script.py
```

### GPU job template
```bash
#!/bin/bash
#SBATCH --job-name=gpu_job
#SBATCH --output=%x.o%j
#SBATCH --error=%x.e%j
#SBATCH --partition=gpua100
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=10
#SBATCH --mem=40G
#SBATCH --gres=gpu:1
#SBATCH --time=04:00:00

module purge
module load anaconda3/2023.09-0/none-none
# module load cuda/...   ← check available: module avail

source activate myenv
python train.py
```

### OpenMP job
```bash
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=20
#SBATCH --mem=80G
export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK}
./my_openmp_binary
```

### MPI job
```bash
#SBATCH --ntasks=80
#SBATCH --partition=cpu_short
srun ./my_mpi_binary
```

### Interactive sessions
```bash
# CPU interactive
srun --nodes=1 --time=00:30:00 -p cpu_short --pty /bin/bash

# GPU interactive (V100)
srun --nodes=1 --gres=gpu:1 -p gpu --pty /bin/bash

# GPU interactive (A100)
srun --nodes=1 --gres=gpu:1 -p gpua100 --pty /bin/bash
```

### Job arrays
```bash
sbatch --array=0-31 job.sh        # indices 0–31
# Access index inside script: $SLURM_ARRAY_TASK_ID
```

---

## 5. SLURM Monitoring Commands

```bash
sbatch script.sh          # submit job → prints JOBID
squeue -u $USER           # your running/pending jobs
scancel <jobid>           # cancel a job
sacct -j <jobid>          # accounting / resource usage
seff <jobid>              # efficiency report (CPU/mem utilisation)
sinfo                     # partition and node status
```

---

## 6. Module System

```bash
module avail                              # list all modules
module list                               # currently loaded
module load <name>                        # load module
module purge                              # clear all

# Key modules
module load anaconda3/2023.09-0/none-none
module load miniconda3/25.5.1/none-none
module load matlab/R2025a/none-none
module load apptainer/1.4.4/gcc-15.1.0
```

Module files live at: `/gpfs/softs/modules/modulefiles/`
For CUDA versions: run `module avail` after login — versions change.

---

## 7. Storage

| Path | Variable | Quota | File limit | Notes |
|------|----------|-------|-----------|-------|
| `/home/username` | `$HOME` | **50 GB** (fixed) | 200k files | Scripts, light config |
| `/workdir/username` | `$WORK` / `$WORKDIR` | **500 GB** (extendable) | 400k files | Datasets, results, envs |
| `scratch/login-jobid` | `$TMPDIR` | Per-job, auto-deleted | — | Local SSD during job |

⚠️ **No backup system.** Both `$HOME` and `$WORKDIR` are unprotected — backup your own data externally.

```bash
ruche-quota              # check your quotas
ruche-quota -m           # compact single-line output
```

**Conda environment best practice** (avoid filling $HOME):
```bash
mv ~/.conda /workdir/$USER/.conda
ln -s /workdir/$USER/.conda ~/.conda
```
Use `source activate myenv` (not `conda activate`) to avoid shell conflicts.

**Quota increase requests:** email ruche.support@universite-paris-saclay.fr
- Up to 2 TB: standard request
- 2–20 TB: requires justification
- Over 20 TB: rejected

---

## 8. Visualization (Nice DCV)

No Open OnDemand portal. Instead use Nice DCV remote desktop:
- **URL:** `https://ruche.mesocentre.universite-paris-saclay.fr:8443`
- Login with cluster credentials → select "Linux Desktop" → configure walltime → launch
- For post-processing only (Paraview, etc.) — NOT for computation
- Also accessible via the native Nice DCV desktop client

**Troubleshooting DCV:** if connection fails, conda or module commands in `.bash_profile`/`.bashrc` are often the culprit. Temporarily rename them.

---

## 9. Common Workflows

### Deep learning training job (PyTorch + A100)
```bash
#!/bin/bash
#SBATCH --job-name=dl_train
#SBATCH --output=%x.o%j
#SBATCH --error=%x.e%j
#SBATCH --partition=gpua100
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=10
#SBATCH --mem=80G
#SBATCH --gres=gpu:1
#SBATCH --time=12:00:00

module purge
module load anaconda3/2023.09-0/none-none
source activate pytorch_env

cd $WORKDIR/my_project
python train.py --epochs 100 --output $WORKDIR/results/
```

### Jupyter notebook via SSH tunnel
1. Start Jupyter on a compute node (in an interactive `srun` session or a batch job):
   ```bash
   jupyter notebook --no-browser --port=8888 --ip=0.0.0.0
   ```
2. From your local machine, create an SSH tunnel:
   ```bash
   ssh -L 8888:nodeXXX:8888 username@ruche.mesocentre.universite-paris-saclay.fr
   ```
3. Open `http://localhost:8888` in your local browser.

---

## 10. Hardware Reference

**Login nodes (2×):**
- Intel Xeon Gold 6230 (20-core, 2.1 GHz), 192 GB RAM
- 1× Nvidia Tesla V100S (32 GB) on one login node (visualization only)

**CPU nodes (216×):** 2× Intel Xeon Gold 6230 = 40 cores/node, 192 GB RAM

**Large-mem nodes (14×):** 4× Intel Xeon Gold 6230 = 80 cores/node, 1.5 TB RAM + 870 GB NVMe

**Fusion nodes (20×):** Intel Xeon Gold 6148, 192 GB–1.5 TB RAM

**GPU nodes:**
- 10× nodes with 4× V100 32 GB (40 V100s total)
- 9× nodes with 4× A100 40 GB (36 A100s total)
- 4× nodes with 2× P100 12 GB (8 P100s total)

---

Now, based on `$ARGUMENTS`, help the user accomplish their specific task using the information above.
If no specific task is given, ask: "What do you need to do on La Ruche? (e.g., submit a job, set up Python environment, check quota, run interactive GPU session, transfer files)"
