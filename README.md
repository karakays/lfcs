
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


### Chapter III Processes

`orphan`: A process whose process has terminated. Its PPID is set to 1 that is, it's adopted by `init`.

`zombie`: A child process terminates without its parent gets notified of this. It still has an entry in the process table. 

`wait` system call can be made by a process after forking a child process. Parent suspends execution and wait child process to complete and gets notified of its exit status.

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

D    uninterruptible sleep (usually IO)
R    running or runnable (on run queue)
S    interruptible sleep (waiting for an event to complete)
T    stopped by job control signal (Ctrl+Z)
t    stopped by debugger during the tracing
X    dead (should never be seen)
Z    defunct ("zombie") process, terminated but not reaped by its parent

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

a. rpm... RedHat Package Management
rpm is low-level package manager.
Higher-level PMs based on it and manage dependencies.
yum: in RedHat
DNF: in Fedora
zypper: in SUSE

b. APT... advanced packaging tool
dpkg is low-level package manager.
apt-cache and apt-get is high-level PMs.

