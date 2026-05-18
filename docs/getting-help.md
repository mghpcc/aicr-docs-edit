---
tags:
  - Support
---

# Getting Help

AICR support is provided through the research computing teams at each participating institution. Contact your institution's team for help with accounts, jobs, software, and general AICR questions.

## Institutional Support Teams

| Institution | Support Team | Contact |
|-------------|-------------|---------|
| Boston University | [Research Computing Services (RCS)](https://www.bu.edu/tech/support/research/) | help@rcs.bu.edu |
| Harvard | TBD | TBD |
| MIT | Office of Research Computing and Data (ORCD) | [MIT ORCD](https://orcd.mit.edu/) | orcd-help-aicr@mit.edu |
| Northeastern | Research Computing (RC) | [NEU RC](https://rc.northeastern.edu/) |  rchelp@northeastern.edu | 
| UMass | [Unity Research Computing Platform](https://unityhpc.org/) | hpc@umass.edu |
| URI | [Unity Research Computing Platform](https://unityhpc.org/) | hpc@umass.edu |
| Yale | [Yale Center for Research Computing (YCRC)](https://research.computing.yale.edu/) | research.computing@yale.edu |

!!! tip
    Your institutional team is the best first point of contact. They know the AICR system and can also help with institution-specific policies, accounts, and allocations.

## How to Write a Good Help Request

A clear, detailed help request gets you a faster answer. Include the following information when contacting support:

### Essential Information

1. **Your username and institution.** Support staff need this to look up your account and jobs.
2. **What you were trying to do.** A brief description of your goal (e.g., "run a multi-GPU PyTorch training job on the B200 partition").
3. **What happened.** Describe the error or unexpected behavior. Include the exact error message, not a paraphrase.
4. **Job ID.** If the problem involves a Slurm job, include the job ID. Support staff can look up everything else from there:

    ```bash
    $ sacct -j JOBID --format=JobID,JobName,Partition,State,ExitCode,Elapsed
    ```

5. **Steps to reproduce.** List the commands you ran, in order. If possible, provide a minimal example that reproduces the problem.

### Helpful Extras

- **Job script.** Attach or paste the full Slurm job script (`#SBATCH` directives and all commands).
- **Output and error files.** Attach the `.out` and `.err` files from the job.
- **Software environment.** Include the output of `module list` and `conda list` or `pip freeze`.
- **What you already tried.** Mention any troubleshooting steps you took so support does not duplicate effort.

### Example Help Request

> **Subject:** Job JOBID fails with CUDA out of memory on rtx-batch
>
> Hi, I am < USERNAME > from < INSTITUTION >. I am trying to fine-tune a language model on the rtx-batch partition, but my job fails after about 30 minutes with:
>
> `RuntimeError: CUDA out of memory. Tried to allocate 2.00 GiB`
>
> **Job ID:** 456789
> **Partition:** rtx-batch
> **GPUs requested:** 1
> **Memory requested:** 32G
>
> I have tried reducing the batch size from 32 to 16 but the error persists. My job script and output file are attached.

## Before Contacting Support

Try these self-service steps first:

1. **Check this documentation.** Many common issues are covered in the docs.
2. **Check job output files.** Read your `.out` and `.err` files for error messages.
3. **Check your quota.** Jobs fail when storage is full

<!-- TO DO: Update when we have paths -->
<!-- ```bash
$ df -h /home/$USER
$ df -h /scratch/$USER
``` -->

4. **Check job status.** See why a job is pending or failed:

    ```bash
    $ squeue -u $USER
    $ sacct -j JOBID --format=JobID,State,ExitCode,Reason
    ```

5. **Search for error messages.** Copy the exact error text and search online. Many CUDA, PyTorch, and Slurm errors have well-documented solutions.

<!-- ## See Also -->

<!-- - [FAQ](faq.md) -- answers to common questions
- [Login Troubleshooting](login-troubleshooting.md) -- SSH and OnDemand connection problems
- [Job Troubleshooting](../running-jobs/troubleshooting.md) -- diagnosing job failures
- [Monitoring Jobs](../running-jobs/monitoring.md) -- checking job status and resource usage -->
