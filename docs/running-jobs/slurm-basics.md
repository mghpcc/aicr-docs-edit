---
tags:
  - Running Jobs
  - Slurm
---

# Slurm Basics

AICR uses [Slurm](https://slurm.schedmd.com/) to manage compute resources and schedule jobs. You request resources, Slurm finds available nodes, and your job runs there.

!!! danger
    Do not run computation on login nodes. They are limited to 4 CPU cores and 8 GB memory per user. Processes exceeding these limits will be terminated.

<!-- TODO: Are these limits true? -->

## Partitions

AICR has a few different types of nodes (CPU only nodes, different types of GPUs) and supports both interactive and batch jobs. Nodes are grouped together in *partitions*, which designate the type of resource and what they should be used for. Different partitions may have different sets of rules about how many resources you can use and how long your jobs can run on them.

The partitions on AICR are:

| Partition | GPUs/Node | Max Time | Default Time | Use Case |
|-----------|-----------|----------|--------------|----------|
| `cpu` | — | 24h | 15 min | Data analysis, workflow orchestration |
| `rtx-batch` | 8 | 24h | 1h | RTX GPU batch jobs |
| `rtx-devel` | 8 | 4h | 15 min | RTX GPU interactive development and testing |
| `b200-batch` | 8 | 24h | 1h | B200 GPU batch jobs |
| `b200-devel` | 8 | 4h | 15 min | B200 GPU interactive development and testing |

Default memory: 1 GB per CPU. Development (devel) partitions limited to 4 concurrent jobs per user. For GPU hardware details, see [GPU Overview](../gpu/overview.md).

To see the partitions on AICR, run the `sinfo` command:

```bash
$ sinfo
```

The `sinfo` command will tell you the names of the partitions, what their time limits are, how many nodes are in each state, and the names of the nodes in the partitions.

## Checking Available Resources

To see what resources are available run `sinfo`. The `sinfo` command will show how many nodes are in each state. Nodes in "idle" state have all cores available, nodes in "mix" state have some cores available, and nodes in "alloc" state have no cores or other resources available.

```
sinfo -p mit_normal
PARTITION  AVAIL  TIMELIMIT  NODES  STATE  NODELIST
mit_normal    up   12:00:00      2   resv  node[2704-2705]
mit_normal    up   12:00:00      1   mix   node1707
mit_normal    up   12:00:00      1   alloc node1708
mit_normal    up   12:00:00     29   idle  node[1600-1625,1706,1806-1807]
```

Common node states are:

- `idle`: nodes that are fully available
- `mix`: nodes that have some, but not all, resources allocated
- `alloc`: nodes that are fully allocated
- `resv`: nodes that are reserved and only available to people in their reservation
- `drained`: nodes that are unavailable for use per system administrator request, usually for maintenance purposes. It is shown as `drain*` due to the default 5-column limit.
- `drng`: nodes that are currently allocated a job, but will not be allocated additional jobs. The node state will be changed to state drained when the last job on it completes. 
- `down`: nodes that are unavailable for use. 
- `plnd`: nodes that are planned by the backfill scheduler for a higher priority job

You can also use `sinfo` to see what resources each node has. For example, to see what node types are in the `rtx-devel` and `b200-devel` partitions, run the command:

```
sinfo -p rtx-devel,b200-devel -O Partition,Nodes,CPUs,Memory,Gres -e
```

In the output you'll see a summary of how many nodes of each configuration is in the partition. You can include multiple partitions by providing a comma separated list to the `-p` flag. The output shows the partition, number of nodes, number of CPUs, amount of Memory (in MB), and any GPUs available on the node:

<!-- TODO: Replace with AICR numbers -->

## Running Jobs

How you run your job depends on the type of job you would like to run. There are two "modes" for running jobs: interactive and batch jobs. Interactive jobs allow you to run interactively on a compute node in a shell. Batch jobs, on the other hand, are for running a pre-written script or executable. Interactive jobs are mainly used for testing, debugging, and interactive data analysis. Batch jobs are the traditional jobs you see on an HPC system and should be used when you want to run a script that doesn't require that you interact with it.

### Job Flags

When you start any type of job you specify what resources you need for your job, including cores, memory, GPUs, and other features. You also specify which [partition](#partitions) you would like your job to run on. If you don't specify any of these you will get the default resources: 1 core, a small amount of memory, no GPUs, and it will run on the current default partition. See the page on [Requesting Resources](requesting-resources.md) for the flags to use to request different types of resources.

### Interactive Jobs

The basic command for requesting an interactive job on the `rtx-devel` partition is:

```batch
salloc --partition=rtx-devel --gpus=1 --cpus-per-task=4 --mem=16G --time=01:00:00 --account=ACCOUNT_NAME
```

The `--partition=rtx-devel` is a flag that is passed to the scheduler, `--partition` specifies the partition. This command will allocate 4 cores, one GPU, and 16GB of RAM on a node in the `rtx-devel` partition for one hour. Since this is the `rtx-devel` you will get an RTX Pro 6000 GPU.

We set aside a certain number of nodes specifically for interactive jobs in each of the "devel" partitions.

For example:

<!-- TODO: Replace with AICR example -->

```bash
[user01@orcd-login001 ~]$ salloc -p rtx-devel -G 1
salloc: Pending job allocation 60159437
salloc: job 60159437 queued and waiting for resources
salloc: job 60159437 has been allocated resources
salloc: Granted job allocation 60159437
salloc: Waiting for resource configuration
salloc: Nodes node1806 are ready for job
[user01@node1806 ~]$ 
```

Notice how the command prompt changes from `[user01@orcd-login001 ~]$` to `[user01@node1806 ~]$`. This indicates that `user01` has started an interactive job on `node1806` and any commands issued will run on this node.

<!-- TODO: Replace with AICR hostnames -->

### Submitting a Batch Job

Batch jobs are used to run pre-written scripts or run commands that do not need input from you throughout the run. The first step to running a batch job is to write a job script. Job scripts can be "launched" with the `sbatch` command:

```bash
sbatch myscript.sh
```

When you run this command the scheduler will look for the resources requested in the script, allocate those resources to your job, run your script on those resources, and then release those resources once your script completes or the time limit is reached.

Here is an example job script that uses Python to train a model:

<!-- TODO: Verify Batch Job fields (is account required?) -->

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

module load minforge
module load cuda

python train.py
```

!!! tip
    Always specify `--account`. Valid accounts: `bu`, `hu`, `mit`, `neu`, `umass`, `uml`, `umb`, `umassd`, `umassmed`, `yale`, `ma`, `aihub`.

This script requests the same resources as the [interactive job above](#interactive-jobs): 1 cpu core on the `rtx-batch` partition. The `#SBATCH -p rtx-batch` may look like a comment but it is not, it is a directive to the scheduler to run with the specified flags. Note that this is the same flag used in the [interactive job example above](#interactive-jobs). It then sets up the job environment to use python with the `miniforge` and `cuda` modules, and then runs a python script. In general, the same steps and commands you would use to run your job in an interactive job you can put in your job script.

You can think of job scripts as having three sections:

1. Scheduler/Job flags: This is where you request your resources using Slurm flags.
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
| [`--account`](https://slurm.schedmd.com/sbatch.html#OPT_account) | `-A` | Institution account | `--account=mit` |
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

## Fairshare and Priority

AICR allocates resources fairly using a three-level hierarchy: **Institution → Project → User**. All institutions have equal shares. Your priority depends on recent usage relative to your institution's and project's allocation. Specify `--account` correctly and Slurm handles the rest.

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
- [Data Analysis and Processing](analysis.md) — CPU partition for data work
- [Job Arrays](job-arrays.md) — parameter sweeps
- [Monitoring Jobs](monitoring.md) — checking status and resource usage
- [Job Troubleshooting](troubleshooting.md) — common failures
