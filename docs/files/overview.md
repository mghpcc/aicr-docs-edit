# Filesystem Overview

AICR Storage is separated into two categories: User storage and Project storage. Each user has both a **Home** and **Scratch** directory, and each Project has a shared **Work** directory. Each of these have a different purpose, size, and characteristics. All AICR storage is fast flash storage.

AICR storage is meant to be ephemeral. Data should only exist on AICR when it is needed for active compute projects.

- **Home**: Your Home Directory is meant for your most important files, has snapshots. We recommend keeping your software and code in your home directory.
- **Scratch**: Scratch space is meant for data used in actively running jobs. Scratch is meant for temporary files and has a 30-day retention policy. Files older than 30 days will be purged. Scratch is **not backed up** and **does not have snapshots** should not be used for long term storage.
- **Work**: Work is shared project space where the bulk of your project data should live. This storage is longer-term than Scratch, but should be cleared up at the end of the project lifecycle. Work also has snapshots. Each individual institution has its own process for determining the size of the Work space for each project.

!!! warning  "AICR Storage is not Backed up"
    All storage on AICR is **not backed up** outside of snapshots in Home and Work. Any files that cannot be easily replaced should be backed up outside of AICR.

See the table below for a description of each storage space.

| Storage Type  | <div style="width:18em">Path</div> | Quota | Backed up | Purpose/Notes |
| ----------- | ----------- |----------- |----------- |----------- |
| Home Directory | `/home/<username>` | 100 GiB | 7 Day Snapshots | Use for important files and software |
| Scratch | `/scratch/<username>/` | 10 TiB | **No Snapshots** | Scratch space for temporary files |
| Work | `/work/<institution>/<groupname>/` | Varies | 7 Day Snapshots | Shared project space |
