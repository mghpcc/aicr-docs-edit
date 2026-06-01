
---
tags:
 - Hardware
---

# AICR System Overview

AICR is a GPU-focused AI computing cluster housed at the [MGHPCC](https://www.mghpcc.org/) in Holyoke, MA. It serves researchers at Boston University, Harvard, MIT, Northeastern, UMass (five campuses), and Yale as part of the [Massachusetts AI Hub](https://aihub.masstech.org/).

## Hardware

| Component | Count | Details  |
|-----------|-------|----------|
| NVIDIA B200 GPU Nodes | 31 | 8 B200 GPUs per node (248 total), 180 GB HBM3e per GPU, NVLink 5, liquid-cooled |
| NVIDIA RTX PRO 6000 GPU Nodes | 19 | 8 RTX PRO 6000 GPUs per node (152 total), 96 GB GDDR7 per GPU, PCIe Gen5 |
| Service Nodes | 32 | Dell R6715, 128-core AMD EPYC 9745, 1.125 TB RAM, login/Slurm/OnDemand/Globus/CPU compute  |
| Storage | ~14.3 PiB effective | VAST Data all-flash, 360 GB/s throughput, encryption at rest, GPU Direct Storage |
| Networking | N/A | Rail-optimized IB (B200 GPU), cluster IB (storage/MPI), Ethernet (management) |

## Partitions

| Partition | Cores | Memory | GPUs/Node | GPU Memory |Max Time | Use Case |
|-----------|----------|-----------|----------|----------|----------|----------|
| `rtx-batch` | 128 | 2.25 TB | 8 RTX PRO 6000 | 96 GB | 24h | RTX GPU batch workloads |
| `rtx-devel` | 128 | 2.25 TB | 8 RTX PRO 6000 | 96 GB | 4h | Interactive GPU development |
| `b200-batch` | 128 | 2.25 TB | 8 B200 | 180 GB | 24h | B200 GPU batch workloads |
| `b200-devel` | 128 | 2.25 TB | 8 B200 | 180 GB | 4h | Interactive GPU development |
| `cpu`     | 128 | 1 TB | — | — | 24h | Data analysis, workflow orchestration |

Default memory per CPU: 1 GB. Development partitions have a 4-concurrent-job limit per user.

## Storage

| Storage | Path | Quota | Snapshots | Purge |
|---------|------|-------|-----------|-------|
| Home | `/home/USERNAME` | 100 GiB | 7-day | None |
| Scratch | `/scratch/USERNAME` | 10 TiB | None | Regular |
| Work | `/work/INSTITUTION/PROJECT` | Varies | 7-day | None |

!!! warning
    AICR storage relies on snapshots only — there are no off-site backups. Back up irreplaceable data externally.

## Interconnect

AICR nodes are connected by InfiniBand NDR400 (400 Gb/s) for high-bandwidth, low-latency communication between GPUs and storage. This enables fast distributed training across multiple GPU nodes and high-throughput access to the VAST storage system via NFS-RDMA.

## See Also

- [Getting Started](getting-started.md) — get running on AICR in 5 minutes
- [Slurm Basics](running-jobs/slurm-basics.md) — how to submit jobs
- [Filesystem Overview](files/overview.md) — storage details