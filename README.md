__wyng-util-qubes__

Wrapper for [Wyng](https://github.com/tasket/wyng-backup) backup system that saves and restores both data and settings for Qubes VMs.


Requirements:

* Qubes OS 4.1 in a thin-LVM configuration
* wyng-backup v0.4alpha

Usage:
```
    wyng-util-qubes backup <qube_name [qube_name...]>
    wyng-util-qubes restore --session=YYYYMMDD-HHMMSS <qube_name [qube_name...]>
    wyng-util-qubes prune [--autoprune=opt] [--session=YYYYMMDD-HHMMSS] [qube_name qube_name...]
    wyng-util-qubes list [--session=YYYYMMDD-HHMMSS] [qube_name qube_name...]

```


Notes:

To address the thorny issue of restoring VM settings on Qubes OS, a best-effort process is used for
individually setting, re-setting or removing each value depending on whether the property exists
in the backup and whether its writable and has a default value according to Qubes.  This differs
from the `qubes-backup` method which always creates new VMs when restoring, often resulting in
extra, unwanted VMs which don't connect to each other or reference appropriate templates as
the user originally intended.

For each qube, both private and root volumes are backed up if the qube type is
either template or standalone.  Otherwise, the backup will include the private volume.


Limitations:

Currently, restore only retrieves the latest data from the default Wyng archive and dom0 backup is not
supported. During restore, encrypted archives will prompt the user multiple times for a passphrase as
Wyng doesn't yet have an authentication buffering ability.

Apart from data, which is restored verbatim, restoration of VM settings may be imperfect.  There is currently
no way to ensure a complete match of settings in Qubes.  However, VM names are preserved and existing
VMs with matching names _will be overwritten._

License:  GPL3

Warranty:  None.  Use at your own risk!

History:

2023-01-28: v0.2a Alpha. Adds list, prune commands plus more detailed handling of metadata, session and passphrase options for Wyng 0.4a.

2023-01-20: v0.1b Beta. Removes/resets a setting in existing VM when setting is not in the backup.

2023-01-19: v0.1a Initial alpha


To Do:

* Archive selection
* Use full sparse mode
* A maintenance function to support regular periodic 'monitor' and 'prune' functions of the archive
