__wyng-util-qubes__

Wrapper for Wyng backup system that saves and restores both data and settings for Qubes VMs.

Uses a best-effort restoration process for restoring VM settings, which is a thorny problem on Qubes OS.

Usage:
```
    wyng-util-qubes backup <qube_name [qube_name...]>
    wyng-util-qubes restore <qube_name [qube_name...]>
```

License:  GPL3

Warranty:  None.  Use at your own risk!

History:

2023-10-20: v0.1b Beta. Removes/resets a setting in existing VM when setting is not in the backup.

2023-01-19: v0.1a Initial alpha


To Do:

* Session selection
* Archive selection
* List archive contents
* Use full sparse mode
