
### Chapter II Filesystem layout

FHS… initiative to standardise filesystem organization in distributions

Pseudo-filesystems
Reside only in memory during runtime and not on disk. Contains nothing on a non-running system.

##### /bin
Essential binaries required when no other filesystems have yet been mounted. Also in single user mode or recover mode.

##### /sbin
System binaries during booting, recover or restore. Also capable of mounting /usr, /home etc.

##### /usr
A secondary hierarchy. /usr/bin, /usr/lib, /usr/local, /usr/sbin
/usr/bin
Not needed for system booting. Contains multi-user applications. Package managers touch here.
/usr/share
Documentation (incl. man pages)
/usr/src
Kernel source

##### /boot
Contains compressed kernel image and initrd

##### /opt
Isolated installation directory (not scattered in multiple directories)
Helpful for proprietary sw or packages downloaded without package managers


##### /proc
Pseudo-fs. for processes to keep internal state. Each process has its place as a subdirectory

##### /root
Home of root user

##### /var
For variable and volatile data that changes frequently. Logs, spool directories (for mail and cron), transient data for
cache, lock files (linked to /run/lock)

##### /run
Pseudo-fs. For transient data that contains runtime information as lock files
