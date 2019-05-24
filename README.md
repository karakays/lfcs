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
Pseudo-fs. Persists only in memory for processes to keep internal state. Each process has its place as a subdirectory

##### /root
Home of root user

##### /var
For variable and volatile data that changes frequently. Logs, spool directories (for mail and cron), transient data for cache (e.g. packages) , lock files (linked to /run/lock)

##### /run
Pseudo-fs. For transient data that contains runtime information as lock files

### Chapter III Processes

`orphan`: A process whose process has terminated. Its PPID is set to 1 that is, it's adopted by `init`.

`zombie`: A child process terminates without its parent gets notified of this. It still has an entry in the process table. 

`wait` is a system call that may be submitted by a process after forking a child process. Parent suspends execution and wait child process to complete and gets notified of its exit status.

PID is 16 bit integer located in /proc/sys/kernel/pid_max and can be altered.

##### Limits
`ulimit` gets/sets limits of the calling process. Limits on resources such as as max cpu time, max file locks, max open files, max threads, stack size, memory size, file size etc.

##### Permissions
A process can inherit permissions from 
* executor - the user who executes it
* executable - executable binary file, `setuid` bit set.

##### Execution modes
At any given time, a process is running in a certain execution mode.
* user mode: usually when executing application code
* kernel mode: when executing system calls to access hardware resources memory, disk, peripherals.

##### fork and exec
$ ls
bash process forks itself and a new copy, i.e. child process is created. fork() returns child's PID to parent. bash process goes into sleep state with `wait` system call on its child.
copy process makes `exec` system call and loads `ls` code in the child process space and executes it.
when execution completed, child process terminates via exit system call and returns exit code to kernel.
bash receives exit status either via polling or SIGCHLD signal handler from kernel, removes child's entry from process table (i.e. reaped) and it resumes execution.

state code |  Description
--- | --- 
D   | uninterruptible sleep (usually IO)
R   | running or runnable (on run queue)
S   | interruptible sleep (waiting for an event to complete)
T   | stopped by job control signal (Ctrl+Z)
t   | stopped by debugger during the tracing
X   | dead (should never be seen)
Z   | defunct ("zombie") process, terminated but not reaped by its parent

Kernel-created processes can run in
* kernel space for maintenance work
* user space - usually short lived

Their parent is kthreadd[2] and names are in encapsulated brackets.

##### Process priority
`nice` executes a command with adjusted niceness. The lower the niceness, the higher is the process priority.
$ nice -n +3 command args # increase niceness by 3, lowers priority by 3
`renice` to prioritize an existing process
$ renice -n -5 <pid>

##### Library types
static: An application compiled with a static library doesn't change thereafter even if static library is updated. .

shared: Such libraries can be loaded into application at runtime - also called dynamic link libraries (DLL). I think it can be thought as an external dependency that is satisfied from the environment and not during application build time. Shared libraries have \*.so extension.

`ldd` list shared dependencies of an executable.

### Chapter IV Signals

A way of communication with a process (basic IPC)
It can originate from:
* _user_: via `kill` command
* _a process_: must be via system call through kernel
* _kernel_: e.g. a process makes an illegal memory reference

`kill` is the command to send signals to a process.

Signal handlers can be implemented in the program code or it can response according to the system defaults.

Signal | Default action | Description
--- | --- | ---
SIGHUP | Terminate | convention: for daemons, it reloads configuration file.
SIGINT | Terminate | interrupt from keyboard Ctrl-C
SIGFPE  | Core dump | sent from kernel to a process that attempts to divide by 0
SIGKILL | Terminate | abnormal termination (cannot be handled nor ignored)
SIGTERM | Terminate | graceful termination (default in `kill`) 
SIGSTOP | Stop | Cannot be handled nor ignored
SIGTSTP | Stop | Ctrl-Z
SIGCONT | Continue | Resume 
SIGCHLD | Ignore | Child stopped/terminated 
SIGPIPE | Terminate | Broken pipe; socket closed

`killall` sends signals to process(es) by name or user given.
`pkill` sends signals to processes by filtering by name or user.
`pgrep` is synopsis. Filters by name, owner, parent, time.
pgrep -u root java  # list java processes owned by root


### Chapter V. Package managers
In general, two families of package managers can be considered.

1. rpm... RedHat Package Manager
2. dpkg... Debian Package Manager

Packaging systems can be classified in two levels:
1. low-level: Low-level PMs deal with installation/removal only. They don't carry out dependency management. If a package is missing a dependency during installation, it will fail. Similarly, if a removal depends on some other package, it will also fail. 
* rpm in RedHat family
* dpkg in Debian family

2. high-level: High-level PMs are based on the low-level PMs. They handle dependency management, automatic dependency resolving and accesses external software repositories i.e. downloading/installing/removing of dependencies when needed.
* yum (RHEL, CentOS), dnf (Fedora) and zypper (SUSE) in RedHat family
* apt suite in Debian family


### Chapter VII. dpkg

dpkg database is located at /var/lib/dpkg/
dpkg is not aware of any repository. It knows, however, what is installed in the local system from dpkg database.

##### Package namings:

Binary package name:
<name>\_<version>-<revision>\_<architecture>.deb
e.g. logrotate\_3.7.8-1-\_amd64.deb
64 bit architecture called amd64.

Src package consists of:
* upstream src package \*.tar.gz (from package maintainer)
* metadata file \*.dsc describing the package
* second src package that contains patches to the upstream source \*.debian.tar.gz

##### Features

```bash
$ dpkg -l         # List installed packages
$ dpkg -c pkg.deb # List contents of \*.deb package
$ dpkg -L package # List files installed from a package
$ dpkg -S file    # Search what package installed <file> (bin, conf etc.)
$ dpkg -s package # Query status of an installed package
$ dpkg -I pkg.deb # Show info about \*.deb binary package 
```

```bash
$ dpkg -i pkg.deb ... # Install/update pkg.deb
$ dpkg -r pkg.deb ... # Remove/purge (-P)
```

### Chapter X. APT
APT software suite contains mainly apt-cache and apt-get which are based on dpkg.

##### Features

`apt-cache` queries packages from internal database (package index) at /var/lib/apt/lists.


```bash
$ apt-cache search term         # full text search on name and desc on all package lists
$ apt-cache show/showpkg pkg    # show information on pkg - exact name search 
$ apt-file search file          # search all packages that contains file
$ apt-file list pkg             # list all files in the pkg. pkg doesn't need to be installed or fetched
```

```bash
$ sudo apt-get autoremove       # uninstall not needed dependencies
$ sudo apt-get clean            # delete installed package archives from cache
```

```bash
$ sudo apt-get update       # sync package index lists with remote repos
$ sudo apt-get install pkg  # install/update pkg
$ sudo apt-get [--purge] remove pkg     # remove/purge pkg
$ sudo apt-get upgrade                  # apply all available updates to all pkgs 
```

### Chapter XI. System Monitoring

##### Monitoring tools

* process/load

--- | --- |
top, ps, pstree | processes
uptime | load
mpstat | multiprocessor usage
strace | system calls tracing

* memory/io

--- | --- |
free | brief memory usage
vmstat | detailed memory and block io usge
pmap | process memory map
iostat | cpu/io stats

* network

--- | --- |
netstat | 
iptraf | ?

##### System logging

Important log files in `/var/log/`
--- | --- |
boot.log | system boot messages
dmesg   | kernel messages
syslog  | all important system messages
security  |  security logging

* /proc/ and /sys/ are pseudo-filesystems and contain information about the system state.

##### /proc/
* Current state of each process running: child processes, memory usage ...
* /proc/interrupts contain statistics on interrupts
- how many times an interrupt type was handled by which CPU?
* /proc/meminfo: meminfo 
* /proc/sys/ contain tunable system parameters in plain text files
- /proc/sys/kernel: kernel paramters
- /proc/sys/vm: virtual memory paramters

##### /sys/
aka sysfs. Provides unified information on device and drivers of various types, unified device model. Much of the hw information moved from /proc to /sys.
/dev vs /sys: /dev contains device files to access the devices themselves whereas /sys contains device information as powered on, vendor, model, bus etc.

##### SAR
System activity reporter for humans. Report is made up from the data that is collected by SADC in /var/log/sa periodically. Report can contain about
- IO, paging, network, per CPU, swap and memory, context switching etc.

$ sudo sar 3 3  # default report 3 times in 3 seconds

### Chapter XII. Process Monitoring

##### ps

```ps [pid]...```

`ps` grabs the information from /proc fs.

###### `ps` output table
column | description
--- | ---
vsz | virtual memory size in KB
rss | resident set size (physical mem size excl. swap)
cpu | cpu utilization
wchan | kernel function where process is sleeping
tty | attached terminal
pri | priority
ni | nice
comm | executable name only
args | command with all its args

###### `ps` options

```ps aux  # interpret options in BSD style grouped```
```ps -aux # interpret options in UNIX style grouped```

a: BSD style all processes
u: output in user-oriented format
x: remove "must have tty" restriction
-e: UNIX style all processes
-o : customize output like ps -o pid,uid,cmd
-l : long format

* `pstree` to visualize process hierarchy by pid or uid.
pstree [options] [pid,user]

### Chapter XIII Memory Monitoring

Virtual memory consists of resident memory and swap area.

`/proc/sys/vm/` contains tunables for the virtual memory system.
entry | purpose
---   | ---
overcommit_memory | enable/disable overcommitting
overcommit_ratio | ?
oom_kill_allocating_task | let oom-killer kill the task that triggered

##### vmstat

```vmstat [options] [delay] [count]```
`vmstat` is the workhorse utility used mainly for memory stats but also for CPU, process and disk statistics.
-a option includes active and inactive memory namely pages recently used (might be clean or dirty) and pages not been recently used.
-d option is for disk statistics.

###### output columns

* procs
- r: # of processes in ready state
- b: # of processes in blocked state
* memory and io
- swpd: swap size being used
- free: current free memory
- buff: buff vs cache??
- cache: cache for data on disk for reads and also writes.
    * flush means write cache to disk.
- si: swap in v-mem read blocks/sec
- so: swap out v-mem written blocks/sec
- bi: blocks recvd from block device per sec
- bo: blocks sent to block device per sec
* system and cpu
- in: interrupts per sec
- cs: context switches per sec
- us: %CPU time spent running _user code_
- sy: %CPU time spent running _kernel code_
- id: %CPU time spent idle
- wa: %CPU time spent blocked in IO 
- st: %CPU time spent stolen from vm

###### Memory management

* Swap area is moved back to memory when enough memory is freed or priority becomes in swap becomes higher.

* To come out of the memory pressure, kernel overcommits memory (exceeds RAM + swap) by means of that many processes don't use all requested memory. This technique applies for user processes only.

* Overcommission is configured in `/proc/sys/vm/overcommit_memory` file.

* In case available memory is exhauseted, OOM-killer exterminates process(es) selected to free up memory. The hard part here is which process will be killed to keep the system alive. To avoid the whole system to crash, a victim process is selected to free up memory. Selection is based on the value `badness` that is at /proc/{id}/oom_score`.

```swapon/swappoff [devices...] # enable/disable devices for paging/swapping```

### Chapter XIV. IO Monitoring

In an I/O-bound system, the CPU is mostly idle waiting IO ops to complete such as disk or network ops.

##### `iostat`

```$ iostat [delay] [count]```

iostat is the workhorse utility for IO monitoring. It shows mainly RW transfer rates by disk partition including CPU utilization. The first report (row) is stats since the system boot.

###### output columns
Data is expressed in number of blocks. A block have a size of 512 bytes.

* logical IO requests can be merged into one actual request

field | desc
---   | ---
device | disk device [or partition]
tps | io transactions per second
kB_read/s | blocks read per seconds in kB
kB_wrtn/s | blocks written per seconds in kB
kB_read | total block read in kB
kB_wrtn | total block written in kB

* -x option for extended output

field | desc
---   | ---
rrqm/s | # of read requests merged per sec, queued to device
wrqm/s | # of write requests merged per sec, queued to device
r/s | # of read requests per sec, to the device
w/s | # of write requests per sec, to the device 
rkB/s | KB read per sec 
wkB/s | KB written per sec 
avgrq-sz | average request size in 512 bytes per sector
avgqu-sz | average queue length???
await | avg time elapsed in ms for queueing + servicing IO request
svctime | avg service time in ms for IO request
%util | ???

##### iotop

* displays current IO usage by process updated periodically as in top.
* requires root access

* output columns

field | desc
---   | ---
SWAPIN | time percentage process blocked waiting swap in
IO | time percentage process blocked waiting io
PRIO | io priority

##### IO scheduling
* Every process is associated with an IO classifier and priority. `ionice` sets scheduling class/priority of a given process manually

```ionice [-c class] [-n pri] [-p pid] [cmd args]```

Apply io scheduling either for existing pid or by starting new process <cmd>
* IO scheduling class:
- 0: none
- 1: real time with a priority [0-7]
- 2: [default] best effort with a priority [0-7]
- 3: idle: served when there are no more requests

### Chapter XV. IO Scheduling
* VM and VFS submit IO requests and it's the job of IO scheduler to prioritize and order these requests before they are given to block devices.

* IO scheduling (sometimes conflicting) requirements:
- Minimize HW access:
    * requests ordered according to the physical location on disk: elevator scheme. SSDs don't require elevator scheme (rotational = 0)
    * requests are merged to get as big a contiguous region as possible
- Write operations can wait to migrate from caches to disk without stalling - async. Read ops are blocking until completed. Reads favored over writes
- Processes to share IO bandwidth fairly

* There are different IO scheduling strategies available
- Completely Fair Queueing (CFQ)
- Deadling scheduling
- noop

* IO scheduler strategy can be specified per device at __kernel boottime__ or at __runtime__
- to see available and current strategy, check `/sys/block/sda/queue/scheduler`
- to switch the io scheduler
```echo noop > /sys/block/sda/queue/scheduler```

* IO scheduling tunables are at ```/sys/block/<device>/queue/iosched/``` directory. These parameters are based on the strategy and change from one strategy to another.

##### CFQ

* In CFQ, each process have its own request queue which works with a global dispatcher queue that submits actual requests to the device. Dequeuing each of these queues is done in round-robin style. Each queue is allocated timeslices to access the disk and works in FIFO order. Tunables:
- quantum: max len of dispatcher queue
- fifo_expire_async: expiry time of async request (buffered write) in queue. After it expires, it goes to dispatcher queue.
- back_seek_max: max distance for backwards seeking. repositioning the head backwards is bad performance.

##### Deadline

* Deadline strategy works based on the deadline - expiry of each request that guarantees to be served. There are 2 queues for r and w ordered by starting block (elevator queue), another 2 ordered by submission time (expire queue) and one global dispatcher queue. Scheduler checks expired requests first and only then moves to the elevator queue. Tunables:
- read_expire: deadline for r request
- write_expire: deadline for w request
- write_starved: reads are preffered to writes. how many w requests can be starved?
- fifo_batch: # of requests to move from sorted list to dispatcher when deadlines expired.
- front_merges: related with contiguous requests?

### Chapter XVI. Filesystems

Applications do not access the physical disk directly. Instead, application code access data contents by file names which is an abstraction by the filesystem.

Every file is associated with an inode. inode is a datastructures that holds following metadata about the file:
* permissions
* owner / owner group
* size
* timestamps (last accessed, modified, change)
To read a file content, you need to have inode number, first. File names are not stored in the file inodes but in the directory inode. Directory is a special file which holds a table that maps file names to inode ids. The name-inode mapping is called `link`.
```ln [-s] target link-name # creates a link```
* Hard link points to an inode. As a result, the same file can have multiple names mapped to the same inode.
* Soft link points to a filename

The entire filesystem tree doesn't need to be in one local disk partition. Some branches may be in another partition, network or removable media (usb etc.) Such branches are `mounted` to the filesystem tree.

##### Virtual file system
Application code doesn't change when accessing files in different disk partitions which might be using different filesystems. VFS is an abstraction layer that provides transparent data access from the underlying filesystem. This permits Linux to work with many filesystems:
* Linux native fs: ext4
* Windows native fs: fat32, vfat, ntfs
* Pseudo-fs: /proc, /sys, /dev
* Network fs; ntfs
* xfs: default fs in RHEL
`/proc/filesystem` holds the filesystem list suppored by the kernel.
`page` is smallest unit of data for virtual memory and a fixed-length contiguous block.

##### Special file systems
Kernel employs them for certain tasks, not available to user space.
Filesystem | Mount point | purpose
--- |   --- |  ---
rootfs  | none | 
proc    | /proc |
tmpfs   | Anywhere | file storage in RAM
sysfs   | /sys |

```df [file]... # disk space on all mounted filesystems```

### Chapter XVII. Disk partitioning

##### Common disk types
* SATA
* SCSI
* USB
* SSD

##### Disk geometry
* `fdisk` to manipulate the partition table.
Rotational disks consist of platters, each of which is read by heads as disk spins. Tracks are divided into sectors in 512 byte size. SSDs are not rotational and have no moving parts.

##### Partitioning
Disk is divided into contiguous groups of sectors called partitions. Partitioning promotes:
* separation of user space from system space.
* easier backups
* performance and security enhancement on certain parts
* swap can be isolated
A partition is associated with
* type: primary, extended or logical
* start: start sector
* end: end sector

Partition table schemes:
#### MBR
* old and obsolete. Stored in the first 512 bytes of the disk. up to 4 primary partitions and one can be as an extended partition.
* Table has 4 entries and each 16 bytes size. Entry in the table contains active bit, file system code (xfs, ext4, swap etc.) and number of sectors.

#### GPT
* modern.
* Up to 128 entries in the table and each 128 bytes of size.

Kernel interacts with disk devices (found in /dev directory) through VFS. SCSI and SATA disks follow xxy[z] naming convention where xx is device type (sd), y is drive number and z is partition number as in `/dev/sdb1` or `/dev/sdc4`

`blkid` reports block device metadata (attributes).

`lsblk` lists block devices in a tree format.

`blkid` and `lsblk` work only with block devices.

#### Backup partition tables

To backup MBR, copy MBR table
```sudo dd if=/dev/sda of=mbrbackup bs=512 count=1```

To restore MBR write it back to the disk
```sudo dd if=mbrbackupk of=/dev/sda bs=512 count=1```

To backup GPT, use `sgdisk`
```sudo sgdisk --backup=sdabackup /dev/sda```

#### Partition table editors
command | Desc
--- |   ---
fdisk  | most standard tool, works interactively
sfdisk | non-interactive fdisk for scripting
parted | GNU partition manipulation
gparted| gui for parted
gdisk  | guid partition table manipulator
sgdisk | script interface for gdisk

To create a partition with `parted`
```sudo parted /dev/loop0 unit MB mkpart primary ext4 0 256```
You can specify partition file-system here or later with `mkfs`.

* /proc/partitions is what kernel is aware of partitions.

* ```losetup``` to associate a file or block device with a loop device. A loop device is pseudo device which makes a file to be accessed as a block device. Certain commands like `lsblk` work only with block devices.


### Chapter XVIII. Filesystem features

Extended attributes are metadata that filesystem does not handle directly. `lsattr` to list and `chattr` to change attributes.

Flags:
* a: append-only
* i: immutable
* d: skip dump
* A: set access time only when mod-time changes
