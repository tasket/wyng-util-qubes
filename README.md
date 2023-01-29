### __wyng-util-qubes__

A wrapper for [Wyng](https://github.com/tasket/wyng-backup) backup system that saves and restores both data and settings for Qubes VMs.


### Requirements

* Qubes OS 4.1 in a thin-LVM configuration
* wyng-backup v0.4alpha


### Command Usage
```
    wyng-util-qubes backup <qube_name [qube_name...]>
    wyng-util-qubes restore --session=YYYYMMDD-HHMMSS <qube_name [qube_name...]>
    wyng-util-qubes prune [--autoprune=opt] [--session=YYYYMMDD-HHMMSS] [qube_name qube_name...]
    wyng-util-qubes list [--session=YYYYMMDD-HHMMSS] [qube_name qube_name...]
```

### Parameters/Options Summary

| _Option_                      | _Description_
|-------------------------------|--------------
--session=_date-time[,date-time]_ | Select a session or session range by date-time or tag (restore, list, prune).
--all-before           | Select all sessions before the specified _--session date-time_ (prune).
--autoprune=off        | Automatic pruning by calendar date. (experimental)
--pass-agent=n         | Minutes to remember auth/decryption passphrase (default: 0)
--dedup, -d            | Use deduplication. (backup)
--local=_vg/pool_      | Volume group + pool containing local volumes. (restore)
--meta-dir=_path_      | Use a different metadata dir than the default.


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

2023-01-28: v0.2a Alpha. Adds list, prune commands plus more detailed handling of metadata, session and passphrase options for Wyng 0.4a.

2023-01-20: v0.1b Beta. Removes/resets a setting in existing VM when setting is not in the backup.

2023-01-19: v0.1a Initial alpha


### To Do

* Verify and Prune
* Archive selection
* Use full sparse mode
* Btrfs support
* A maintenance function to support regular periodic 'monitor' and 'prune' functions of the archive
