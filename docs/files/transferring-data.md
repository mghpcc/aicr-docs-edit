# Transferring data

There are a few different ways to transfer files depending on your goals, the data you are transferring, and what you are comfortable with. On this page we cover the different methods of transferring files, as well as touch on how to transfer files between systems.

We recommend using [OnDemand](#ondemand) for every-day file transfer and [Globus](#globus) for transferring large files or large numbers of files. For those who prefer to use the command line you can use [scp or rsync](#command-line).

## OnDemand

AICR has an [Open OnDemand Portal](https://ood.aicr.ai).

With the AICR OnDemand portal you can do the following using the File Browser:

- Upload and download files and directories
- Rename, move, and delete files and directories
- View and edit files

Once you are logged into OnDemand, you can use the File Browser by selecting Files -> Home Directory in the menu bar at the top left of the page.

To upload and download you can drag and drop files and directories between your File Browser window and your desktop Finder/Explorer windows. You can also use the "Upload" and "Download" buttons. Select multiple files by holding the Control (or Command) key and clicking on the files you'd like to select. Those files can then be downloaded with the "Download" button.

### Viewing and Editing Files

The File Browser is also the easiest way to view and edit files in place. Click on the file that you would like to view or edit. Then click either the "View" or "Edit" buttons directly above the list of files.

The editor has some nice features like line numbers and syntax highlighting for most languages. You can also change the display colors, the font size, and whether you'd like the words to wrap. Click "Save" to save any edits you made.

## Globus

If you are moving more than a few files, or your files are particularly large, we recommend using Globus. Globus is a tool that helps transfer large amounts of data between systems. We have a Globus collection set up on AICR. Collections are the mechanism Globus uses for accessing data.

Some advantages of using Globus are:

- It is easy to initiate transfers through the Globus webpage
- You don't need to stay logged into Globus through the entire transfer
- If your transfer is interrupted it will continue automatically where it left off once the connection is re-established

The AICR collection on Globus is called AICR Collection.

!!! tip
    The AICR Globus collection is not yet set up. When it is set up we will update this page.

 <!-- TODO: Replace Globus URL and add table
 Below is a table listing the collections for a few institution's home cluster.

| Institution  | Cluster | Globus Collection | 
| ----------- | ----------- |----------- |
| MA AI Hub | AICR | AICR Collection |
| MIT | Engaging | [MIT ORCD Engaging Collection](https://app.globus.org/file-manager?origin_id=ec54b570-cac5-47f7-b2a1-100c2078686f) |

 -->

To transfer data:

1. **Log in:** Log into [Globus](https://www.globus.org/) with your institution's credentials. These should be the same that you use to log into [ood.aicr.ai](https://ood.aicr.ai).
2. **Select your source and destination collections:** In the "File Manager" tab in each of the two "Collection" boxes search for the collections for the systems you want to transfer data between (AICR Collection for AICR). To transfer data to or from your own computer you will need to set up Globus Connect Personal. Follow the instructions on the page for your system listed [here](https://docs.globus.org/globus-connect-personal/).
3. **Navigate to your source and destination directories:** On the source side navigate to the source directory and select the files and/or directories you'd like to transfer. On the destination side navigate to the location where you'd like to copy your files
4. **Select any additional settings:** Click on "Transfer and Timer Options" for additional settings, such as syncing new or changed files and scheduling recurring transfers.
5. **Initiate the transfer:** Once you have selected the files and options you want, press the "Start" button on the source column.
6. **Monitor your transfer (optional):** Click on "Activity" to view the status of your active transfers. You will also get an email when you transfer is complete, or if Globus runs into any issues with the transfer.

The Globus Documentation has a [tutorial](https://docs.globus.org/guides/tutorials/manage-files/transfer-files/) with screenshots to demonstrate this process.

More documentation on transferring files through Globus can be found on the [Globus Documentation Pages](https://docs.globus.org/guides/tutorials/manage-files/transfer-files/). Globus also has an [FAQ](https://docs.globus.org/faq/transfer-sharing/) that is helpful for answering any questions you might have.

## Command Line

The most common commands used to transfer files at the command line are `scp` and `rsync`. You will need to run both of these commands from your local computer (or other system you'd like to transfer files from) before logging into AICR. In order to use these two commands you will need:

- The full path on the remote machine where you are copying the file to or from
- The ability to ssh to the remote machine where you are transferring files to or from

Both `scp` and `rsync` work similar to `cp`, in that you specify a source (where the file is coming from) and destination (where the file is going to). The main difference is that you will need to specify the hostname of the remote system.

<!-- TODO: Replace AICR DTN hostname -->

```bash
# using cp to copy files within the same system
cp /path/to/source /path/to/destination

# using scp to copy files from the local system to AICR
scp /path/to/source USERNAME@login.aicr.ai:/path/to/destination

# using scp to copy files from AICR to the local system
scp USERNAME@login.aicr.ai:/path/to/source /path/to/destination
```

Unless you have your paths memorized, the easiest way to do this is to have two terminals open. The first terminal is logged into AICR or other remote system, the second is on your local computer. In each navigate to the respective source and destination directories. In the AICR tab you can run the `pwd` command to print out the path to your current location and copy the output to use in the `scp` or `rsync` command.

### scp

First, open a terminal on your computer (not logged into AICR).

To transfer a file from your local computer to AICR you would use the command:

<!-- TODO: Replace AICR DTN hostname -->

``` bash
scp <local-file-name> USERNAME@login.aicr.ai:<path-to-aicr-dir>
```

For example, let's say you have the local file `myscript.py` and you want to transfer it to the directory `mycode` in your home directory. The command would be:

``` bash
scp myscript.py USERNAME@login.aicr.ai:/home/USERNAME/mycode/
```

To transfer the other direction (from AICR to your local computer) switch the order:

``` bash
scp USERNAME@login.aicr.ai:<path-to-aicr-file> <path-to-local-dir>
```

If you were to have the file `results.csv` that you want to copy from the `output` directory in your remote home directory to the current directory on your computer the command would be:

``` bash
scp USERNAME@login.aicr.ai:/home/USERNAME/output/results.csv .
```

Note the `.` in the command above means the current directory.

Similar to the `cp` command, if you want to transfer an entire directory and all of its subdirectories, use the `-r` (recursive) flag for either direction:

``` bash
scp -r <local-directory-name> USERNAME@login.aicr.ai:<path-to-aicr-dir>
```

<!-- TODO: Check 2FA behavior with AICR DTN -->

!!! note
    To `scp` files to/from the login nodes on AICR, you will need to
    authenticate with Duo. You may get a Duo push without any indication from the command line.

### rsync

The use of `rsync` is very similar to `scp`, but the behavior is different. By default `rsync` will not transfer files that are identical at both the source and destination. There are additional flags you can use to specify what `rsync` should do when files differ. The `rsync` command can be very useful when you want to "sync" updates to a directory or when transferring large directories. If a transfer fails during `rsync` you can re-run the command and it will pick up where it left off, rather than re-transfer everything.

For general use, the example commands above for `scp` apply, use the same command but replace `scp` with `rsync`.

Some useful flags include:

- `-r`, `--recursive` to recursively copy files in all sub-directories
- `-l`, `--links` to copy and retain symbolic links
- `-u`, `--update` skips any files for which the destination file already exists and has a date later than the source file
- `-v`, `--verbose` prints out more information during the file transfer, add more `v`s for more information
- `--partial` keeps partially transferred files, useful when transferring large files so rsync can continue where it left off if the transfer fails
- `--progress` prints information about the progress of the transfer
- `-n`, `--dry-run` does not run the transfer but prints out what actions it would be taken, useful to avoid unintended file overwrites

You can run `rsync --help` to print out a full list of flags that can be used with the `rsync` command.

!!! note
    To `rsync` files to/from the login nodes on AICR, you will need to
    authenticate with Duo. You may get a Duo push without any indication from the command line.

### Moving files between AICR and another Cluster

If you need to move files between AICR and other system first ssh to either and then initiate the transfer from that system to the other. Once you are logged into one system the process is the same as if you were to transfer files to or from your own computer. For example to move a file from the MIT Engaging cluster to AICR using `scp` you would first log into your local cluster and then use `scp` to transfer the file:

<!-- TODO: Replace DTN hostname -->

```bash title="Transferring files from MIT Engaging to AICR"
ssh USERNAME@orcd-login.mit.edu
scp <path-to-Engaging-file> USERNAME@login.aicr.ai:<path-to-AICR-directory>
```
You can also `ssh` into AICR and initiate the transfer from there using a similar command.

## Graphical Applications for File Transfer

There are a few applications you can download that will allow you to transfer files with  drag-and-drop, similar to how you would move files around on your own computer.

Some of the most common options are:

- [Cyberduck](https://cyberduck.io/) (Mac and Windows)
- [FileZilla](https://filezilla-project.org/) (Mac, Windows, and Linux)
- [WinSCP](https://winscp.net/eng/index.php) (Windows only)

<!-- TODO: Replace DTN hostname -->

To use these you will need to use the hostname of the AICR data transfer nodes: `login.aicr.ai`.