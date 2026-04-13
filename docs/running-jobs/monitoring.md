---
tags:
  - Running Jobs
  - Slurm
---

# Monitoring and Managing Jobs

## Check Your Jobs

View your running and pending jobs:

```bash
$ squeue --me
```

View jobs on a specific partition:

```bash
$ squeue -p rtx-batch
```

## Detailed Job Information

Inspect a running or pending job:

```bash
$ scontrol show job JOBID
```

This shows the full job configuration: allocated nodes, resources, time limits, working directory, and more.

## Job History and Resource Usage

After a job completes, use `sacct` to see what resources it actually used:

```bash
$ sacct -j JOBID --format=JobID,JobName,Partition,Elapsed,MaxRSS,State
```

Check your recent job history:

```bash
$ sacct -u $USER --starttime=now-7days --format=JobID,JobName,Partition,Elapsed,State,ExitCode
```

!!! tip
    Use `sacct` to check whether your jobs are using the resources you requested. If `MaxRSS` is much lower than your `--mem` request, reduce memory in future jobs to improve your fairshare.

### Useful sacct Format Fields

| Field | Description |
|-------|-------------|
| `JobID` | Job identifier |
| `JobName` | Job name |
| `Partition` | Partition used |
| `Elapsed` | Actual wall time |
| `MaxRSS` | Peak memory usage |
| `State` | Final job state (COMPLETED, FAILED, TIMEOUT, etc.) |
| `ExitCode` | Exit code (0 = success) |
| `AllocCPUS` | CPUs allocated |
| `AllocTRES` | All resources allocated (CPUs, memory, GPUs) |

## Partition and Node Status

```bash
$ sinfo
```

See which nodes are available, allocated, or down. Useful for checking whether your target partition has capacity.

```bash
$ sinfo -p rtx-batch --format="%N %T %G %m"
```

This shows node names, state, GPUs, and memory for a specific partition.

## Cancel a Job

```bash
$ scancel JOBID
```

Cancel all your jobs:

```bash
$ scancel -u $USER
```

Cancel a specific array task:

```bash
$ scancel JOBID_TASKID
```

## Checking GPU Utilization in a Running Job

If you have a running GPU job, check GPU utilization by connecting to the node:

```bash
$ squeue -u $USER                    # find your node name
$ ssh NODE_NAME nvidia-smi           # check GPU usage (requires active job on that node)
```

!!! note
    You can only SSH to compute nodes where you have an active Slurm job.

## Jobstats

[Jobstats](https://princetonuniversity.github.io/jobstats/) is available for reviewing job performance — CPU, memory, and GPU utilization for completed and in-progress jobs. This is useful for identifying underutilized resources and right-sizing future requests.

<!-- TODO: verify Jobstats is deployed and document usage -->

## See Also

- [Slurm Basics](slurm-basics.md) — submitting jobs
- [Job Troubleshooting](troubleshooting.md) — diagnosing failures
- [Efficient Resource Usage](../best-practices/resource-usage.md) — right-sizing requests
