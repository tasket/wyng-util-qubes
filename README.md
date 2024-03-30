### __wyng-util-qubes__

A wrapper for [Wyng](https://github.com/tasket/wyng-backup) backup system that saves and restores both data and settings for Qubes VMs.


### Requirements

* Qubes OS 4.1 in a thin-LVM configuration
* wyng-backup v0.8beta


### Command usage
```
 wyng-util-qubes backup [--dedup] [-i] [qube_name...]
 wyng-util-qubes restore --session=YYYYMMDD-HHMMSS [qube_name...]
 wyng-util-qubes verify --session=YYYYMMDD-HHMMSS [qube_name...]
 wyng-util-qubes prune [--autoprune=opt] [--all-before] [--session=YYYYMMDD-HHMMSS[,YYYYMMDD-HHMMSS]] [qube_name...]
 wyng-util-qubes delete <qube_name>
 wyng-util-qubes list [--session=YYYYMMDD-HHMMSS] [qube_name...]
```

### Command summary
| _Command_                     | _Description_
|-------------------------------|--------------
backup             | Begin a backup session to store Qubes VMs in the Wyng archive
restore            | Restore Qubes VMs from the Wyng archive
verify             | Verify archive data integrity
prune              | Remove older backup sessions from archive
delete             | Remove VMs from the Wyng archive
list               | Show contents of archive


### Parameters/Options summary

| _Option_                      | _Description_
|-------------------------------|--------------
--includes, -i         | Select all Qubes VMs marked as "include in backups". (backup)
--exclude=qube_name    | Exclude a specific VM from the operation (backup, restore, verify
--dedup, -d            | Use deduplication. (backup)
--session=_date-time[,date-time]_ | Select a session or session range by date-time or tag (restore, list, prune).
--all-before           | Select all sessions before the specified _--session date-time_ (prune).
--autoprune=off        | Automatic pruning by calendar date. _(experimental)_
--exclude=qube_name    | Exclude a qube from backup by name.
--dest=_URL_           | URL location of archive.
--pool=_qubespool_     | Override default 'vm-pool' Qubes local storage pool. (backup, restore)
--pool-info            | Show local disk storage (list)
--local=_vg/pool_      | Deprecated. Use --pool instead.
--volume=volname       | Include a specific disk volume by name (not qube name)
--unattended, -u       | Operate without prompts.
--meta-dir=_path_      | Use a different metadata dir than the default.
-w _wyng_option_spec_  | Pass an option directly to Wyng using the form `-w optname[=value]`


### Examples

```

$ # Start by creating a fresh Wyng archive:
$ sudo wyng arch-init --dest=file:/mnt/backups/laptop3.backup


$ # Show Qubes local storage
$ sudo wyng-util-qubes list --pool-info
wyng-util-qubes v0.8beta

Qubes local storage pools:
 varlibqubes             : file-reflink, /var/lib/qubes
 vm-pool                 : thin_lvm, qubes_dom0/vm-pool

 
$ # Make wyng backups of the VMs _work_ and _personal_
$ sudo wyng-util-qubes backup work personal --pool=vm-pool --dest=file:/mnt/backups/laptop3.backup


$ # Restore VM _personal_ from a wyng archive
$ sudo wyng-util-qubes restore personal --pool=vm-pool --dest=file:/mnt/backups/laptop3.backup

```

### Notes

To address the thorny issue of restoring VM settings on Qubes OS, a best-effort process is used for
individually setting, resetting or removing each value depending on whether the property exists
in the backup and whether its writable and has a default value according to Qubes.  This differs
from the `qubes-backup` method which always creates new, differently-named VMs when restoring,
often resulting in extra, unwanted VMs which don't connect to each other or reference appropriate templates as the user originally intended.  Since users' security expectations, scripts and
configuration are likely to hinge on VM names, `wyng-util-qubes` addresses a security risk posed by
Qubes' built-in tools.

For each qube/VM, both private and root volumes are backed up if the qube type is
either template or standalone.  Otherwise, the backup will include the private volume.
A 'wyng-qubes-metadata' volume will also be added to the backup session.  By default,
only backup sessions which include this metadata volume will be accessible for
`restore` operations.


### Limitations

Apart from data, which is restored verbatim, restoration of VM settings may be imperfect.  There is currently
no way to ensure a complete match of settings in Qubes.  However, VM names are preserved and existing
VMs with matching names _will be overwritten._

Currently, selection of different archives can only be done via the `---meta-dir` option.


### License and Warranty
GPL3 License.

Warranty:  None.  Use at your own risk!


### History

2023-07-19: v0.7b Works with Wyng v0.8beta.

2023-02-10: v0.4b Beta. Adds option passthrough and delete command.

2023-02-03: v0.4a Adds verify & prune commands plus selection options for vms and sessions

2023-01-28: v0.2a Alpha. Adds list command plus more detailed handling of metadata, session and passphrase options for Wyng 0.4a.

2023-01-20: v0.1b Beta. Removes/resets a setting in existing VM when setting is not in the backup.

2023-01-19: v0.1a Initial alpha


### To Do

* Archive selection
* Btrfs support
