# Linux Advanced Last-Minute Interview Notes
## Commands • Short Notes • Scripts • Troubleshooting • Answers

> Fast revision for Linux administration, production/application support, networking, performance, Bash, systemd, security, and scenario-based interviews.

---

# 0. How to Use This File

Use this in three passes:

1. **Before an interview:** revise the command tables and one-line answers.
2. **For technical rounds:** use the investigation flows instead of memorizing commands.
3. **For scenario questions:** explain `symptom → evidence → hypothesis → test → fix → verification`.

A good Linux answer should explain **why** a command is being used, not just name the command.

---

# 1. Linux Fundamentals

## Q1. What is Linux?

Linux is an open-source family of operating systems built around the Linux kernel. A complete Linux distribution combines the kernel with system libraries, utilities, package managers, shells, services, and applications.

### Key distinction

```text
Linux kernel
    ↓
Distribution
    ↓
Ubuntu / RHEL / Debian / Fedora / etc.
```

---

## Q2. What is the Linux kernel?

The kernel is the privileged core of the operating system.

It manages:

- CPU scheduling
- Memory
- Processes
- Devices
- Filesystems
- Networking
- System calls
- Security mechanisms

```text
Applications
     ↓
Libraries / APIs
     ↓
System calls
     ↓
Linux Kernel
     ↓
Hardware
```

---

## Q3. Kernel vs Shell?

| Kernel | Shell |
|---|---|
| Core OS component | Command interpreter |
| Manages hardware/resources | Accepts user commands |
| Runs in privileged mode | Normally runs as a user process |
| Exposes system calls | Uses programs/system calls |

Examples of shells:

```text
bash
zsh
fish
sh
```

---

## Q4. Linux vs Unix?

Unix is a family of operating systems and a historical standards ecosystem.

Linux is a Unix-like, open-source kernel used by many distributions.

A safe interview answer:

> Linux is Unix-like but is not the original Unix operating system.

---

## Q5. What happens when you execute a command?

For a shell command such as:

```bash
ls -l /var/log
```

roughly:

```text
Shell parses command
       ↓
Shell identifies builtin or executable
       ↓
Environment/path resolution
       ↓
Process creation when required
       ↓
Program execution
       ↓
Program performs system calls
       ↓
Output → stdout
Errors → stderr
       ↓
Process exits
       ↓
Shell receives exit status
```

Useful tools:

```bash
type ls
which ls
command -v ls
echo $?
```

---

## Q6. What is a process?

A process is a running instance of a program with its own process state and virtual address space.

Useful identifiers:

```text
PID  → process ID
PPID → parent process ID
UID  → user ID
GID  → group ID
```

Inspect:

```bash
ps -ef
ps -o pid,ppid,user,%cpu,%mem,stat,cmd -p <PID>
```

---

## Q7. What is a thread?

A thread is an execution context within a process.

Threads in the same process generally share:

- Address space
- Code
- Heap
- Open file descriptors

But each thread has its own:

- Stack
- Registers
- Execution state

Inspect threads:

```bash
ps -eLf
top -H -p <PID>
```

---

## Q8. What is a daemon?

A daemon is a background service designed to provide functionality without direct interactive control.

Examples:

```text
sshd
systemd
cron
nginx
```

Check:

```bash
systemctl status ssh
ps -ef | grep ssh
```

---

## Q9. What is a system call?

A system call is an interface through which user-space programs request services from the kernel.

Examples:

```text
open()
read()
write()
close()
fork()
execve()
mmap()
socket()
```

Observe system calls:

```bash
strace -p <PID>
strace ./program
```

---

## Q10. What is `/proc`?

`/proc` is a virtual filesystem exposing process and kernel/runtime information.

Examples:

```bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/loadavg
cat /proc/uptime
cat /proc/<PID>/status
ls -l /proc/<PID>/fd
```

It is not ordinary persistent disk storage.

---

# 2. Processes

## Q11. Process vs thread?

```text
Process
 ├── virtual address space
 ├── resources
 └── one or more threads

Thread
 ├── execution context
 ├── own stack
 └── shares process resources
```

Processes provide stronger isolation. Threads are lighter-weight but shared memory introduces synchronization concerns.

---

## Q12. What is PID?

PID = Process ID.

```bash
pgrep nginx
pidof nginx
ps -ef | grep nginx
```

Current shell:

```bash
echo $$
```

Last background process:

```bash
echo $!
```

---

## Q13. What is PPID?

PPID = Parent Process ID.

```bash
ps -o pid,ppid,cmd -p <PID>
```

Find parent:

```bash
ps -fp <PPID>
```

---

## Q14. What is a zombie process?

A zombie is a process that has terminated but whose parent has not yet collected its exit status.

It may appear with:

```text
Z
```

Find zombies:

```bash
ps -eo pid,ppid,stat,cmd | awk '$3 ~ /Z/'
```

Important:

> Killing a zombie itself is generally meaningless because the process has already terminated. The parent must reap it.

---

## Q15. What is an orphan process?

An orphan is a process whose original parent has terminated.

Linux reparents such processes to an appropriate system process, commonly PID 1 or a subreaper.

---

## Q16. What happens with `kill -9`?

```bash
kill -9 <PID>
```

sends `SIGKILL`.

The process cannot catch or ignore SIGKILL.

It should not normally be the first option because graceful cleanup is skipped.

Prefer:

```bash
kill -TERM <PID>
```

and investigate if the process does not terminate.

---

## Q17. SIGTERM vs SIGKILL?

| Signal | Meaning | Catch/handle? |
|---|---|---|
| SIGTERM | Graceful termination request | Yes |
| SIGKILL | Immediate forced termination | No |
| SIGINT | Interrupt | Yes |
| SIGHUP | Hangup/config reload convention | Yes |
| SIGSTOP | Stop process | No |
| SIGCONT | Continue stopped process | N/A |

---

## Q18. How do you find a process?

```bash
pgrep -a nginx
pidof nginx
ps -ef
ps aux
```

Search:

```bash
ps -ef | grep '[n]ginx'
```

---

## Q19. How do you kill a process?

Preferred:

```bash
kill -TERM <PID>
```

Then verify:

```bash
ps -p <PID>
```

Force only when appropriate:

```bash
kill -KILL <PID>
```

---

## Q20. How do you find the process consuming CPU?

```bash
top
```

or:

```bash
ps aux --sort=-%cpu | head
```

More detailed:

```bash
pidstat -u 1
```

For threads:

```bash
top -H -p <PID>
```

---

# 3. Memory

## Q21. What is virtual memory?

Virtual memory gives each process a virtual address space that the kernel maps to physical memory and other backing mechanisms.

Conceptually:

```text
Process virtual address
        ↓
Page tables
        ↓
Physical page / file-backed page / swap
```

Benefits include:

- Process isolation
- Flexible memory mapping
- Shared libraries
- Memory-mapped files
- More efficient resource management

---

## Q22. What is swap?

Swap is disk-backed space that can be used for memory pages under memory pressure.

Check:

```bash
free -h
swapon --show
```

Important:

> Swap is not equivalent to additional RAM. Heavy swapping can severely hurt performance.

---

## Q23. What is paging?

Paging divides virtual memory into fixed-size pages and physical memory into page frames.

The MMU and kernel cooperate to translate virtual addresses to physical addresses.

---

## Q24. What is a page fault?

A page fault occurs when a memory access requires translation or data that is not currently available in the expected memory state.

Not every page fault is bad.

Minor faults can be normal. Major faults may require storage I/O.

Inspect:

```bash
vmstat 1
pidstat -r 1
```

---

## Q25. What is OOM Killer?

When the kernel cannot satisfy memory allocation under severe memory pressure, the Out-Of-Memory mechanism can terminate selected processes to recover.

Look for evidence:

```bash
dmesg | grep -i -E 'oom|out of memory'
journalctl -k | grep -i oom
```

---

## Q26. How do you check memory usage?

```bash
free -h
vmstat 1
top
ps aux --sort=-%mem | head
cat /proc/meminfo
```

---

## Q27. Why can "free" memory be low on a healthy system?

Linux uses available RAM for useful purposes such as filesystem cache.

Check:

```bash
free -h
```

Focus on `available` memory and overall memory pressure rather than treating low `free` alone as failure.

---

## Q28. How do you troubleshoot high memory?

```text
Confirm pressure
      ↓
free -h
      ↓
Check swap
      ↓
vmstat 1
      ↓
Find large processes
      ↓
ps/top
      ↓
Check application behavior
      ↓
Check OOM events
      ↓
Check recent deployment/configuration changes
```

Commands:

```bash
free -h
vmstat 1
ps aux --sort=-%mem | head -20
journalctl -k | grep -i oom
```

---

# 4. Filesystem

## Q29. `df` vs `du`?

### `df`

Reports filesystem-level space usage.

```bash
df -h
```

### `du`

Estimates space consumed by files/directories.

```bash
du -sh /var/log/*
```

Think:

```text
df → filesystem perspective
du → directory/file perspective
```

---

## Q30. Hard link vs symbolic link?

Hard link:

```bash
ln file hardlink
```

Points to the same inode.

Symbolic link:

```bash
ln -s file symlink
```

Stores a path/reference to another filesystem object.

Check:

```bash
ls -li
```

A symbolic link can point across filesystems. A hard link normally cannot cross filesystem boundaries and generally cannot be made to directories by ordinary users.

---

## Q31. What is an inode?

An inode stores filesystem metadata such as:

- File type
- Permissions
- Owner/group
- Size
- Timestamps
- Links
- References to file data

Check:

```bash
ls -li file
df -i
```

A filesystem can run out of inodes even when it still has free disk space.

---

## Q32. What happens when a file is deleted?

`rm` removes a directory entry. The underlying file data is reclaimable when no directory entry and no open file descriptor references it.

This explains why an open deleted file can continue consuming space.

---

## Q33. How can disk space remain consumed after deleting a file?

Find deleted-but-open files:

```bash
lsof +L1
```

Typical cause:

```text
Large log file
   ↓
Application keeps file open
   ↓
File is deleted
   ↓
Directory entry disappears
   ↓
Process still holds descriptor
   ↓
Space remains allocated
```

Restarting/closing the relevant process may release the space, but determine the correct operational fix first.

---

## Q34. What is `/etc`?

System-wide configuration files.

Examples:

```text
/etc/hosts
/etc/passwd
/etc/ssh/
/etc/systemd/
```

---

## Q35. What is `/var`?

Variable data such as:

- Logs
- Caches
- Spools
- Application state

Common:

```text
/var/log
/var/lib
/var/cache
```

---

## Q36. What is `/tmp`?

Temporary files.

Do not assume every file under `/tmp` is automatically safe to delete. Application behavior and distribution-specific cleanup policies matter.

---

## Q37. What is `/proc`?

Virtual filesystem exposing process and kernel information.

See section 1.

---

## Q38. What is `/dev`?

A filesystem containing device nodes representing devices or pseudo-devices exposed to user space.

Examples:

```text
/dev/null
/dev/zero
/dev/random
/dev/sda
```

---

# 5. Permissions

## Q39. Explain `755`.

```text
755
│
├── Owner  = 7 = rwx
├── Group  = 5 = r-x
└── Others = 5 = r-x
```

Values:

```text
r = 4
w = 2
x = 1
```

---

## Q40. Explain `644`.

```text
Owner  = rw-
Group  = r--
Others = r--
```

Numeric:

```text
644
```

Common for ordinary files.

---

## Q41. What does `chmod` do?

Changes permission bits.

```bash
chmod 755 script.sh
chmod u+x script.sh
```

---

## Q42. `chmod` vs `chown`?

```text
chmod → permissions
chown → owner/group
```

Examples:

```bash
chmod 640 config
chown appuser:appgroup config
```

---

## Q43. What is `umask`?

`umask` controls which permission bits are removed from default permission creation.

Check:

```bash
umask
```

Example:

```bash
umask 027
```

The exact resulting permissions depend on the requested default mode and file type.

---

## Q44. What is ACL?

Access Control Lists provide more granular permissions than basic owner/group/other bits.

Check:

```bash
getfacl file
```

Set:

```bash
setfacl -m u:alice:r file
```

---

## Q45. How do you troubleshoot "Permission denied"?

Check in order:

```bash
id
ls -ld <path>
namei -l <path>
getfacl <path>
```

Then consider:

```text
Parent directory execute permission
File permission
Ownership
ACL
SELinux/AppArmor
Mount options
Application user
```

Useful:

```bash
namei -l /path/to/file
```

---

# 6. Networking

## Q46. TCP vs UDP?

| TCP | UDP |
|---|---|
| Connection-oriented | Connectionless |
| Reliable ordered byte stream | Datagram-based |
| Retransmission | No built-in delivery guarantee |
| Flow/congestion control | Minimal transport overhead |

Neither is universally "better"; application requirements determine the choice.

---

## Q47. What is a port?

A transport-layer endpoint identifier used to distinguish services/applications on a host.

Example:

```text
IP: 192.168.1.10
Port: 8080
```

Together with protocol and IP addressing, this identifies a network endpoint.

---

## Q48. What is a socket?

A socket is an OS abstraction used by applications for network communication.

Conceptually:

```text
Application
    ↓
Socket API
    ↓
Transport
    ↓
IP
    ↓
Network
```

---

## Q49. What does `ss` show?

`ss` displays socket information.

Listening TCP ports:

```bash
ss -lnt
```

Listening TCP/UDP with processes:

```bash
ss -lntup
```

All established TCP connections:

```bash
ss -tn
```

---

## Q50. `ping` vs `traceroute`?

`ping` tests reachability/round-trip behavior using ICMP echo or implementation-specific mechanisms.

`traceroute` attempts to reveal the network path hop-by-hop.

Neither alone proves that an application is healthy.

For an HTTP service:

```bash
curl -v https://example.com
```

is often more relevant.

---

## Q51. What is DNS?

DNS maps names to resource records such as IP addresses.

Example:

```text
example.com
      ↓
DNS
      ↓
IP address
```

Query:

```bash
dig example.com
dig example.com A
dig example.com AAAA
```

---

## Q52. How do you troubleshoot DNS?

```text
Does the name resolve?
        ↓
dig / resolvectl
        ↓
Correct DNS server?
        ↓
Correct record?
        ↓
Can the resolved IP be reached?
        ↓
Does the application work by IP?
```

Commands:

```bash
dig example.com
resolvectl status
cat /etc/resolv.conf
```

---

## Q53. How do you check listening ports?

```bash
ss -lntup
```

For a specific port:

```bash
ss -lntp 'sport = :8080'
```

---

## Q54. How do you find which process owns port 8080?

```bash
ss -lntp | grep ':8080'
```

or:

```bash
lsof -i :8080
```

---

## Q55. How do you troubleshoot an unreachable server?

Use layers:

```text
1. Local interface
2. Local route
3. DNS
4. Remote reachability
5. TCP port
6. Firewall
7. Application
```

Commands:

```bash
ip addr
ip route
ping <host>
traceroute <host>
nc -vz <host> <port>
curl -v http://<host>:<port>
ss -s
```

---

# 7. systemd

## Q56. How do you start a service?

```bash
sudo systemctl start nginx
```

Check:

```bash
systemctl status nginx
```

---

## Q57. How do you stop a service?

```bash
sudo systemctl stop nginx
```

---

## Q58. How do you restart a service?

```bash
sudo systemctl restart nginx
```

For configuration reload where supported:

```bash
sudo systemctl reload nginx
```

Restart and reload are not interchangeable.

---

## Q59. How do you enable a service at boot?

```bash
sudo systemctl enable nginx
```

Enable and start:

```bash
sudo systemctl enable --now nginx
```

---

## Q60. How do you view service logs?

```bash
journalctl -u nginx
```

Recent:

```bash
journalctl -u nginx -n 100
```

Follow:

```bash
journalctl -u nginx -f
```

---

## Q61. What is `journalctl`?

`journalctl` queries the systemd journal.

Useful:

```bash
journalctl -b
journalctl -p err
journalctl -k
journalctl -u nginx
journalctl --since "1 hour ago"
```

---

## Q62. How do you troubleshoot a failed service?

```bash
systemctl status <service>
journalctl -u <service> -n 100
```

Then check:

```text
Configuration
Dependencies
Permissions
Environment variables
Port conflicts
Disk space
Resource limits
Certificate expiry
Network dependencies
Recent changes
```

---

# 8. High-Value Scenario Questions

# Scenario 1 — CPU Suddenly Reaches 100%

### Interview answer

> First I confirm whether CPU is actually saturated and identify whether the load is caused by user CPU, system CPU, I/O wait, or a specific process. Then I identify the process/thread responsible, correlate it with recent changes and application logs, mitigate safely, and verify recovery.

Commands:

```bash
uptime
top
ps aux --sort=-%cpu | head -20
mpstat -P ALL 1
pidstat -u 1
```

If one process is responsible:

```bash
top -H -p <PID>
strace -p <PID>
```

Do not immediately run:

```bash
kill -9
```

Find the cause first unless there is an urgent operational reason to terminate it.

---

# Scenario 2 — Server is Slow

Do not answer only "check CPU."

Check:

```text
CPU
Memory
Disk I/O
Network
Load
Processes
Application
Dependencies
```

Commands:

```bash
uptime
top
free -h
vmstat 1
iostat -xz 1
ss -s
```

Then inspect application-specific logs and recent changes.

---

# Scenario 3 — Disk is 100% Full

Start:

```bash
df -h
```

Identify the filesystem.

Then:

```bash
du -xhd1 /var | sort -h
```

Look for large files:

```bash
find /var -xdev -type f -size +1G -ls
```

If `du` does not explain the usage:

```bash
lsof +L1
```

Also check inode exhaustion:

```bash
df -i
```

---

# Scenario 4 — Port Is Not Reachable

First determine whether anything is listening:

```bash
ss -lntp
```

Then:

```bash
ps -ef | grep <application>
```

Local test:

```bash
curl -v http://127.0.0.1:8080
```

Then test externally.

Possible causes:

```text
Application stopped
Wrong bind address
Wrong port
Firewall
Security group/network ACL
Routing
Reverse proxy
Application failure
```

---

# Scenario 5 — Service Won't Start

```bash
systemctl status nginx
journalctl -u nginx -n 100
```

Then check:

```bash
nginx -t
ss -lntp
df -h
```

Typical causes:

```text
Invalid configuration
Port already occupied
Missing dependency
Permission problem
Missing file
Certificate problem
Environment variable
Disk full
```

---

# Scenario 6 — Application Is Down but Process Is Running

Check:

```bash
ps -fp <PID>
ss -lntp
lsof -p <PID>
curl -v localhost:<PORT>
```

Then logs:

```bash
journalctl -u <service>
tail -f application.log
```

Reason through:

```text
Process exists
      ≠
Application is healthy
```

A process can be alive while:

- Its worker threads are stuck
- Its listening socket is unavailable
- It is deadlocked
- It cannot reach dependencies
- It is returning errors
- It is out of useful resources

---

# Scenario 7 — Memory Is 90% Used

Do not automatically call it a memory leak.

Check:

```bash
free -h
vmstat 1
ps aux --sort=-%mem | head -20
swapon --show
```

Then:

```bash
journalctl -k | grep -i oom
```

Questions:

```text
Is available memory low?
Is swap active?
Is swap heavily used?
Is one process growing?
Is cache reclaimable?
Are there OOM events?
Did a deployment change memory behavior?
```

---

# Scenario 8 — DNS Works but Application Does Not

Separate layers:

```text
DNS
 ↓
IP
 ↓
Route
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
Application
```

Commands:

```bash
dig example.com
ip route
ss -tn
curl -v https://example.com
```

If DNS returns an IP but `curl` fails, DNS may not be the problem.

---

# Scenario 9 — SSH Is Not Working

Server-side:

```bash
systemctl status sshd
ss -lntp | grep ':22'
journalctl -u sshd
```

Client-side:

```bash
ssh -vvv user@host
```

Check:

```text
DNS
Network reachability
Port 22
Firewall
sshd status
Authentication
Permissions
SSH keys
User account
```

---

# Scenario 10 — "Permission Denied"

Start:

```bash
id
namei -l /path/to/file
ls -l /path/to/file
getfacl /path/to/file
```

Then check:

```text
Parent directory permissions
File permissions
Owner/group
ACL
SELinux/AppArmor
Mount options
Actual service user
```

---

# 9. Essential Bash Scripts

## Script 1 — System Health Check

```bash
#!/usr/bin/env bash

set -u

echo "===== SYSTEM HEALTH ====="

echo
echo "Hostname:"
hostname

echo
echo "Uptime:"
uptime

echo
echo "Load:"
cat /proc/loadavg

echo
echo "Memory:"
free -h

echo
echo "Disk:"
df -h

echo
echo "Top CPU processes:"
ps aux --sort=-%cpu | head -6

echo
echo "Top memory processes:"
ps aux --sort=-%mem | head -6

echo
echo "Listening ports:"
ss -lntup

echo
echo "===== END ====="
```

Run:

```bash
chmod +x system-health.sh
./system-health.sh
```

---

# Script 2 — Disk Usage Check

```bash
#!/usr/bin/env bash

set -u

THRESHOLD=80

df -P -x tmpfs -x devtmpfs | tail -n +2 |
while read -r filesystem blocks used available percent mountpoint
do
    usage=${percent%\%}

    if [ "$usage" -ge "$THRESHOLD" ]; then
        echo "WARNING: $mountpoint is ${usage}% full"
    fi
done
```

This is a basic operational script. In production, handle unusual mountpoint names and command failures more defensively.

---

# Script 3 — Check a Service

```bash
#!/usr/bin/env bash

SERVICE="${1:-nginx}"

if systemctl is-active --quiet "$SERVICE"; then
    echo "$SERVICE is running"
else
    echo "$SERVICE is NOT running"
    systemctl status "$SERVICE" --no-pager
fi
```

Run:

```bash
./check-service.sh nginx
```

---

# Script 4 — Check a Port

```bash
#!/usr/bin/env bash

HOST="${1:-127.0.0.1}"
PORT="${2:-8080}"

if timeout 3 bash -c "</dev/tcp/$HOST/$PORT" 2>/dev/null; then
    echo "$HOST:$PORT is reachable"
else
    echo "$HOST:$PORT is NOT reachable"
fi
```

---

# 10. Log Analysis Commands

Find errors:

```bash
grep -i "error" application.log
```

Last 100 errors:

```bash
grep -i "error" application.log | tail -100
```

Follow logs:

```bash
tail -f application.log
```

Multiple patterns:

```bash
grep -Ei "error|exception|failed|timeout" application.log
```

Count status codes:

```bash
awk '{print $9}' access.log | sort | uniq -c | sort -nr
```

Top client IPs:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head
```

---

# 11. Performance Command Matrix

| Area | Commands |
|---|---|
| Load | `uptime`, `w` |
| CPU | `top`, `mpstat`, `pidstat` |
| Memory | `free`, `vmstat`, `pidstat` |
| Disk space | `df`, `du` |
| Disk I/O | `iostat`, `pidstat -d` |
| Network | `ss`, `ip`, `sar` |
| Processes | `ps`, `pgrep`, `top` |
| Syscalls | `strace` |
| Profiling | `perf` |
| Kernel logs | `dmesg`, `journalctl -k` |

---

# 12. Important Linux Files

```text
/etc/passwd       users
/etc/shadow       password hashes / account data
/etc/group        groups
/etc/hosts        local hostname mapping
/etc/resolv.conf  resolver configuration
/etc/fstab        filesystem mounts
/etc/ssh/         SSH configuration
/var/log/         logs
/proc/            runtime kernel/process information
/sys/             kernel/device information
/dev/             device nodes
/tmp/             temporary files
```

---

# 13. High-Value One-Liners

### Find top CPU processes

```bash
ps aux --sort=-%cpu | head
```

### Find top memory processes

```bash
ps aux --sort=-%mem | head
```

### Find process by name

```bash
pgrep -a nginx
```

### Find process using port

```bash
lsof -i :8080
```

### Find listening ports

```bash
ss -lntup
```

### Find largest directories

```bash
du -xhd1 /var | sort -h
```

### Find files larger than 1 GB

```bash
find / -xdev -type f -size +1G -ls 2>/dev/null
```

### Follow a log

```bash
tail -F application.log
```

### Find errors in recent logs

```bash
journalctl -p err --since "1 hour ago"
```

### Find deleted open files

```bash
lsof +L1
```

### Check current routes

```bash
ip route
```

### Check DNS

```bash
dig example.com
```

### Check HTTP

```bash
curl -v https://example.com
```

---

# 14. Advanced Concepts to Know

For advanced Linux interviews, don't stop at commands.

Study:

```text
Process lifecycle
Scheduling
Context switching
Virtual memory
Page tables
Memory mapping
File descriptors
Inodes
VFS
System calls
Signals
IPC
Pipes
Sockets
Namespaces
cgroups
Capabilities
SELinux/AppArmor
systemd
Containers
eBPF
```

---

# 15. Common Interview Traps

## Trap 1

**"CPU is 100%, so the server is overloaded."**

Not necessarily.

Determine:

```text
user CPU
system CPU
iowait
steal
per-core distribution
load average
```

---

## Trap 2

**"Memory is 90%, so there is a memory leak."**

Not necessarily.

Check:

```text
available memory
swap
cache
process growth
OOM events
```

---

## Trap 3

**"Ping works, so the application is reachable."**

Not necessarily.

Ping tests a different layer.

Test the actual service:

```bash
curl
nc
ss
```

---

## Trap 4

**"The process exists, so the application is healthy."**

False.

A process can be alive while its workers are stuck or its dependencies are unavailable.

---

## Trap 5

**"Kill -9 fixes the problem."**

It terminates the process but does not explain the root cause.

---

## Trap 6

**"df and du show the same thing."**

They don't.

```text
df → filesystem allocation
du → files/directories
```

---

# 16. Fast Troubleshooting Cheat Sheet

## CPU

```bash
uptime
top
ps aux --sort=-%cpu | head
mpstat -P ALL 1
pidstat -u 1
```

## Memory

```bash
free -h
vmstat 1
ps aux --sort=-%mem | head
swapon --show
journalctl -k | grep -i oom
```

## Disk

```bash
df -h
df -i
du -xhd1 / | sort -h
lsof +L1
```

## Network

```bash
ip addr
ip route
ss -lntup
ping <host>
traceroute <host>
dig <domain>
curl -v <url>
```

## Process

```bash
ps -ef
pgrep -a <name>
top
lsof -p <PID>
```

## Service

```bash
systemctl status <service>
journalctl -u <service> -n 100
```

## Logs

```bash
tail -F
grep
awk
sed
journalctl
```

---

# 17. The 30-Second Interview Framework

When asked:

> "A production Linux server is slow. What do you do?"

Answer structurally:

```text
1. Confirm the symptom.
2. Determine scope and impact.
3. Check recent changes.
4. Check CPU/load.
5. Check memory/swap.
6. Check disk space/I/O.
7. Check network.
8. Check processes/services.
9. Check application/system logs.
10. Form and test a hypothesis.
11. Apply the safest mitigation.
12. Verify recovery.
13. Identify root cause.
14. Add prevention/monitoring.
```

This is much stronger than listing commands randomly.

---

# 18. Final Interview Revision

Memorize these distinctions:

```text
Process vs Thread
SIGTERM vs SIGKILL
df vs du
Hard link vs Symlink
TCP vs UDP
ping vs curl
CPU usage vs load average
free vs available memory
chmod vs chown
systemctl vs journalctl
DNS vs HTTP
Process alive vs application healthy
Filesystem full vs inode full
```

Know these commands cold:

```text
ps
top
free
vmstat
df
du
lsblk
lsof
ss
ip
ping
traceroute
curl
dig
grep
awk
sed
find
systemctl
journalctl
kill
chmod
chown
mount
```

And know how to explain:

```text
/proc
system calls
signals
virtual memory
inodes
file descriptors
zombies
orphans
OOM
systemd
DNS
TCP
permissions
load average
```

---

# 19. Golden Rule

Don't answer Linux troubleshooting questions with:

> "I will run `top`, then `free`, then `df`."

Answer with:

> **"I will first define the symptom and scope, collect evidence from the relevant resource layer, form a hypothesis, test it, mitigate safely, verify recovery, and then identify the root cause."**

Commands are tools.

**Reasoning is the skill.**
