---
tags:
  - Getting Started
  - Linux
---

# Getting Started Tutorial

This page contains the most common steps for setting up and getting
started with your AICR account. We provide this page as a
convenient reference to get started.

## Getting an Account

AICR is currently available to initial alpha and beta-test users. Each institution may have its own process for getting an account. When it is available we will include links to that information here.

## Logging In

The first thing you should do when you get a new account is verify that
you can log in. You can log in using either the [AICR OnDemand portal](connecting/ondemand.md) or through [SSH in the terminal](connecting/ssh.md).

### OnDemand Portal

You can log into OnDemand Web Portal using your institutional credentials. Access AICR OnDemand through: [https://ood.aicr.ai](https://ood.aicr.ai).

### Terminal with SSH

Log into AICR with the following command in a terminal window. See the [SSH Login page](connecting/ssh.md) for how to set up SSH access. Once you have set up your SSH keys as instructed, you can log in with the following command:

```bash
ssh USERNAME@login.aicr.ai
```

Replace `USERNAME` above with your AICR username. Your AICR username is similar to but not the same as your institutional username. It was given to you when your account was created.

## Shared HPC Clusters

AICR is a shared HPC cluster. You are sharing this
resources with a number of other researchers, staff, and students, so it
is important that you read this page and use the system as intended.

Being a cluster, there are several machines connected together with a
network. We refer to these as **nodes**. Most nodes in the cluster are
referred to as **compute nodes**, this is where the computation is done
on the system (where you will run your code). When you ssh into the
system you are on a special purpose node called the **login node**. The
login node, as its name suggests, is where you log in and is for editing
code and files, installing packages and software, downloading data, and
starting jobs to run your code on one of the compute nodes.

Each job is started using a piece of software called the **scheduler**,
which you can think of as a resource manager. You let it know what
resources you need and what you want to run, and the scheduler will find
those resources and start your job on them. When your job completes
those resources are relinquished. The scheduler is what ensures that no
two jobs are using the same resources, so it is very important not to
run anything unless it is submitted properly through the scheduler.

## Software and Packages

The first thing you may want to do is make sure the system has the
software and packages you need. We have installed a lot of software and
packages on the system already, even though it may not be immediately
obvious that it is there. AICR uses modules to manage software, which should be similar to your home institution's system. AICR as a common set of core modules shared across institutions, as well as institution-specific modules whose naming and layout may be more familiar to you. Institution-specific modules are available to all AICR users, but may be hidden by default.

Run `module avail` to see the default set of modules available to you. You can load a module with the `module load` command. Please see the [Software](software.md) page for more information.

<!-- TODO: How to view modules for a different institution -->

If you are ever unsure if we have a particular
software, and you cannot find it, you can try installing it in your home directory. If you need help, please send us an email and ask before
you spend a lot of time trying to install it. If we have it, we can
point you to it, provide advice on how to use it, and if we don't have
it we can often give pointers on how to install it. Further, if a lot of
people request the same software, we may consider adding it as a module.

## Linux Command Line

AICR runs Linux, so much of what you do on the cluster
involves the Linux command line. That doesn't mean you have to be a
Linux expert to use the system! However the more you can get comfortable
with the Linux command line and a handful of basic commands, the easier
using the system will be. If you are already familiar with Linux, feel
free to skip this section, or skim as a refresher.

Most Linux commands deal with **directories** and **files**. A
**directory**, synonymous to a folder, contains files and other
directories. The list of directories that lead to a particular directory
or file is called its **path**. In Linux, directories on a path are
separated by forward slashes `/`. It is also important to note that
everything in Linux is case sensitive, so a file `myScript.sh` is not
the same as the file `myscript.sh`. When you first log in, you are in your
**home directory**. Your home directory is where you can put all the
code and data you need to run your job. Your home directory is not
accessible to other users, so if you need a space to share files with other
users, use a **project** directory.

The path to your home directory on AICR is `/home/<USERNAME>`, where `<USERNAME>` is your username. The character `~` is also shorthand for your home directory in any Linux commands.

Anytime after you start typing a Linux command you can press the "Tab"
button your your keyboard. This called tab-complete, and will try to
autocomplete what you are typing. This is particularly helpful when
typing out long directory paths and file names. Pressing "Tab" once
will complete if there is a single completion, pressing it twice will
list all potential completions. It is a bit difficult to explain in
text, but you can try it out yourself and watch the short demonstration
[here](https://en.wikipedia.org/wiki/Command-line_completion).

Finally, click on the box below for a list of Linux Commands. If you are new to Linux try them out for yourself at the command line.

??? "Common Linux Commands"

    -   Creating, navigating and viewing directories:
        -   `pwd`: tells you the full path of the directory you are
            currently in
        -   `mkdir dirname`: creates a directory with the name "dirname"
        -   `cd dirname`: change directory to directory "dirname"
            -   `cd ../`: takes you up one level
        -   `ls`: lists the files in the directory
            -   `ls -a`: lists all files including hidden files
            -   `ls -l`: lists files in "long format" including ownership
                and date of last update
            -   `ls -t`: lists files by date stamp, most recently updated
                file first
            -   `ls -tr`: lists files by dates stamp in reverse order, most
                recently updated file is listed last (this is useful if you
                have a lot of files, you want to know which file you changed
                last and the list of files results in a scrolling window)
            -   `ls dirname`: lists the files in the directory "dirname"
    -   Viewing files
        -   `more filename`: shows the first part of a file, hitting the
            space bar allows you to scroll through the rest of the file, q
            will cause you to exit out of the file.
        -   `less filename`: allows you to scroll through the file, forward
            and backward, using the arrow keys.
        -   `tail filename`: shows the last 10 lines of a file (useful when
            you are monitoring a log file or output file to see that the
            values are correct)
            -   `tail <number> filename`: show you the last &lt;number\>
                lines of a file.
            -   `tail -f filename`: shows you new lines as they are written
                to the end of the file. Press CMD+C or Control+C to exit.
                This is helpful to monitor the log file of a batch job.
    -   Copying, moving, renaming, and deleting files
        -   `mv filename dirname`: moves filename to directory dirname.
            -   `mv filename1 filename2`: moves filename1 to filename2, in
                essence renames the file. The date and time are not changed
                by the mv command.
        -   `cp filename dirname`: copies to directory dirname.
            -   `cp filename1 filename2`: copies filename1 to filename2. The
                date stamp on filename2 will be the date/time that the file
                was moved
            -   `cp -r dirname1 dirname2`: copies directory dirname1 and its
                contents to dirname2.
        -   `rm filename`: removes (deletes) the file

## Transferring Files

One of the first tasks is to get your code, data, and any other files
you need into your home directory on the system. Review the [Transferring Data](files/transferring-data.md) page for a full description of how to move data on and off AICR.

If your code is in GitHub you can use git commands on the system to clone your repository
to your home directory. The primary method we recommend for copying your files to and from AICR is through [Globus](https://www.globus.org/). You can also transfer your files from your computer using the OnDemand File Browser or by using the commands `scp` or `rsync`.

You can use `scp` or `rsync` from the command line on your local computer. Both commands work similarly to the `cp` command, following the pattern `<command> <source> <destination>`, the only difference being that you will need to include the hostname of the system you are transferring to or from. For this reason you *must* run this command from the terminal on your computer *before you've logged in*.

To transfer a file from your computer to AICR:

``` bash
scp <local-file-name> USERNAME@login.aicr.ai:<path-to-aicr-dir>
```

To transfer a file from AICR to your computer:

``` bash
scp USERNAME@login.aicr.ai:<path-to-aicr-dir>/<file-name> <path-to-local-dest>
```

Similar to `cp`, use the `-r` flag to copy over an entire directory and its contents. 

``` bash
scp -r <local-dir-name> USERNAME@login.aicr.ai:<path-to-aicr-dir>
```

The `rsync` command can be used similarly and has some additional flags you can use. It also can be used to transfer only new or modified files to the destination, which makes it easy to keep a directory in "sync".


## Running your First Job

At this point you may want to do a test-run of your code. You always
want to start small in your test runs, so you should choose a small
example that tests the functionality of what you would ultimately like
to run on the system. If your test code is serial and runs okay on a
moderate personal laptop or desktop you can request an interactive
session to run your code in by executing the command:

``` bash
# Requesting four cores and one RTX Pro 6000 GPU for an interactive job for 1 hour
salloc -t 01:00:00 -p rtx-devel -c 4 -G 1
```

After you run this command you will be on a compute node and you can do
a test-run of your code. This command will allocate four cores and one RTX Pro 6000 GPU to your
job. If your test code needs additional cores or uses a lot of
memory, you should request additional resources as needed.

We provide the basics of running jobs on the [Running Jobs](running-jobs/slurm-basics.md) page. For more information you can consult the documentation at your home institution.

## Getting Help

AICR is being supported by the staff at your home institution's research computing group. If you need help or have any questions you can reach out to them. See [Getting Help](getting-help.md) for more information.