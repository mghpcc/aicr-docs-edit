---
tags:
  - Running Jobs
  - GPU
  - Slurm
---

# GPU Jobs

AICR's primary resource is its GPUs. This page covers how to request GPUs, choose the right partition, and submit GPU jobs. <!-- TODO: Adding later For GPU hardware details, see [GPU Overview](../gpu/overview.md).-->

## Choosing a Partition

| Partition | GPU Model | GPUs/Node | Max Time | Best For |
|-----------|-----------|-----------|----------|----------|
| `rtx-batch` | RTX PRO 6000 | 8 | 24h | Production GPU workloads |
| `rtx-devel` | RTX PRO 6000 | 8 | 4h | Testing and debugging |
| `b200-batch` | B200 | 8 | 24h | Large-scale AI/ML training |
| `b200-devel` | B200 | 8 | 4h | Testing and debugging |

!!! tip
    Use `devel` partitions for testing — they have shorter queue times. Switch to `batch` for production runs.

## Requesting GPUs

The simplest way to request GPUs:

```bash
#SBATCH --gpus=1                    # one GPU (any type on the partition)
#SBATCH --gpus=4                    # four GPUs
#SBATCH --gpus-per-node=8           # all 8 GPUs on a node
```

To request a specific GPU type, use the following syntax:

```bash
#SBATCH --gpus=rtx6000:4    # 4 RTX PRO 6000 GPUs
#SBATCH --gpus=b200:8       # 8 B200 GPUs
```

<!-- TODO: GRES type names (rtx6000, b200) are based on the gres.conf.j2 template pattern — verify against production gres.conf once deployed -->

!!! note
    Match your partition to your GPU type. Use `rtx-batch`/`rtx-devel` for RTX GPUs and `b200-batch`/`b200-devel` for B200 GPUs.

    If you specify the partition you do not need to specify the GPU type, since partitions are homogenous (have only one type of GPU).

## Single-GPU Job

The following script requests one GPU, 8 CPUs, and 32GB of RAM for 8 hours in the `rtx-batch` partition. Since the job is in `rtx-batch`, this job will get an RTX Pro 6000 GPU.

```bash
#!/bin/bash
#SBATCH --job-name=single_gpu
#SBATCH --partition=rtx-batch
#SBATCH --gpus=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=08:00:00
#SBATCH --account=ACCOUNT_NAME
#SBATCH --output=%x-%j.out

module load miniforge
module load cuda

python train.py
```

## Multi-GPU Job (Single Node)

The following script request multiple GPUs (4) on one node in the `b200-batch` partition. It is also asking for 32 CPUs, 128GB of CPU memory, and 12 hours. Since the job is in `b200-batch`, this job will get 4 B200 GPUs.

```bash
#!/bin/bash
#SBATCH --job-name=multi_gpu
#SBATCH --partition=b200-batch
#SBATCH --nodes=1
#SBATCH --gpus-per-node=4
#SBATCH --cpus-per-task=32
#SBATCH --mem=128G
#SBATCH --time=12:00:00
#SBATCH --account=ACCOUNT_NAME
#SBATCH --output=%x-%j.out

module load miniforge
module load cuda

torchrun --nproc_per_node=4 train.py
```

<!-- TODO: Should this be torchrun?? Check that this works.-->

<!-- TODO: Add when pages exist: For multi-node GPU jobs (distributed training across nodes), see [Distributed Training](../ai-tools/distributed-training.md) and [Multi-GPU Computing](../gpu/multi-gpu.md). -->


## Verifying GPU Access

Inside a running job, check that GPUs are visible with:

```bash
$ nvidia-smi
```

This shows GPU model, memory, utilization, and temperature. If no GPUs appear, verify that you requested them with `--gpus` and submitted to a GPU partition.

For PyTorch:

```python
import torch
print(torch.cuda.is_available())       # should print True
print(torch.cuda.device_count())       # should match requested GPUs
print(torch.cuda.get_device_name(0))   # GPU model name
```

## GPU Memory Considerations

- RTX PRO 6000 and B200 GPUs each have substantial memory, but large models or large batch sizes can still cause out-of-memory (OOM) errors
- If you encounter OOM: reduce batch size, enable mixed precision (`torch.amp`), or use gradient checkpointing
- Monitor GPU memory usage with `nvidia-smi` during interactive sessions to right-size your requests

## See Also

<!-- Adding Later
- [GPU Overview](../gpu/overview.md) — hardware specs and architecture
- [Multi-GPU Computing](../gpu/multi-gpu.md) — NCCL and GPU-to-GPU communication
- [Distributed Training](../ai-tools/distributed-training.md) — scaling beyond one node
 -->
- [Slurm Basics](slurm-basics.md) — general job submission
