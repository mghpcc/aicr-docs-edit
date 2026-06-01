---
tags:
  - Getting Started
  - SSH
---

# SSH

SSH (Secure Shell) is the standard way to access AICR from a terminal. If you prefer a browser-based interface with no setup, see [OnDemand](ondemand.md) instead.

AICR uses **SSH certificate authentication**. When your account is created, you receive a short-lived SSH certificate signed by AICR's Certificate Authority (CA). There is no separate AICR password. Your certificate is your credential.

You can download a new certificate through the AICR OnDemand portal when your account is created or your current certificate approaches expiration.

## Connecting

If this is your first time logging in through SSH, see below on how to [set up your certificate and ssh keys](#setting-up-your-ssh-keys-and-certificate). Once your keys are set up, open a terminal on your local computer and run:

```bash
$ ssh USERNAME@login.aicr.ai
```

Replace `USERNAME` with your AICR username. This follows the format `institutionalusername_institutioncode` (e.g., `jsmith_mit`) and was provided when your account was created.

!!! tip
    If this is your first time connecting, you will see a host key verification prompt. Type `yes` to continue. This only happens once per machine.

## Setting Up Your SSH Keys and Certificate

When your AICR account is created, three four files will be generated for you:

- `id_ed25519_aicr`: your private key (passphrase-protected)
- `id_ed25519_aicr.pub`: your public key
- `id_ed25519_aicr-cert.pub`: your CA-signed certificate
- `.passphrase`: initial passphrase for your private key

Retrieve these files from [https://ood.aicr.ai](https://ood.aicr.ai) (see the [OnDemand page](ondemand.md) page for more information on how to log in). Once you've logged in, go to the File Browser (Files -> Home Directory). Download the `aicr_keys` folder by clicking on the `...` icon and selecting "Download", or check the box next to the folder icon and click the "Download" button at the top.

Unzip the downloaded folder and place each these files in your `~/.ssh/` directory and set the correct permissions. 

=== "macOS / Linux"

    ```bash
    $ chmod 600 ~/.ssh/id_ed25519_aicr
    $ chmod 644 ~/.ssh/id_ed25519_aicr.pub
    $ chmod 644 ~/.ssh/id_ed25519_aicr-cert.pub
    ```

=== "Windows"

    **Windows Terminal / PowerShell**: Place the files in `C:\Users\YOUR_WINDOWS_USER\.ssh\`. SSH is built in — use the same `ssh` command from PowerShell or Windows Terminal.

    **WSL**: Place the files in `~/.ssh/` inside your WSL home directory (e.g., `/home/YOUR_LINUX_USER/.ssh/`). If you downloaded the files to Windows, copy them from `/mnt/c/Users/YOUR_WINDOWS_USER/Downloads/` into `~/.ssh/` in WSL. Then set permissions the same way as macOS/Linux:

    ```bash
    $ chmod 600 ~/.ssh/id_ed25519_aicr
    $ chmod 644 ~/.ssh/id_ed25519_aicr.pub
    $ chmod 644 ~/.ssh/id_ed25519_aicr-cert.pub
    ```

    Use the `ssh` command from your WSL terminal as you would on Linux.

Once the files are in place and permissions are set, update your passphrase with the following command:

```bash
ssh-keygen -p -f ~/.ssh/id_ed25519_aicr
```

Note that your initial passphrase is in the `.passphrase` file in the `aicr_keys` directory that you downloaded from OnDemand. Choose a passphrase that you will remember. If you forget your passphrase you can download the `aicr_keys` directory again and repeat this process.

### Certificate Renewal

Certificates have a limited validity period. When your certificate approaches expiration, regenerate it through the AICR OnDemand portal — log in, navigate to the SSH Certificate app, and download a new certificate. Replace your existing certificate with this new one.

## SSH Config File

For convenience, add AICR to your SSH config file (`~/.ssh/config`):

```
Host aicr
    HostName login.aicr.ai
    User AICR_USERNAME
    IdentityFile ~/.ssh/id_ed25519_aicr
    CertificateFile ~/.ssh/id_ed25519_aicr-cert.pub
```

Then connect with:

```bash
$ ssh aicr
```

## What You Can Do on Login Nodes

Login nodes are shared and have strict resource limits:

- **4 CPU cores** and **8 GB memory** per user
- Processes exceeding these limits will be terminated

!!! danger
    Do not run computation on login nodes. Use them for:

    - Editing code and files
    - Installing packages and managing environments
    - Submitting and monitoring jobs
    - Small file transfers

    For computation, submit a job with `sbatch` or request an interactive session with `salloc`. See [Slurm Basics](../running-jobs/slurm-basics.md).

## Connecting to Compute Nodes

You cannot SSH directly to a compute node. Instead:

1. Request resources [through Slurm](../running-jobs/slurm-basics.md#interactive-jobs)
2. Slurm places you on a compute node with the requested resources
3. When done, type `exit` to release the resources

If you have a running job, you can SSH to the assigned node to check on it:

```bash
$ squeue -u $USER          # find your node name
$ ssh NODE_NAME             # connect to the node (only works with an active job)
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `Permission denied (publickey)` | Your certificate may be expired or missing. Re-downlod it through the OnDemand portal, or contact your institution's [RC team](../getting-help.md). Check file permissions: private key should be `600` (see [above](#setting-up-your-ssh-keys-and-certificate)). |
| `Connection timed out` | Check your network connection. AICR login nodes are publicly accessible — no VPN is required. If the problem persists, check your local firewall or try from a different network. |
| `Host key verification failed` | The host key has changed. Remove the old entry: `ssh-keygen -R login.aicr.ai` then reconnect. |
| Connection drops frequently | Add `ServerAliveInterval 60` to your SSH config for AICR. |

## See Also

- [OnDemand Web Portal](ondemand.md) — browser-based access, no SSH setup needed
- [Getting Started Tutorial](../getting-started.md) — first steps on AICR
<!-- 
TODO: remove comment when VSCode instructions are added.
- [VS Code Remote](vscode.md) — IDE-based development over SSH
-->