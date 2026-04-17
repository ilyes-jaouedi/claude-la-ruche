# claude-la-ruche

A Claude Code skill for the **La Ruche HPC cluster** at Paris-Saclay / Mésocentre.

## What it does

Invoke `/la-ruche` followed by your task and Claude will assist you with:

- SSH connection and VS Code Remote SSH setup
- SLURM job submission (CPU, GPU, MPI, OpenMP, arrays, interactive)
- GPU partition selection (V100, A100, P100)
- Module loading (`module load`, `module avail`, etc.)
- Storage management (`$HOME`, `$WORKDIR`, `$TMPDIR`, quotas)
- Conda environment setup (avoiding home quota issues)
- Deep learning training job templates (PyTorch, etc.)
- Jupyter notebook via SSH tunnel
- Nice DCV visualization portal

## Install

Inside Claude Code:

```
/plugin marketplace add IlyesJaouedi/claude-la-ruche
/plugin install la-ruche@claude-la-ruche
```

## Usage

```
/la-ruche submit a PyTorch training job on A100
/la-ruche set up a conda environment in $WORKDIR
/la-ruche check why my job is stuck pending
/la-ruche open an interactive GPU session
/la-ruche how do I transfer files to the cluster?
```

## Cluster reference

| Resource | Details |
|----------|---------|
| Login node | `ruche.mesocentre.universite-paris-saclay.fr` |
| GPU options | V100 32 GB · A100 40 GB · P100 12 GB |
| CPU nodes | 216 × 40-core nodes, 192 GB RAM |
| Storage | 50 GB `$HOME` · 500 GB `$WORKDIR` (extendable) |
| Scheduler | SLURM |
| Visualization | Nice DCV at `https://ruche.mesocentre.universite-paris-saclay.fr:8443` |
| Support | ruche.support@universite-paris-saclay.fr |

## Documentation

- [La Ruche official docs](https://mesocentre.pages.centralesupelec.fr/user_doc/ruche/)
- [Mésocentre Paris-Saclay](https://mesocentre.universite-paris-saclay.fr)

## License

MIT
