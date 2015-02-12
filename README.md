Occasional rsnapshot
====================

This is a tool that allows automatic backups using rsnapshot
when the external backup drive is connected.

Although the ideal setup would be to have periodic backups on
a system that is always online, this is not always possible.
But when the connection is done, the backup should start fairly
quickly and should respect the daily/weekly/... schedules of
rsnapshot so that it accurately represents history.

So if you find yourself in a simillar situation, this script
might help you to do easily do sparse backups, instead of none.


Usage
-----

    occasional_rsnapshot [--debug] /path/to/rsnapshot.conf

The script tries to find `snapshot_root` based on the
information in the configuration file, then, if found, it
starts the appropriate backups that are detected as
necessary, for all the enabled intervals.

If the script does not find `snapshot_root`, it will
terminate silently, since it assumes the drive is not
connected.

Run via cron
------------

Copy the script to an appropriate place and add a line
simillar to this to your crontab for automatic running:

    */5 *   *   *  *    /home/eddy/bin/occasional_rsnapshot /home/eddy/etc/rsnapshot/local.conf

This will run the script every 5 minutes.

The script is smart enough to exit silently when appropriate:
 * when other instance of rsnapshot for the same config file
   is running
 * when the drive is not connected
 * when the time passed since the last backup is not above
   the minimum interval set by the rsnapshot configuration
   file

This means that the script does the right thing at all times,
even if called repeatedly.

Notes
-----

Because the script is silent by default when no drive is
found, it makes it appropriate for placing in cron with a
very high frequency as a means of triggering backups by just
connecting the backup drive; no noise will be made in the
cron emails when the script does not find the backup drive
connected.

When passing '--debug' the script is verbose and does not
invoke rsnapshot at all.

Versions
--------
This project uses [Semantic Versioning](http://semver.org/)

For short, here is what that means:

Given a version number MAJOR.MINOR.PATCH, increment the:

    MAJOR version when you make incompatible API changes,
    MINOR version when you add functionality in a backwards-compatible manner, and
    PATCH version when you make backwards-compatible bug fixes.

