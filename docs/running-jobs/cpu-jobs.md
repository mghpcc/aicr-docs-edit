---
tags:
  - Running Jobs
  - CPU
  - Dask
  - Python
---

# CPU Jobs for Data Analysis and Processing

The `cpu` partition provides high-memory nodes for data analysis, preprocessing, and workflow orchestration. These nodes do not have GPUs — use them for work that benefits from many CPU cores and large memory rather than GPU acceleration.

## CPU Partition Specs

| Property | Value |
|----------|-------|
| Nodes | w-series (5 nodes) |
| Cores per node | 128 |
| Memory per node | 1 TB |
| GPUs | None |
| Max wall time | 24 hours |
| Default time | 15 minutes |

## When to Use the CPU Partition

- **Data preprocessing**: cleaning, transforming, or merging large datasets before GPU training
- **Parallel analysis**: running analysis in parallel across many cores with Dask, mpi4py, or GNU Parallel
- **Post-processing**: aggregating results, generating reports, statistical analysis
- **Workflow orchestration**: coordinating multi-step pipelines that launch GPU jobs

For GPU-accelerated work (training, inference, anything using CUDA), use the `rtx-*` or `b200-*` partitions instead.

!!! tip
    AICR is not meant for CPU workloads. A small CPU-only partition is provided for small data analysis tasks related to your GPU jobs. Heavy CPU workloads should be done on your home institution's cluster.

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
#SBATCH --account=ACCOUNT_NAME
#SBATCH --output=%x-%j.out

module load miniforge

python analyze_data.py
```

<!-- TODO: Probably don't need this, AICR is not the right place.

## Parallel Analysis with Dask

[Dask](https://dask.org/) scales Python data analysis (Pandas, NumPy, scikit-learn) across multiple cores or nodes. It is well-suited for the CPU partition's large-memory nodes.

### Single-Node Dask

For analysis that fits on one node but benefits from parallel processing:

```bash
#!/bin/bash
#SBATCH --job-name=dask_analysis
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --cpus-per-task=64
#SBATCH --mem=512G
#SBATCH --time=04:00:00
#SBATCH --account=ACCOUNT_NAME
#SBATCH --output=%x-%j.out

module purge
module load python

python dask_analysis.py
```

In your Python script:

```python
import dask.dataframe as dd
from dask.distributed import Client

# Use local cluster with allocated CPUs
client = Client(n_workers=16, threads_per_worker=4)

# Read and process a large dataset in parallel
df = dd.read_parquet("/scratch/USERNAME/data/large_dataset.parquet")
result = df.groupby("category").agg({"value": "mean"}).compute()
result.to_csv("/scratch/USERNAME/results/summary.csv")
```

### Multi-Node Dask

For datasets that exceed a single node's memory or processing capacity, see the [Parallel Data Analysis with Dask](../recipes/dask.md) recipe.

## Parallel Analysis with mpi4py

For MPI-based parallel analysis across CPU nodes:

```bash
#!/bin/bash
#SBATCH --job-name=mpi_analysis
#SBATCH --partition=cpu
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=16
#SBATCH --time=02:00:00
#SBATCH --account=ACCOUNT_NAME
#SBATCH --output=%x-%j.out

module purge
module load python

srun --mpi=pmix python parallel_analysis.py
```

!!! note
    AICR uses PMIx for MPI process management. Always pass `--mpi=pmix` to `srun`.

-->

## See Also

- [Slurm Basics](slurm-basics.md) — general job submission
<!-- TODO: Review
- [Parallel Data Analysis with Dask](../recipes/dask.md) — detailed Dask recipe
- [Working with Large Datasets](../ai-tools/large-datasets.md) — staging and I/O patterns
-->