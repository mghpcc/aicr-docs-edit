
# AICR System Overview

AICR is a GPU-focused AI computing cluster housed at the [MGHPCC](https://www.mghpcc.org/) in Holyoke, MA. It serves researchers at Boston University, Harvard, MIT, Northeastern, UMass (five campuses), and Yale as part of the [Massachusetts AI Hub](https://aihub.masstech.org/).

## Hardware

| Component | Count | Details | More Info |
|-----------|-------|---------|-----------|
| NVIDIA B200 GPU Nodes | 31 | 8 B200 GPUs per node (248 total), 180 GB HBM3e per GPU, NVLink 5, liquid-cooled | [Details](hardware/b200-nodes.md) |
| NVIDIA RTX PRO 6000 GPU Nodes | 19 | 8 RTX PRO 6000 GPUs per node (152 total), 96 GB GDDR7 per GPU, PCIe Gen5 | [Details](hardware/rtx-nodes.md) |
| Service Nodes | 32 | Dell R6715, 128-core AMD EPYC 9745, 1.125 TB RAM, login/Slurm/OnDemand/Globus | [Details](hardware/service-nodes.md) |
| Storage | ~14.3 PiB effective | VAST Data all-flash, 360 GB/s throughput, encryption at rest, GPU Direct Storage | [Details](hardware/storage.md) |
| Networking | 4 fabrics | Rail-optimized IB (B200 GPU), cluster IB (storage/MPI), Ethernet (management), OOB (control) | [Details](hardware/networking.md) |

## Partitions

| Partition | Hardware | GPUs/Node | Max Time | Use Case |
|-----------|----------|-----------|----------|----------|
| `cpu` | w-series (128 cores, 1 TB) | — | 24h | Data analysis, workflow orchestration |
| `rtx-batch` | a-series | 8 RTX PRO 6000 | 24h | RTX GPU batch workloads |
| `rtx-devel` | a-series | 8 RTX PRO 6000 | 4h | Interactive GPU development |
| `b200-batch` | b-series | 8 B200 | 24h | B200 GPU batch workloads |
| `b200-devel` | b-series | 8 B200 | 4h | Interactive GPU development |

Default memory per CPU: 1 GB. Development partitions have a 4-concurrent-job limit per user.

## Storage

| Storage | Path | Quota | Snapshots | Purge |
|---------|------|-------|-----------|-------|
| Home | `/home/USERNAME` | 100 GiB | 7-day | None |
| Scratch | `/scratch/USERNAME` | ~10 TiB | None | Regular |
| Work | `/work/INSTITUTION/PROJECT` | Varies | 7-day | None |

!!! warning
    AICR storage relies on snapshots only — there are no off-site backups. Back up irreplaceable data externally.

## Interconnect

AICR nodes are connected by InfiniBand NDR400 (400 Gb/s) for high-bandwidth, low-latency communication between GPUs and storage. This enables fast distributed training across multiple GPU nodes and high-throughput access to the VAST storage system via NFS-RDMA.


## Links

[Slurm Basics/Running Jobs](#)

[Filesystem Overview](#)

---
tags:
 - hardware
---
