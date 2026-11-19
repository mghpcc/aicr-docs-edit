---
tags:
  - Running Jobs
  - Slurm
  - Reservation
---

# Reservations on AICR

## Overview 

!!! warning "Reservations are intended for exceptional non-routine work that needs whole nodes continuously for several days."  

!!! warning "Reservations are arranged through your institution."


AICR does not take reservation requests from individual users. Your institution decides which projects receive a reservation and requests it on their behalf at least one week in advance.

A reservation holds up to three whole B200 nodes for a single project for up to one week. Jobs in a reservation have no per-job time limit. Reservations are a temporary offering while AICR develops a longer-term allocation framework, and the terms below may change.

**Try [`b200-fullnode`](#whole-node-b200-jobs-trial) first.** It needs no request and suits work that fits in 24-hour runs or can checkpoint and resubmit. 

**Use it or lose it.** If no job is running in the reservation within 2 hours of its start, the reservation is cancelled and the nodes return to general use.

**B200 only.** Reservations are not available on RTX nodes.

**The `temp-reservation` partition.** Reserved jobs run in this partition. It can be used only with a reservation arranged through your institution.

When your reservation is set up, your institution's support team will send instructions for submitting jobs to it.

!!! info "Getting help with reservations"
    Contact your institution's [support team](../getting-help.md#institutional-support-teams) if you have questions about reservations, would like to request a reservation, or need help with an existing reservation.

## Submitting jobs to a reservation

A reservation sets aside a fixed number of B200 nodes for you for a set time window. Only jobs submitted to the reservation can run on those nodes during the reservation window.

### Before you start

Once your reservation has been set up, you should receive a confirmation email or ticket with the following:

| Item | Meaning |
|---|---|
| `<reservation_name>` | The name of your reservation |
| `<N>` | Number of nodes in the reservation (each node has 8 GPUs) |
| Start and end time | When the reservation begins and ends |


To confirm `<N>` or the start and end times, you can run:

```bash
scontrol show reservation <reservation_name>
```

Look for `NodeCnt`, `StartTime`, and `EndTime` in the output.

!!! warning "Start within two hours"

    You must have a job running in the reservation within two hours of the start time. If the reservation is not in use by then, it will be cancelled and the nodes returned to general use. See the [reservation policy](<link to policy page>).

!!! tip "Submit your jobs before the reservation starts"

    You don't have to wait for the start time. Once you have the reservation name, you can submit jobs to it. They stay pending until the reservation begins and then start right away. This is the easiest way to make sure your reservation is in use from the first minute.

    While waiting, `squeue --me` shows the job as pending (`PD`) with the reason `Reservation`. Make sure `--time` ends before the reservation's end time, measured from the reservation start rather than from when you submit.

### Batch jobs

Add these options to your batch script:

```bash
#SBATCH --partition=temp-reservation
#SBATCH --reservation=<reservation_name>
#SBATCH --nodes=<N>
#SBATCH --exclusive
#SBATCH --mem=0
#SBATCH --time=<D-HH:MM:SS>
```

| Option | What it does |
|---|---|
| `--partition=temp-reservation` | Sends the job to the partition used for reservations. |
| `--reservation=<reservation_name>` | Runs the job on your reserved nodes. |
| `--nodes=<N>` | Number of nodes for the job. Must not be more than the reservation holds. |
| `--exclusive` | Gives the job all CPUs and GPUs on each reserved node. |
| `--mem=0` | Gives the job all of each node's memory. Without it, the job gets only a small default amount of memory. |
| `--time` | Maximum run time. See [Time limit](#time-limit). |


### Time limit

The usual 24-hour limit does not apply in the `temp-reservation` partition. The only limit is the end of the reservation, so you can run a single long job and do not need to chain jobs.

To set `--time`, check `EndTime` with `scontrol show reservation <reservation_name>`, and pick a value that ends the job before then, with some margin to write final output, e.g. `1-12:00:00` for 36 hours.

!!! warning "Jobs are stopped at the end of the reservation"

    Any job still running when the reservation ends is terminated. Always set `--time` so your job finishes first.

### Interactive sessions

For an interactive session on the reserved nodes, use the same options with `srun`:

```bash
srun -p temp-reservation --reservation=<reservation_name> -N <N> --exclusive --mem=0 --time=01:00:00 --pty /bin/bash
```

Interactive sessions end if your connection drops, so use a batch script for long runs.

### Job size and resubmission

Your job must fit in `<N>` nodes (`8 × <N>` GPUs).

!!! warning "Jobs that are too large never start"

    A job that asks for more nodes than the reservation holds cannot start, and will wait for the whole reservation.

If your script resubmits itself, include all the options above on the resubmission too, and stop the chain before the reservation ends.

### Monitoring

To check your jobs:

```bash
squeue --me
```
To see the reservation's details:

```bash
scontrol show reservation <reservation_name>
```

### Checkpointing

!!! tip "Save checkpoints regularly"

    Reservations are best-effort and not guaranteed against system or hardware failures. Saving checkpoints at regular intervals helps keep the work you've done in the reservation.

