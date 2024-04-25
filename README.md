### __wyng-util-qubes__

A wrapper for the [Wyng](https://github.com/tasket/wyng-backup) backup system that saves and
restores both data and settings for Qubes VMs.


### Requirements

* Qubes OS 4.2 in a thin-LVM configuration
* wyng-backup v0.8beta4


### Command usage

__wyng-util-qubes__ is run in the Admin VM (dom0):

```
 wyng-util-qubes backup [--dedup] [-i] [--pool=poolname] [qube_name...]
 wyng-util-qubes restore --session=YYYYMMDD-HHMMSS [--pool=poolname] [qube_name...]
 wyng-util-qubes verify --session=YYYYMMDD-HHMMSS [qube_name...]
 wyng-util-qubes prune [--autoprune=opt] [--all-before] [--session=YYYYMMDD-HHMMSS[,YYYYMMDD-HHMMSS]] [qube_name...]
 wyng-util-qubes delete <qube_name>
 wyng-util-qubes list [--session=YYYYMMDD-HHMMSS] [qube_name...]
```

### Command summary
| _Command_                     | _Description_
|-------------------------------|--------------
backup             | Store Qubes VMs in the Wyng archive as a session (i.e. snapshot)
restore            | Restore Qubes VMs from the Wyng archive
verify             | Verify archive data integrity
prune              | Remove older backup sessions from archive
delete             | Remove VMs from the Wyng archive
list               | Show contents of archive


### Parameters/Options summary

| _Option_                      | _Description_
|-------------------------------|--------------
--includes, -i         | Select all Qubes VMs marked as "include in backups". (backup)
--exclude=qube_name    | Exclude a specific VM from the operation (backup, restore, verify)
--dedup, -d            | Use deduplication. (backup)
--session=_date-time[,date-time]_ | Select a session or session range by date-time or tag (restore, list, prune).
--all                  | Show all VM names and backup sessions (list)
--all-before           | Select all sessions before the specified _--session date-time_ (prune).
--autoprune=<off|on|full>  | Automatic pruning by calendar date.
--dest=_URL_           | URL location of archive.
--pool=_qubespool_     | Override default 'vm-pool' Qubes local storage pool. (restore)
--pool-info            | Show local disk storage (list)
--local=_vg/pool_      | Deprecated. Use --pool instead.
--authmin=N            | Extend authentication time by N minutes
--unattended, -u       | Operate without prompts.
--meta-dir=_path_      | Use a different metadata dir than the default.
-w _wyng_option_spec_  | Pass an option directly to Wyng using the form `-w optname[=value]`


### Examples

```

$ # Start by creating a fresh Wyng archive:
$ sudo wyng arch-init --dest=file:/mnt/backups/laptop3.backup


$ # Make wyng backups of the VMs _work_ and _personal_
$ sudo wyng-util-qubes backup work personal --dest=file:/mnt/backups/laptop3.backup


$ # Restore VM _personal_ from a wyng archive
$ sudo wyng-util-qubes restore personal --dest=file:/mnt/backups/laptop3.backup

```

The above examples show the creation and use of an archive named 'laptop3.backup' located in
the local '/mnt/backups' path.  For non-local archives, use one of the other URL types supported
by Wyng, such as 'qubes://' for backing up to a VM filesystem, or 'qubes-ssh://' for backing up
to remote via an ssh-equipped VM...

```
$ sudo wyng arch-init --dest=qubes://sys-backup/mnt/backups/laptop3.backup

$ sudo wyng arch-init --dest=qubes-ssh://sys-backup:user@192.168.1.10/mnt/backups/laptop3.backup
```


### Commands

Note that `--dest` is assumed to be specified along with each of the commands below.

#### list

`list [vm name] [--session=YYYYMMDD-HHMMSS] [--all]`

Shows a directory of archive contents.  If no parameters are given, a list of qube / VM names
is given.


#### backup

`backup [vm names] [--includes] [--exclude=vmname] [--dedup] [--autoprune] `

Backs up VMs to a Wyng archive.  A list of invdividual VM names may be specified, or
the `--includes` option may be used to include all VMs that are flagged in Qubes
for automatic inclusion in backups.  The `--dedup` option will attempt to detect
duplicate data chunks and reduce the amount of data sent to and disk space taken
by the archive.  Automatic pruning is also possible; see the Wyng Readme doc for
details.

`restore [vm name] [--session=YYYYMMDD-HHMMSS]`

Restores VMs from a Wyng archive.  Individual VM name or a session may be specified.


### Notes

To address the thorny issue of restoring VM settings on Qubes OS, a best-effort process is used for
individually setting, resetting or removing each value depending on whether the property exists
in the backup and whether its writable and has a default value according to Qubes.  This differs
from the `qubes-backup` method which always creates new, differently-named VMs when restoring,
often resulting in extra, unwanted VMs which don't connect to each other or reference appropriate
templates as the user originally intended.  Since users' security expectations, scripts and
configuration are likely to hinge on VM names, `wyng-util-qubes` addresses a security risk posed by
Qubes' built-in tools.

For each qube/VM, both private and root volumes are backed up if the qube type is
either template or standalone.  Otherwise, the backup will include only a private volume.
A 'wyng-qubes-metadata' volume will also be added to the backup session.  By default,
only backup sessions which include this metadata volume will be accessible for
`restore` operations.

Use of `--pool` is optional with restore, but should be used if you have setup any non-default
Qubes pools.  Otherwise, the Qubes default pool will be used when possible.


### Limitations

Apart from data, which is restored verbatim, restoration of VM settings may be imperfect.  There is currently
no way to ensure a complete match of settings in Qubes.  However, VM names are preserved and existing
VMs with matching names _will be overwritten._


### License and Warranty
GPL3 License.

Warranty:  None.  Use at your own risk!


### History

2024-04-24: v0.9beta Support reflink (i.e. Btrfs, XFS) in addition to lvmthin.

2024-03-30: v0.8beta Ease of use update.

2023-07-19: v0.7beta Works with Wyng v0.8beta.

2023-02-10: v0.4b Beta. Adds option passthrough and delete command.

2023-02-03: v0.4a Adds verify & prune commands plus selection options for vms and sessions

2023-01-28: v0.2a Alpha. Adds list command plus more detailed handling of metadata, session and passphrase options for Wyng 0.4a.

2023-01-20: v0.1b Beta. Removes/resets a setting in existing VM when setting is not in the backup.

2023-01-19: v0.1a Initial alpha

