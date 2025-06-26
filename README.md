## __wyng-util-qubes__

A wrapper for the [Wyng](https://github.com/tasket/wyng-backup) backup system that saves and
restores both data and settings for Qubes VMs.


### Requirements

* Qubes OS 4.2 using thin-LVM, Btrfs or XFS local storage
* [Wyng backup](https://github.com/tasket/wyng-backup) v0.8beta 20250215 or later


### Installation quick start

See [here](https://github.com/tasket/wyng-backup?tab=readme-ov-file#verifying-code) for instructions on verifying downloaded code with git.  Placing `wyng` and
`wyng-util-qubes` together in the same directory ensures that the util can find & run Wyng; in the
example below _/usr/local/bin_ is used, but you can choose a different location.

```
[user@dom0 ~]$ sudo qubes-dom0-update python3-pycryptodomex python3-zstd
[user@dom0 ~]$ sudo cp -a wyng-backup/src/wyng wyng-util-qubes/src/wyng-util-qubes /usr/local/bin
```
If using `wyng-util-qubes` Python API, it can also be installed to a Python module path like so:
```
[user@dom0 ~]$ sudo ln -s /usr/local/bin/wyng-util-qubes /usr/lib64/python3.11/site-packages/wyng_util_qubes.py
```

#### Btrfs Storage
If your Qubes storage pools are on Btrfs then you will need to convert any pool directories to Btrfs subvolumes before doing backups.  For the default Qubes Btrfs install, there is one pool dir to convert:
```
qvm-shutdown --all --force
sudo bash wyng-backup/misc/convert-dir-to-subvolume.sh /var/lib/qubes
```

### Command usage

__wyng-util-qubes__ is run in the Admin VM (dom0):

```
 wyng-util-qubes create --dest=<URL>
 wyng-util-qubes backup --dest=<URL> [--dedup] [-i] [qube_names...]
 wyng-util-qubes restore --dest=<URL> [--session=YYYYMMDD-HHMMSS] [--pool=poolname] [qube_names...]
 wyng-util-qubes verify --dest=<URL> [--session=YYYYMMDD-HHMMSS] [qube_names...]
 wyng-util-qubes prune --dest=<URL> [--autoprune=opt] [--all-before] [--session=YYYYMMDD-HHMMSS[,YYYYMMDD-HHMMSS]] [qube_names...]
 wyng-util-qubes delete --dest=<URL> <qube_name>
 wyng-util-qubes list --dest=<URL> [--session=YYYYMMDD-HHMMSS] [qube_names...]
```

### Command summary
| _Command_                     | _Description_
|-------------------------------|--------------
create             | Create a new Wyng archive
backup             | Store Qubes VMs in the Wyng archive as a session (i.e. snapshot)
restore            | Restore Qubes VMs from the Wyng archive
verify             | Verify archive data integrity
prune              | Remove older backup sessions from archive
delete             | Remove VMs from the Wyng archive
list               | Show contents of archive


### Parameters/Options summary

| _Option_                      | _Description_
|-------------------------------|--------------
--dest=_URL_           | URL location of archive.
--includes, -i         | Select all Qubes VMs marked as "include in backups". (backup)
--exclude=qube_name    | Exclude a specific VM from the operation (backup, restore, verify)
--dedup, -d            | Use deduplication. (backup)
--session=_date-time[,date-time]_ | Select a session or session range by date-time or tag (restore, list, prune).
--all                  | Show all VM names and backup sessions (list)
--all-before           | Select all sessions before the specified _--session date-time_ (prune).
--autoprune=_<off\|on\|full\>_  | Automatic pruning. See Wyng docs for details.
--pool=_qubespool_     | Override default Qubes local storage pool. (restore)
--pool-info            | Show local disk storage (list)
--pref=_pspec_         | Skip or override VM prefs (restore)
--include-disposable=_<off\|on\>_ | Include disposable VMs (restore, list)
--authmin=_N_          | Retain authentication for _N_ minutes
--no-snapshot-restore  | Don't restore from local snapshots (restore)
--no-auto-rename       | Don't rename volumes between LVM <-> Reflink formats (backup)
--remap                | Auto-remap LVM snapshots to current archive (backup)
--unattended, -u       | Operate without prompts.
--meta-dir=_path_      | Use a different metadata dir than the default.
--wyng=_path_          | Alternate path to Wyng executable
-w _wyng_option_spec_  | Pass an option directly to Wyng using the form `-w optname[=value]`


### Examples

```

$ # Start by creating a fresh Wyng archive:
$ sudo wyng-util-qubes create --dest=qubes://sys-usb/mnt/backups/laptop3.backup


$ # Make wyng backups of the VMs _work_ and _personal_
$ sudo wyng-util-qubes backup work personal --dest=qubes://sys-usb/mnt/backups/laptop3.backup


$ # Restore VM _personal_ from a wyng archive
$ sudo wyng-util-qubes restore personal --dest=qubes://sys-usb/mnt/backups/laptop3.backup

```

The above examples show the creation and use of an archive named 'laptop3.backup' located in
the '/mnt/backups' path of the _sys-usb_ VM.  Other destination types may be used such as 'qubes-ssh://' for backing up to remote via an SSH-equipped VM...

```
$ sudo wyng arch-init --dest=qubes-ssh://remote-vm:user@192.168.1.10/mnt/backups/laptop3.backup
```
See the _Options_ section for a description of all `--dest` types.


### Commands

Note that `--dest` is assumed to be specified along with each of the commands below.

#### list

`list [vm names] [--session=YYYYMMDD-HHMMSS] [--all] [--pool-info]`

Shows a directory of archive contents.  If no parameters are given, a list of qube / VM names
is given.  Disposable VMs will not be shown unless a session is specified along
with the `--include-disposable=on` option.


#### backup

`backup [vm names] [--includes] [--exclude=vmname] [--dedup] [--autoprune] `

Backs up Qubes VMs to a Wyng archive.  A list of invdividual VM names may be specified, or
the `--includes` option may be used to include all VMs that are flagged in Qubes
for automatic inclusion in backups.  The `--dedup` option will attempt to detect
duplicate data chunks and reduce the amount of data sent to and disk space taken
by the archive.  Automatic pruning is also possible; see the Wyng Readme doc for
details.

#### restore
`restore [vm names] [--session=YYYYMMDD-HHMMSS] [--exclude=vmname] [--include-disposable] [--pool=poolname]`

Restores VMs from a Wyng archive into a Qubes system.  VM names and/or a _session_ (containing
one or more VMs) may be specified.  If a session is not specified, the last session will be
auto-selected; if only a session is specified, all of the VMs in the session will be selected.

#### verify
`verify [vm names] [--session=YYYYMMDD-HHMMSS] [--exclude=vmname]`

Verifies the integrity of archived VMs. Specifying a _session_ by itself will verify
all VMs in that session.

#### prune
`prune [vm names] [--session=YYYYMMDD-HHMMSS[,YYYYMMDD-HHMMSS]] [--all] [--autoprune=<off|on|full>]`

Removes older backup sessions from the archive to reclaim space. The latest session
cannot be selected for removal. If an entire session is to be removed, `--all` (refering
to all volumes) must be specified with `--session`, otherwise VM names may be used to limit
pruning to those VMs. The session may also be a date-time range
with the start and end separated by a comma.
See Wyng documentation for specifics on using `prune` and `--autoprune`.

#### delete
`delete <vm name>`

Deletes a VM from the archive. Only one VM may be specified at a time. To remove
a _session_, see the `prune` command.


### Options

#### --dest=_URL_

This (non-optional) option specifies the location of the archive.
It accepts one of the following forms:

| _URL Form_ | _Destination Type_
|----------|-----------------
|__file:__/path                           | Local filesystem
|__ssh:__//user@example.com[:port][/path] | SSH server
|__qubes:__//vm-name[/path]               | Qubes virtual machine
|__qubes-ssh:__//vm-name:me@example.com[:port][/path]  | SSH server via a Qubes VM

Note that paths are optional for all except ___file:___ and they are always absolute.

#### --includes, -i

When backing up, select all Qubes VMs marked as "include in backups".

#### --exclude=_qube_name_

Exclude a specific VM from the backup, restore, or verify operation.  May be specified more
than once.


#### --dedup, -d

Use deduplication when backing up.  To dedup an entire archive at once, see the
Wyng documentation on `arch-deduplicate` command.

#### --session=_date-time[,date-time]_

Select a session or session range by date-time or tag. Used with restore, verify, list, and prune.

#### --pool=_poolname_
When restore creates new VMs in the system, use the Qubes storage pool specified by <poolname> instead of the system default.

#### --pref=_prefname::x_

Control how Qubes VM pref _'prefname'_ is handled during `restore` where _'::x'_ indicates skipping
the specified pref instead of trying to set it.  This can be used as a workaround allowing
`restore` to complete when an
archived VM setting isn't compatible with the current system configuration.


### Notes

Most of the notes and tips on using Wyng also apply to `wyng-util-qubes` usage. It is recommended
to read or at least skim them to gain general familiarity with Wyng.

To address the thorny issue of restoring VM settings on Qubes OS, a best-effort process is used for
individually setting, resetting or removing each value depending on whether the property exists
in the backup and whether its writable and has a default value according to Qubes.  This differs
from the `qubes-backup` method which often creates new, differently-named VMs when restoring,
often resulting in extra, unwanted VMs which don't connect to each other or reference appropriate
templates as the user originally intended.  Since users' security expectations, scripts and
configuration are likely to hinge on VM names, `wyng-util-qubes` addresses a security risk posed by
Qubes' built-in tools.

Disposable VMs: Since dispVMs do not have their own physical storage, they exist only in the Qubes XML metata which is always backed up in its entirety.  This means all dispVMs are included in each backup session.  However, `wyng-util-qubes` cannot see all versions of this XML data at any given time, so it cannot list dispVMs without giving a specific `--session` ID, and it cannot delete dispVMs.

For each qube/VM, the private and/or root volumes are automatically backed up and restored depending on the type of VM,
and a 'wyng-qubes-metadata' volume will always be added to the backup session as well.
By default, only backup sessions which include this metadata volume will be accessible for
`restore` operations, so backups performed directly with `wyng send` are unlikely to have
the metadata needed to make them accessible from `wyng-util-qubes` (the volumes can of course
still be accessed with `wyng`).

When a system relies on the QubesOS default of Thin LVM there is an avoidable cause of pool metadata space exhaustion, a condition that can cause your system storage to go offline or become corrupted. Since Wyng adds its own snapshots on top of Qubes snapshots, using Wyng adds a bit more demand for Thin LVM metadata. The answer to this is almost always to increase the qubes-dom0/vm-pool metadata size with `sudo lvextend --poolmetadatasize`. 3X as large as the original default is a good choice to avoid excess space consumption.

Likewise, Btrfs metadata can experience added stress from Wyng snapshots.  Here the metadata stress manifests as a slowdown of system operations.  This can be avoided by periodically defragmenting your Btrfs Qubes storage pools like so: `sudo btrfs filesystem defrag -fr -t256K /var/lib/qubes` approximately once per week or month, depending on how active your system is.  This is good to do any time you are intensively using large disk image files on Btrfs where the default _datacow_ capability remains enabled.


### Limitations

Apart from data, which is restored verbatim, restoration of VM settings may be imperfect.  There is currently
no way to ensure a complete match of settings in Qubes.  However, VM names are preserved and existing
VMs with matching names _will be overwritten._

### Python API

`wyng-util-qubes` may also be imported and used as a module in Python. See [issue #37](https://github.com/tasket/wyng-util-qubes/issues/37) for details on module usage.

### License and Warranty
GPLv3 License.

Warranty:  None.  Use at your own risk!


### History

2025-06-21: v0.9beta(5) Improved error handling; support Qubes notes.txt; safer extraction

2025-05-10: v0.9beta Minor fixes & options (Wyng path, remap)

2025-03-31: v0.9beta Python API & restore from snapshot

2024-04-24: v0.9beta Support reflink (i.e. Btrfs, XFS) in addition to lvmthin.

2024-03-30: v0.8beta Ease of use update.

2023-07-19: v0.7beta Works with Wyng v0.8beta.

2023-02-10: v0.4b Beta. Adds option passthrough and delete command.

2023-02-03: v0.4a Adds verify & prune commands plus selection options for vms and sessions

2023-01-28: v0.2a Alpha. Adds list command plus more detailed handling of metadata, session and passphrase options for Wyng 0.4a.

2023-01-20: v0.1b Beta. Removes/resets a setting in existing VM when setting is not in the backup.

2023-01-19: v0.1a Initial alpha



### Donations

<a href="https://liberapay.com/tasket/donate"><img alt="Donate using Liberapay" src="media/lp_donate.svg" height=54></a>

<a href="https://ko-fi.com/tasket"><img src="media/ko-fi.png" height=57></a> <a href="https://ko-fi.com/tasket">Ko-Fi donate</a>

<a href="https://www.buymeacoffee.com/tasket"><img src="media/buymeacoffee_57.png" height=57></a> <a href="https://www.buymeacoffee.com/tasket">Buy me a coffee!</a>


If you like this project, monetary contributions are welcome and can
be made through [Liberapay](https://liberapay.com/tasket/donate) or [Ko-Fi](https://ko-fi.com/tasket) or [Buymeacoffee](https://www.buymeacoffee.com/tasket).
