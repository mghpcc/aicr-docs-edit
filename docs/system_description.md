
# System Description

An outline of the components which make up the AICR cluster.

## Hardware


| #| NVIDIA GPU Nodes | Details|
| --- | --------- | ----------- |
| 31 | 8x B200 (Blackwell) Nodes | 248 B200 GPUs with InfiniBand connectivity |
| 19| 8x RTX6000 PRO (Blackwell) Nodes | 152 RTX6000 GPUs |
| | | |


| #| Service Nodes | Details |
| --- | --------- | ----------- |
| 32| Dell R6715 Services nodes | Provides login, Open OnDemand, Globus, Slurm, etc. |
| | | |

| #| Storage | Details |
| --- | --------- | ----------- |
| 15PB| VAST Storage system |  Provides Home, Scratch, and Work space (See [Filesystem Overview](#))|
| | | |

## Links

[Slurm Basics/Running Jobs](#)

[Filesystem Overview](#)

---
tags:
 - hardware
---
