---
tags:
  - Support
---

# Getting Help

AICR support is provided through the research computing teams at each participating institution. Contact your institution's team for help with accounts, jobs, software, and general AICR questions.

## Institutional Support Teams

| Institution | Support Team | Contact |
|-------------|-------------|---------|
| Boston University | Research Computing Services (RCS) | [BU RCS](https://www.bu.edu/tech/support/research/) |
| Harvard | FAS Research Computing (FASRC) | [FASRC](https://www.rc.fas.harvard.edu/) |
| MIT | Office of Research Computing and Data (ORCD) | [MIT ORCD](https://orcd.mit.edu/) |
| Northeastern | Research Computing (RC) | [NEU RC](https://rc.northeastern.edu/) |
| UMass | Unity Research Computing Platform | [Unity](https://unityhpc.org/) |  
| URI | Unity Research Computing Platform | [Unity](https://unityhpc.org/) | 
| Yale | Yale Center for Research Computing (YCRC) | [YCRC](https://research.computing.yale.edu/) |

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
---
tags:
  - Support
  - SSH
  - OnDemand
  - Troubleshooting
---

# Troubleshooting Login Issues

This page covers common problems connecting to AICR via SSH and OnDemand, and how to resolve them.

## SSH Problems

### Permission Denied (publickey)

```
Permission denied (publickey).
```

AICR uses SSH certificate authentication. This error means the server rejected your certificate. Check the following:

**1. Certificate files are in place.**

You should have three files in `~/.ssh/`:

```bash
$ ls ~/.ssh/id_ed25519_aicr*
id_ed25519_aicr          # private key
id_ed25519_aicr.pub      # public key
id_ed25519_aicr-cert.pub # CA-signed certificate
```

If any are missing, regenerate your certificate through the OnDemand portal at `ood.aicr.ai`, or contact your institution's RC team.

**2. Correct file permissions.**

```bash
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/id_ed25519_aicr
$ chmod 644 ~/.ssh/id_ed25519_aicr.pub
$ chmod 644 ~/.ssh/id_ed25519_aicr-cert.pub
```

**3. Correct username.**

```bash
$ ssh USERNAME@login.aicr.ai
```

Your AICR username will differ from your institutional username — it follows the format `institutionalusername_institutioncode` (e.g., `jsmith_mit`). It was provided when your account was created. Double-check spelling and case.

**4. Certificate not expired.**

Check your certificate's validity period:

```bash
$ ssh-keygen -L -f ~/.ssh/id_ed25519_aicr-cert.pub
```

Look for the `Valid:` line. If the certificate has expired, regenerate it through the OnDemand portal.

**5. SSH config points to the right files.**

Your `~/.ssh/config` should reference the AICR certificate:

```
Host aicr
    HostName login.aicr.ai
    User USERNAME
    IdentityFile ~/.ssh/id_ed25519_aicr
    CertificateFile ~/.ssh/id_ed25519_aicr-cert.pub
```

**6. SSH config file does not have an extension**

Some operating systems will append a file extension to newly created files by default. From the command line, verify your config file has no extension with `ls ~/.ssh/`.


**7. Verbose output for diagnosis.**

```bash
$ ssh -vvv USERNAME@login.aicr.ai
```

Look for lines containing `Offering public key` and `Server accepts key` to see which credentials are being tried.

### Connection Timed Out

```
ssh: connect to host login.aicr.ai port 22: Connection timed out
```

**1. Check your network connection.** Verify you can reach external hosts:

```bash
$ ping -c 3 login.aicr.ai
```

**2. Check for firewall restrictions.** AICR login nodes are publicly accessible — no VPN is required. However, some institutional or corporate networks block outbound SSH (port 22). If you are on a restricted network:

- Try from a different network (home, mobile hotspot) to isolate the problem.

**3. Check AICR status.** The cluster may be undergoing maintenance. Check with your [institutional support team](getting_help.md#institutional-support-teams) for scheduled downtime.

### Connection Refused

```
ssh: connect to host login.aicr.ai port 22: Connection refused
```

The server is reachable but not accepting connections on port 22. This usually indicates a system-wide issue or maintenance. Wait and retry, or contact your institutional support team.

### Host Key Verification Failed

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

This means the server's SSH host key does not match the one stored in your `~/.ssh/known_hosts` file. This can happen after AICR maintenance that replaces login nodes.

!!! warning
    Do not blindly remove the old key. Verify with your institutional support team that the host key has legitimately changed before proceeding.

If the key change is expected (confirmed by support), remove the old entry:

```bash
$ ssh-keygen -R login.aicr.ai
```

Then reconnect and accept the new key:

```bash
$ ssh USERNAME@login.aicr.ai
```

### Connection Drops or Freezes

If your SSH session freezes or disconnects after a period of inactivity, configure keepalive in your SSH client config:

Create or edit `~/.ssh/config`:

```
Host aicr
    HostName login.aicr.ai
    User USERNAME
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

Then connect with:

```bash
$ ssh aicr
```

## OnDemand Issues

### Cannot Reach the OnDemand Portal

- Verify the URL is correct: **<https://ood.aicr.ai>**
- Clear your browser cache and cookies, then try again.
- Try a different browser or incognito/private window.
- AICR OnDemand does not require a VPN. If you still cannot reach it, check your local network or try a different browser.

### Login Succeeds but Sessions Fail to Start

If you can log in to OnDemand but interactive sessions (Jupyter, terminal, desktop) fail to launch:

- **Check your home directory quota.** A full home directory prevents OnDemand from writing session files:

    ```bash
    $ df -h /home/$USER
    ```

- **Check for stale session files.** Old session data can interfere with new sessions. Connect via SSH and clean up:

    ```bash
    $ rm -rf /home/$USER/ondemand/data/sys/dashboard/batch_connect/sys/*
    ```

- **Check Slurm.** OnDemand sessions run as Slurm jobs. If the cluster is busy, your session may be queued. Check with:

    ```bash
    $ squeue -u $USER
    ```

### Authentication Loops

If the login page keeps redirecting you back to itself without completing authentication:

1. Clear all cookies for the OnDemand domain.
2. Close all browser tabs with OnDemand open.
3. Open a fresh browser window and try again.
4. If the problem persists, try a different browser.

## Two-Factor Authentication (2FA)

2FA applies to **OnDemand web portal login** through your institution's identity provider. If your institution requires 2FA (Duo, TOTP, etc.), you will be prompted during the OnDemand login flow.

**SSH access does not require 2FA.** SSH uses short-lived CA-signed certificates — your certificate is your credential.

If you have trouble with 2FA during OnDemand login:

- **Make sure your 2FA device is synchronized.** Time-based one-time passwords (TOTP) require your device clock to be accurate.
- **Use the correct 2FA method.** Check with your institutional support team whether they use push notifications (Duo) or TOTP codes.
- **Do not reuse codes.** Each TOTP code is valid for a single use and a short time window.

<!-- TODO: 2FA for OOD login is handled by institutional identity providers, not by AICR directly. SSH certificate auth does not use 2FA initially. This is not fully finalized — verify when deployed. -->

## Diagnostic Checklist

If you are still stuck, gather this information before contacting support:

| Item | Command |
|------|---------|
| SSH verbose output | `ssh -vvv USERNAME@login.aicr.ai 2>&1 \| head -80` |
| Your public key fingerprint | `ssh-keygen -lf ~/.ssh/id_rsa.pub` |
| Your local OS and SSH version | `ssh -V` |
| Network path | `traceroute login.aicr.ai` |
| DNS resolution | `nslookup login.aicr.ai` |

