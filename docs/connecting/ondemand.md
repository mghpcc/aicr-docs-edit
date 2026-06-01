---
tags:
  - Getting Started
  - OnDemand
---

# OnDemand Web Portal

Open OnDemand provides browser-based access to AICR. You can manage files, open a terminal, and launch interactive applications, all without installing SSH or configuring keys.

Access OnDemand at **<https://ood.aicr.ai>**.

## Logging In

1. Open the [AICR OnDemand portal URL](https://ood.aicr.ai) in your web browser.
2. Select your institution from the login page.
3. Authenticate with your institutional credentials (the same username and password you use at your university).

No separate AICR password or SSH keys are needed for OnDemand access.

## Dashboard

After logging in, the OnDemand dashboard provides access to:

- **Files**: browse, upload, download, edit, and manage files on the cluster
- **Jobs**: view active jobs, submit new jobs, access job output
- **Shell**: open a terminal session on a login node directly in your browser
- **Interactive Apps**: launch applications like Jupyter notebooks and remote desktop

## File Manager

Select **Files > Home Directory** from the navigation bar to open the file manager.

From here you can:

- **Navigate** through your Home, Scratch, and Work directories
- **Upload** files by dragging them from your desktop into the browser
- **Download** files by selecting them and clicking Download
- **Edit** text files in the built-in editor (with syntax highlighting and line numbers)
- **Create** new files and directories
- **Move, rename, and delete** files

!!! tip
    The file manager is the easiest way to transfer small files. For large datasets or many files, use [Globus](../files/transferring-data.md).

## Terminal (Shell Access)

Select **Clusters > Shell Access** to open a terminal session in your browser. This gives you the same command-line access as SSH, running on a login node.

!!! warning
    The same login node resource limits apply in OnDemand shell sessions: 4 CPU cores, 8 GB memory. Do not run computation here, instead submit jobs through Slurm.

## Interactive Applications

OnDemand launches interactive applications as Slurm jobs on compute nodes. Available applications:

- **JupyterLab**: interactive notebooks for Python, R, or Julia
- **Remote Desktop (XFCE)**: a full Linux desktop session for GUI applications

Interactive OOD sessions are limited to **4 hours** and run on the `*-devel` and `cpu` partitions only. You can have up to **4 concurrent interactive sessions** (shared limit with `salloc` jobs).

To launch an interactive app:

1. Select the application from the **Interactive Apps** menu
2. Specify resources (partition, number of GPUs, memory, time)
3. Click **Launch**
4. Wait for the job to start, then click **Connect** to open the session

Your session runs as a Slurm job and counts against your fairshare allocation.

## See Also

- [SSH](ssh.md) — terminal access for users who prefer command-line SSH
- [Transferring Data](../files/transferring-data.md) — other methods for moving files
<!-- 
TODO: remove comment when VSCode instructions are added.
- [VS Code Remote](vscode.md) — IDE-based development over SSH
-->