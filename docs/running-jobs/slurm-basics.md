---
tags:
  - Running Jobs
  - Slurm
---

# Slurm Basics

AICR uses [Slurm](https://slurm.schedmd.com/) to manage compute resources and schedule jobs. You request resources, Slurm finds available nodes, and your job runs there.

!!! danger "Do not run computation on login nodes"
    Login nodes are shared. Each user is limited to **4 CPU cores** and **5 GiB of memory** across all of their processes on a login node. Work that exceeds the memory limit will be killed; work that exceeds the CPU limit will be throttled. Run computation in a Slurm job instead.

## Partitions

AICR has a few different types of nodes (CPU only nodes, different types of GPUs) and supports both interactive and batch jobs. Nodes are grouped together in *partitions*, which designate the type of resource and what they should be used for. Different partitions may have different sets of rules about how many resources you can use and how long your jobs can run on them.

The partitions on AICR are:

| <div style="width:8em">Partition</div> | GPU Type | Nodes | Max Time | Default Time | Use Case |
|-----------|-----------|-------|----------|--------------|----------|
| `rtx-batch` | RTX Pro 6000 | 17 | 24h | 1h | RTX GPU batch jobs |
| `rtx-devel` | RTX Pro 6000 | 2 | 4h | 15 min | RTX GPU interactive development and testing |
| `b200-batch` | B200 | 25 | 24h | 1h | B200 GPU batch jobs |
| `b200-devel` | B200 | 2 | 4h | 15 min | B200 GPU interactive development and testing |
| `b200-fullnode` | B200 | 4 | 24h | 1h | Whole-node B200 jobs — [trial](#whole-node-b200-jobs-trial) |
| `preemptable` | Mixed | 46 | 24h | 15 min | Lower-priority work that can be [preempted](#preemption) |
| `cpu` *(default)* | — | 5 | 24h | 15 min | Data analysis, workflow orchestration |

Default memory: 1 GB per CPU, except on `b200-fullnode` (16 GB per CPU). Each devel partition limits you to 2 GPUs at a time. Interactive jobs are limited to 4 at a time across the cluster and cannot run in the `-batch` partitions — see [Interactive Jobs](#interactive-jobs). For GPU hardware details, see [System Description](../system-description.md).

To see the partitions on AICR, run the `sinfo` command:

```bash
sinfo
```

The `sinfo` command will tell you the names of the partitions, what their time limits are, how many nodes are in each state, and the names of the nodes in the partitions.

## Checking Available Resources

To see what resources are available run `sinfo`. The `sinfo` command will show how many nodes are in each state. Nodes in "idle" state have all cores available, nodes in "mix" state have some cores available, and nodes in "alloc" state have no cores or other resources available.

```
PARTITION       AVAIL  TIMELIMIT  NODES  STATE NODELIST
cpu*               up 1-00:00:00      5    mix w[0001-0005]
rtx-batch          up 1-00:00:00     14    mix a[0001-0014]
rtx-batch          up 1-00:00:00      3  alloc a[0015-0017]
rtx-devel          up    4:00:00      2    mix a[0018-0019]
b200-batch         up 1-00:00:00     18    mix b[0001-0018]
b200-batch         up 1-00:00:00      7  alloc b[0019-0025]
b200-devel         up    4:00:00      2    mix b[0030-0031]
b200-fullnode      up 1-00:00:00      1    mix b0029
b200-fullnode      up 1-00:00:00      2  alloc b[0026,0028]
b200-fullnode      up 1-00:00:00      1   idle b0027
preemptable        up 1-00:00:00     46    mix a[0001-0017],b[0001-0029]
```

The `*` after `cpu` marks it as the default partition — a job submitted without `--partition` will run there, on CPU nodes with no GPU.

Common node states are:

- `idle`: nodes that are fully available
- `mix`: nodes that have some, but not all, resources allocated
- `alloc`: nodes that are fully allocated
- `resv`: nodes that are reserved and only available to people in their reservation
- `drained`: nodes that are unavailable for use per system administrator request, usually for maintenance purposes, shown as `drain*` due to the default 5-column limit
- `drng`: nodes that are currently allocated a job, but will not be allocated additional jobs; node state will be changed to state `drained` when the last job on it completes
- `down`: nodes that are unavailable for use
- `plnd`: nodes that are planned by the backfill scheduler for a higher priority job

You can also use `sinfo` to see what resources each node has. For example, to see what node types are in the `rtx-devel` and `b200-devel` partitions, run the command:

```
sinfo -O Partition,Nodes,CPUs,Memory,Gres -e
```

In the output you'll see a summary of how many nodes of each configuration is in the partition. You can include multiple partitions by providing a comma separated list to the `-p` flag. The output shows the partition, number of nodes, number of CPUs, amount of Memory (in MB), and any GPUs available on the node:

```

PARTITION       NODES       CPUS        MEMORY          GRES                
cpu*            5           128         1159937         (null)              
rtx-batch       17          128         2321639         gpu:rtx_pro_6000:8       
rtx-devel       2           128         2321639         gpu:rtx_pro_6000:8       
b200-batch      25          128         2321633         gpu:b200:8          
b200-devel      2           128         2321633         gpu:b200:8          
b200-fullnode   4           128         2321633         gpu:b200:8          
preemptable     17          128         2321639         gpu:rtx_pro_6000:8       
preemptable     29          128         2321633         gpu:b200:8          
```

`preemptable` appears twice because it is the only partition that spans more than one node type.

## Running Jobs

How you run your job depends on the type of job you would like to run. There are two "modes" for running jobs: interactive and batch jobs. Interactive jobs allow you to run interactively on a compute node in a shell. Batch jobs, on the other hand, are for running a pre-written script or executable. Interactive jobs are mainly used for testing, debugging, and interactive data analysis. Batch jobs are the traditional jobs you see on an HPC system and should be used when you want to run a script that doesn't require that you interact with it.

### Job Flags

When you start any type of job you specify what resources you need for your job, including cores, memory, GPUs, and other features. You also specify which [partition](#partitions) you would like your job to run on. If you don't specify any of these you will get the default resources: 1 core, a small amount of memory, no GPUs, and it will run on the current default partition. See [Common Job Flags](#common-job-flags) below for the flags to use to request different types of resources, as well as the following pages:

- [GPU Jobs](gpu-jobs.md): For specifics on running GPU jobs
- [CPU Jobs](cpu-jobs.md): For specifics on running CPU jobs

### Interactive Jobs

The basic command for requesting an interactive job on the `rtx-devel` partition is:

```batch
salloc --partition=rtx-devel --gpus=1 --cpus-per-task=4 --mem=16G --time=01:00:00
```

!!! note "Where interactive jobs can run"
    Interactive jobs — `salloc`, or `srun` without a batch script — are not permitted in `rtx-batch`, `b200-batch`, or `b200-fullnode`. Submitting one fails immediately.

    Use `rtx-devel` or `b200-devel` for interactive GPU work, or `cpu` for interactive work that does not need a GPU.

You may hold up to 4 interactive jobs at a time across the cluster. On the devel partitions the binding limit is usually GPUs rather than job count: each devel partition allows you 2 GPUs at a time, so two single-GPU jobs — or one job using both.

The `--partition=rtx-devel` is a flag that is passed to the scheduler, `--partition` specifies the partition. This command will allocate 4 cores (`--cpus-per-task`), one GPU (`--gpus`), and 16GB of RAM (`--mem`) on a node in the `rtx-devel` partition for one hour. Since this is the `rtx-devel` you will get an RTX Pro 6000 GPU.

We set aside a certain number of nodes specifically for interactive jobs in each of the "devel" partitions.

For example:

```bash
[USERNAME@login0001 ~]$ salloc --partition=rtx-devel --gpus=1 --cpus-per-task=4 --mem=16G --time=01:00:00
salloc: Pending job allocation 855421
salloc: job 855421 queued and waiting for resources
salloc: job 855421 has been allocated resources
salloc: Granted job allocation 855421
salloc: Waiting for resource configuration
salloc: Nodes a0018 are ready for job
[USERNAME@a0018 ~]$ 
```

Notice how the command prompt changes from `[USERNAME@login0001 ~]$` to `[USERNAME@a0018 ~]$`. This indicates that your job has started on `a0018` and any commands you issue will run on that node.

### Submitting a Batch Job

Batch jobs are used to run pre-written scripts or run commands that do not need input from you throughout the run. The first step to running a batch job is to write a job script. Job scripts can be "launched" with the `sbatch` command:

```bash
sbatch myscript.sh
```

When you run this command the scheduler will look for the resources requested in the script, allocate those resources to your job, run your script on those resources, and then release those resources once your script completes or the time limit is reached.

Here is an example job script that uses Python to train a model:

```bash
#!/bin/bash
#SBATCH --job-name=my_analysis       # name for identification
#SBATCH --partition=rtx-batch        # which partition
#SBATCH --cpus-per-task=4            # CPU cores per task
#SBATCH --gpus=1                     # number of GPUs
#SBATCH --mem=16G                    # memory
#SBATCH --time=04:00:00              # max wall time (HH:MM:SS)
#SBATCH --output=%x-%j.out           # stdout file

module load miniforge3
module load cuda

python train.py
```

!!! tip "AICR Slurm Accounts"
    Slurm accounts are a way to associate a user's usage to a project. Everyone has a default account associated with their primary project. If you are part of multiple projects you can switch the account by using the `--account` flag.

This script requests the same resources as the [interactive job above](#interactive-jobs): 4 cpu core on the `rtx-batch` partition. The `#SBATCH --partition rtx-batch` may look like a comment but it is not, it is a directive to the scheduler to run with the specified flags. Note that this is the same flag used in the [interactive job example above](#interactive-jobs). It then sets up the job environment to use python with the `miniforge3` and `cuda` modules, and then runs a python script. In general, the same steps and commands you would use to run your job in an interactive job you can put in your job script.

You can think of job scripts as having three sections:

1. Scheduler/Job flags: This is where you request your resources using Slurm flags. See a list of flags below in [Common Job Flags](#common-job-flags).
2. Set up your environment: Load any modules you need, set environment variables, etc. It is better to set this in your job scripts to ensure consistent environments across jobs. We don't recommend putting these commands in your `.bashrc` or running them at the command line before you launch your job.
3. Run your code or application as you would from the command line.

#### Output Files

When you specify your output file name you can use the following patterns in your output file name. For example `my-out.log-%j` creates an output file with the Job ID on at the end: `my-out.log-12345` for Job ID 12345.

| Pattern | Expands To |
|---------|-----------|
| `%j` | Job ID |
| `%x` | Job name |
| `%a` | Array task ID |
| `%A` | Array job ID |
| `%N` | First node hostname |

#### Common Job Flags

Some of the most common job flags are listed below. Some job flags have a single letter "short flag" that you may find convenient to use. You can click on any of the flags to read the full Slurm documentation description for that flag.

| Directive | Short Flag | Description | Example |
|-----------|--|-----------|---------|
| [`--job-name`](https://slurm.schedmd.com/sbatch.html#OPT_job-name) | `-J` |Job name | `--job-name=training` |
| [`--partition`](https://slurm.schedmd.com/sbatch.html#OPT_partition) | `-p` | Partition | `--partition=b200-batch` |
| [`--account`](https://slurm.schedmd.com/sbatch.html#OPT_account) | `-A` | Project account | `--account=proj1_mit` |
| [`--nodes`](https://slurm.schedmd.com/sbatch.html#OPT_nodes) | `-N` | Node count | `--nodes=2` |
| [`--ntasks`](https://slurm.schedmd.com/sbatch.html#OPT_ntasks) | `-n` | Total tasks | `--ntasks=8` |
| [`--ntasks-per-node`](https://slurm.schedmd.com/sbatch.html#OPT_ntasks-per-node) | NA | Tasks per node | `--ntasks-per-node=4` |
| [`--cpus-per-task`](https://slurm.schedmd.com/sbatch.html#OPT_cpus-per-task) | `-c` | CPUs per task | `--cpus-per-task=8` |
| [`--mem`](https://slurm.schedmd.com/sbatch.html#OPT_mem) | NA | Memory per node | `--mem=64G` |
| [`--gpus`](https://slurm.schedmd.com/sbatch.html#OPT_gpus) | `-G`  | Total GPUs | `--gpus=2` |
| [`--gpus-per-node`](https://slurm.schedmd.com/sbatch.html#OPT_gpus-per-node) | NA | GPUs per node | `--gpus-per-node=8` |
| [`--time`](https://slurm.schedmd.com/sbatch.html#OPT_time) | `-t` | Wall time | `--time=12:00:00` |
| [`--output`](https://slurm.schedmd.com/sbatch.html#OPT_output) | `-o`  | Stdout file | `--output=%x-%j.out` |
| [`--error`](https://slurm.schedmd.com/sbatch.html#OPT_error) | `-e`  | Stderr file | `--error=%x-%j.err` |
| [`--array`](https://slurm.schedmd.com/sbatch.html#OPT_array) | `-a`  | Job array | `--array=0-99%10` |
| [`--exclusive`](https://slurm.schedmd.com/sbatch.html#OPT_exclusive) | NA  | Exclusive node | `--exclusive` |
| [`--dependency`](https://slurm.schedmd.com/sbatch.html#OPT_dependency) | `-d`  | Wait for job | `--dependency=afterok:12345` |
| [`--mail-type`](https://slurm.schedmd.com/sbatch.html#OPT_mail-type) | NA  | Email alerts | `--mail-type=END,FAIL` |

## Preemption

The `preemptable` partition spans 46 nodes — the RTX batch nodes, plus the B200 batch and whole-node nodes. The CPU nodes are not part of it, so jobs in the `cpu` partition are never preempted. It gives you access to capacity that is otherwise committed to other partitions, at the cost of your job being interrupted when that capacity is needed. It also applies no per-user GPU cap, unlike the 32-GPU limit on `rtx-batch` and `b200-batch`. Work here counts toward your usage at the same rate as the equivalent batch partition.

Jobs in `preemptable` run at the lowest priority on the cluster. When a job in any other partition needs a node your job is running on, your job is preempted.

!!! warning "Jobs in `preemptable` can be interrupted at any time"
    Only submit work to `preemptable` if it can tolerate being stopped and restarted. Checkpoint your work.

### What happens when a job is preempted

1. Your job receives `SIGTERM`.
2. Up to 60 seconds later, it receives `SIGKILL`.
3. **Batch jobs are automatically requeued.** The job script starts again from the beginning, keeping the same job ID. Any work not written to disk is lost.
4. **Interactive jobs (`salloc`) are not requeued.** The allocation ends.

Your job's final state is `PREEMPTED` with exit code `0:0`.

If your job must never be requeued, submit it with `--no-requeue`. Note that this also prevents the job from being restarted after a node failure or a scheduled maintenance window, so it is rarely the right choice in `preemptable`.

### Checkpointing

Because a requeued job restarts from the beginning, long jobs in `preemptable` should write checkpoints and resume from them. The 60-second window between `SIGTERM` and `SIGKILL` is not enough time to write a large model checkpoint, so checkpoint on a schedule during the run rather than trying to catch the signal.

### Checking whether your job was preempted

```bash
sacct -D -j JOBID --format=JobID,Partition,State,ExitCode,Start,End
```

The `-D` flag is required. Without it, `sacct` shows only the most recent run of a requeued job, so a job that was preempted and requeued will appear under its later state rather than as `PREEMPTED`.

If you search by state rather than by job ID, give both a start and an end time:

```bash
sacct -u $USER -S 2026-09-01 -E now -s PREEMPTED --format=JobID,Partition,State,End
```

## Whole-Node B200 Jobs (Trial)

!!! info "This is a trial"
    The `b200-fullnode` partition is currently being trialled, and may change or be withdrawn in the future.

The `b200-fullnode` partition provides four B200 nodes allocated whole. It is intended for work that needs at least a full node — distributed training across eight or more GPUs, and jobs that need the full NVLink domain.

Request GPUs in multiples of eight — 8, 16, 24 or 32 — which is one node per eight GPUs. Fewer than eight is rejected at submission. Nodes are allocated exclusively, so you receive all 128 cores per node whether or not you ask for them, and you do not need `--exclusive`.

If your work needs one or two GPUs, or parallelises easily across separate jobs, use `b200-batch` or `rtx-batch` instead — a whole-node job counts all eight GPUs toward your usage for as long as it holds the node, whether or not they are all busy.

Memory is still requested explicitly, up to 2000 GB per node.

The following script requests one whole node — eight B200 GPUs and 1800 GB of memory — for 12 hours:

```bash
#!/bin/bash
#SBATCH --job-name=fullnode_train
#SBATCH --partition=b200-fullnode
#SBATCH --nodes=1
#SBATCH --gpus=8
#SBATCH --mem=1800G
#SBATCH --time=12:00:00
#SBATCH --account=ACCOUNT_NAME
#SBATCH --output=%x-%j.out

module load miniforge3
module load cuda

torchrun --nproc_per_node=8 train.py
```

### Interaction with preemption

These nodes are also part of the `preemptable` partition, so a whole-node job will preempt any `preemptable` work running on the node it is assigned.

## Fairshare and Priority

AICR allocates resources fairly using a three-level hierarchy: **Institution → Project → User**. All institutions have equal shares. Your priority depends on recent usage relative to your institution's and project's allocation. Specify `--account` correctly and Slurm handles the rest.

### Slurm Accounts

Slurm accounts are a way to associate a user's usage to a project. Everyone on AICR has a default account associated with their primary project. If you are part of multiple projects you may have multiple Slurm accounts.

You can see your default account with the command:

```
sacctmgr show user $USER -p
```

To see all of the Slurm accounts that you have access to run on, run:

```
sacctmgr show user $USER withassoc format=user,account -p
```

When you run a job think about what project that job is for, and specify the account for that project. This ensures your usage is accounted for in the correct project if you are on multiple projects.

## Slurm Environment Variables

Slurm keeps track of job parameters in environment variables, which can sometimes be useful. Inside a running job:

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
- [CPU Jobs for Data Analysis and Processing](cpu-jobs.md) — CPU partition for data work
- [Monitoring Jobs](monitoring.md) — checking status and resource usage
<!-- TODO: Add later
- [Job Arrays](job-arrays.md) — parameter sweeps
- [Job Troubleshooting](troubleshooting.md) — common failures
 -->
