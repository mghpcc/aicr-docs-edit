# Maintenance

AICR reserves a recurring monthly window for system maintenance. This may include firmware and driver updates, Slurm and software upgrades, storage and network work, and other upkeep that keeps the cluster reliable and secure. Planning around a predictable window lets us apply changes safely with minimal disruption to research.

## Monthly maintenance schedule

Scheduled maintenance takes place on the **third Tuesday of each month**.

- **Start:** 5:00 AM ET  
- **Target end:** 5:00 PM ET — we make reasonable efforts to return the system to service by 5:00 PM, and often sooner.

Any change to a specific window (date, duration, or scope) will be announced in advance through the channels below.

## What to expect during maintenance

- **Logins are unavailable.** OnDemand and SSH access to login nodes are offline for the duration of the window.  
- **No jobs run.** The scheduler does not start new jobs during maintenance.  
- **Storage may be offline.** Home, scratch, and work filesystems can be unavailable while storage or network maintenance is performed.

Not every window affects every service. When maintenance is limited in scope, that is noted in the advance announcement.

## How jobs are handled

Ahead of each window, a Slurm reservation is placed to ensure no job runs during the maintenance period.

If a job's requested wall time would extend into the maintenance start, Slurm will **not** start it until after maintenance completes. The job stays pending with a reason such as `ReqNodeNotAvail` or `(ReservedForMaintenance)`. This is expected — the job will start once the system is back in service.

To check whether a pending job is waiting on the maintenance reservation:

```shell
squeue -u $USER --start
```

## Preparing for maintenance

1. **Right-size wall time.** Request only the time your job needs. A job asking for more time than remains before the window will wait until after maintenance. Use the `--time` flag to specify a time limit.
2. **Checkpoint long-running work.** Save intermediate state so multi-day work can resume cleanly after the window rather than restarting.  
3. **Plan submissions around the window.** If you need results before maintenance, submit early enough that the job can finish before 5:00 AM ET on the maintenance day.  
4. **Save unsaved work in interactive sessions.** OnDemand and interactive jobs end when the window begins.

## Emergency and unscheduled maintenance

Occasionally urgent issues, such as a security patch or a hardware failure, require maintenance outside the monthly window. 

## Staying informed

Maintenance reminders and per-window details (scope, affected services, and any early return to service) are communicated through:

- Announcements from the AICR team or your [institutional support team](http://getting-help.md)
- The AICR message of the day (MOTD) shown at login and an Open OnDemand banner message