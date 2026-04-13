---
tags:
  - Running Jobs
  - Slurm
---

# Slurm Basics

AICR uses [Slurm](https://slurm.schedmd.com/) (version 25.11.1) to manage compute resources and schedule jobs. You request resources, Slurm finds available nodes, and your job runs there.

!!! danger
    Do not run computation on login nodes. They are limited to 4 CPU cores and 8 GB memory per user. Processes exceeding these limits will be terminated.

## Partitions

| Partition | GPUs/Node | Max Time | Default Time | Use Case |
|-----------|-----------|----------|--------------|----------|
| `cpu` | — | 24h | 15 min | Data analysis, workflow orchestration |
| `rtx-batch` | 8 | 24h | 1h | RTX GPU batch jobs |
| `rtx-devel` | 8 | 4h | 15 min | RTX GPU development and testing |
| `b200-batch` | 8 | 24h | 1h | B200 GPU batch jobs |
| `b200-devel` | 8 | 4h | 15 min | B200 GPU development and testing |

Default memory: 1 GB per CPU. Devel partitions limited to 4 concurrent jobs per user. For GPU hardware details, see [GPU Overview](../gpu/overview.md).

```bash
$ sinfo
```

## Submitting a Batch Job

```bash
#!/bin/bash
#SBATCH --job-name=my_analysis       # name for identification
#SBATCH --partition=rtx-batch        # which partition
#SBATCH --nodes=1                    # number of nodes
#SBATCH --ntasks=1                   # number of tasks (processes)
#SBATCH --cpus-per-task=4            # CPU cores per task
#SBATCH --gpus=1                     # number of GPUs
#SBATCH --mem=16G                    # memory
#SBATCH --time=04:00:00              # max wall time (HH:MM:SS)
#SBATCH --account=ACCOUNT_NAME       # institution account (required)
#SBATCH --output=%x-%j.out           # stdout file

module purge
module load cuda

python train.py
```

```bash
$ sbatch my_job.sh
Submitted batch job 12345
```

!!! tip
    Always specify `--account`. Valid accounts: `bu`, `hu`, `mit`, `neu`, `umass`, `uml`, `umb`, `umassd`, `umassmed`, `yale`, `ma`, `aihub`.

## Interactive Sessions

```bash
$ salloc --partition=rtx-devel --gpus=1 --cpus-per-task=4 --mem=16G --time=01:00:00 --account=ACCOUNT_NAME
```

This gives you a shell on a compute node. Type `exit` when done.

## Output Files

| Pattern | Expands To |
|---------|-----------|
| `%j` | Job ID |
| `%x` | Job name |
| `%a` | Array task ID |
| `%N` | First node hostname |

## Common Directives

| Directive | Description | Example |
|-----------|-------------|---------|
| `--job-name` | Job name | `--job-name=training` |
| `--partition` | Partition | `--partition=b200-batch` |
| `--account` | Institution account | `--account=mit` |
| `--nodes` | Node count | `--nodes=2` |
| `--ntasks` | Total tasks | `--ntasks=8` |
| `--ntasks-per-node` | Tasks per node | `--ntasks-per-node=4` |
| `--cpus-per-task` | CPUs per task | `--cpus-per-task=8` |
| `--mem` | Memory per node | `--mem=64G` |
| `--gpus` | Total GPUs | `--gpus=2` |
| `--gpus-per-node` | GPUs per node | `--gpus-per-node=8` |
| `--gres` | Generic resource | `--gres=gpu:2` |
| `--time` | Wall time | `--time=12:00:00` |
| `--output` | Stdout file | `--output=%x-%j.out` |
| `--error` | Stderr file | `--error=%x-%j.err` |
| `--array` | Job array | `--array=0-99%10` |
| `--exclusive` | Exclusive node | `--exclusive` |
| `--dependency` | Wait for job | `--dependency=afterok:12345` |
| `--mail-type` | Email alerts | `--mail-type=END,FAIL` |

## Resource Defaults

| Setting | Value |
|---------|-------|
| Default memory per CPU | 1 GB |
| Batch max wall time | 24 hours |
| Devel max wall time | 4 hours |
| Max concurrent devel jobs | 4 per user |
| Max job array size | 10,000 tasks |

## Fairshare

AICR allocates resources fairly using a three-level hierarchy: **Institution → Project → User**. All institutions have equal shares. Your priority depends on recent usage relative to your institution's and project's allocation. Specify `--account` correctly and Slurm handles the rest.

## Slurm Environment Variables

Inside a running job:

| Variable | Contents |
|----------|----------|
| `$SLURM_JOB_ID` | Job ID |
| `$SLURM_JOB_NAME` | Job name |
| `$SLURM_ARRAY_TASK_ID` | Array task index |
| `$SLURM_NNODES` | Number of nodes |
| `$SLURM_NTASKS` | Total tasks |
| `$SLURM_CPUS_PER_TASK` | CPUs per task |
| `$SLURM_NODELIST` | Allocated nodes |
| `$SLURM_SUBMIT_DIR` | Submission directory |
| `$SLURM_GPUS_ON_NODE` | GPUs on this node |

## See Also

- [GPU Jobs](gpu-jobs.md) — requesting GPUs and choosing partitions
- [Analysis and Data Processing](analysis.md) — CPU partition for data work
- [Job Arrays](job-arrays.md) — parameter sweeps
- [Monitoring Jobs](monitoring.md) — checking status and resource usage
- [Job Troubleshooting](troubleshooting.md) — common failures
