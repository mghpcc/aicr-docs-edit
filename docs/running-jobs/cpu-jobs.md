---
tags:
  - Running Jobs
  - CPU
  - Dask
  - Python
---

# CPU Jobs for Data Analysis and Processing

The `cpu` partition provides high-memory nodes for data analysis, preprocessing, and workflow orchestration. These nodes do not have GPUs — use them for work that benefits from many CPU cores and large memory rather than GPU acceleration.

!!! tip "CPU Jobs on AICR"
    AICR is not meant for CPU workloads. A small CPU-only partition is provided as a convenience for small data analysis tasks related to your GPU jobs. Heavy CPU workloads should be done on your home institution's cluster.

## When to Use the CPU Partition

- **Data preprocessing**: cleaning, transforming, or merging large datasets before GPU training
- **Post-processing**: aggregating results, generating reports, statistical analysis
- **Workflow orchestration**: coordinating multi-step pipelines that launch GPU jobs

For GPU-accelerated work (training, inference, anything using CUDA), use the `rtx-*` or `b200-*` partitions instead. See [GPU Jobs](gpu-jobs.md) for more information.

## CPU Partition Specs

| Property | Value |
|----------|-------|
| Nodes | AMD EPYC 9745 (5 nodes) |
| Cores per node | 128 |
| Memory per node | 1 TB |
| GPUs | None |
| Max wall time | 24 hours |
| Default time | 15 minutes |

## Example: Basic CPU Job

The following is an example job script for a CPU job with 16 cores and 32GB of RAM.

```bash
#!/bin/bash
#SBATCH --job-name=data_analysis
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=32G
#SBATCH --time=04:00:00
#SBATCH --output=%x-%j.out

module load miniforge3

python analyze_data.py
```

## See Also

- [Slurm Basics](slurm-basics.md) — general job submission
- [GPU Jobs](gpu-jobs.md) - running GPU jobs