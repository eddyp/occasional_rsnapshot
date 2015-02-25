Occasional rsnapshot
====================

This is a tool that allows automatic backups using rsnapshot
when the external backup drive or remote backup media is
connected.

Although the ideal setup would be to have periodic backups on
a system that is always online, this is not always possible.
But when the connection is done, the backup should start fairly
quickly and should respect the daily/weekly/... schedules of
rsnapshot so that it accurately represents history.

In other words, if you backup to an external drive or to some
network/internet connected storage that you don't expect to
have always connected (which is is case with laptops) you can
use `occasional_rsnapshot` to make sure your data is backed
up when the backup storage is connected.

`occasional_rsnapshot` is appropriate for:
 * laptops backing up on:
   * a NAS on the home LAN or
   * a remote or an internet hosted storage location
 * systems making backups online (storage mounted locally
   somehow)
 * systems doing backups on an external drive that is not
   always connected to the system

The only caveat is that all of these must be mounted in the
local file system tree somehow by any arbitrary tool,
`occasional_rsnapshot` or `rsnapshot` do not care, as long as
the files are mounted.


So if you find yourself in a simillar situation, this script
might help you to easily do backups in spite of the occasional
availability of the backup media, instead of no backups at all.
You can even trigger backups semi-automatically when you
remember to or decide is time to backup, by simply pulging in
your USB backup HDD.

Usage
-----

Install `rsnapshot` and make sure your rsnapshot configuration
file contains the stanza below so no backup is tried when the
backup media is not mounted:

    no_create_root	1

''Note: `rsnapshot.conf` uses tabs as separators between the
option and the value, so copying the line above from this file
into your `rsnapshot.conf` might not yeld the expected
results.''

Then run `occasional_rsnapshot` with the 'rsnapshot.conf'
file as parameter.

    occasional_rsnapshot /path/to/rsnapshot.conf

''Note: Optionally, if you want to check first what would be
done, you can pass the `--debug` option and see what decisions
`occasional_rsnapshot` would make and it will not call
`rsnapshot`.''

    occasional_rsnapshot --debug /path/to/rsnapshot.conf

`occasional_rsnapshot` tries to find `snapshot_root` based on
the information in the configuration file, then, if found, it
analyzes its contents and, based on the options in the
`rsnapshot.conf` file and the status of the `snapshot_root`,
it starts the appropriate backups that are detected as
necessary, for all the enabled intervals.

If the script does not find `snapshot_root`, it will terminate
silently, since it assumes the backup media is not connected.

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


Encryption and advanced setups
------------------------------

Since `rsnapshot` uses `rsync` for the backup and `rsync` does
not encrypt the files on the backup media, a way to implement
encrypted backups is to encrypt the source files which
`rsnapshot` backups.

You can use `encfs --reverse` to transparently obtain an
ecrypted version of your source filesystem. After that, change
your `backup` stanzas in `rsnapshot.conf` to do backups of the
encypted versions of your files instead of the originals.

Since the original filenames will not be available/visible to
`rsnapshot`, if you want to exclude specific files from the
backup, you will have to filter those files by other means
than the `exclude` `rsnapshot.conf` stanzas. This can be
achieved by a filtering filesystem such as the `filterfs` FUSE
filesystem (or, if you run GNU/Hurd, a filtering translator).


In essence, when using encfs and filterfs, the resulting setup
would be:

          Occasional Rsnapshot
                   |
                  \|/
               Rsnapshot
                   |
                  \|/
    +-------------------------------+
    |  Filtered and ecrypted files  |
    |           (encfs)             |
    +-------------------------------+
                   |
                  \|/
    +-------------------------------+
    |        Filtered files         |
    |          (filterfs)           |
    +-------------------------------+
                   |
                  \|/
    +-------------------------------+
    |        Original files         |
    +-------------------------------+


`encfs` is available in most distributions or can be
downloaded from https://github.com/vgough/encfs .

Unfortunately the `filterfs` fuse filesystem upstream doesn't
seem to be active anymore, but since I used this for this
exact setup, a more recent version with some fixes can be
grabbed from: https://github.com/eddyp/filterfs


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


Development
-----------

Development of `occasional_rsnapshot` happens on github at:

[https://github.com/eddyp/occasional\_rsnapshot](https://github.com/eddyp/occasional_rsnapshot)

Patches and pull requestst are welcome.

Versions
--------
This project uses [Semantic Versioning](http://semver.org/)

For short, here is what that means:

Given a version number MAJOR.MINOR.PATCH, increment the:

    MAJOR version when you make incompatible API changes,
    MINOR version when you add functionality in a backwards-compatible manner, and
    PATCH version when you make backwards-compatible bug fixes.

