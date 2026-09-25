# Linux-Production-Notes
Fast, practical Linux reference covering commands, administration, troubleshooting, networking, processes, permissions, performance, Bash, systemd, logs, security, and interview questions.


# 🐧 Linux Quick Reference

> **Fast last-minute notes for Linux administration, troubleshooting, networking, performance, Bash, systemd, security, and interviews.**

A practical Linux reference built for **learning, revision, troubleshooting, production support, and technical interviews**.

This repository focuses on one question:

> **"The Linux system is behaving unexpectedly. How do I find out why?"**

---

## 🎯 What This Repository Covers

```text
Linux Fundamentals
        ↓
Files & Permissions
        ↓
Processes & Threads
        ↓
Memory Management
        ↓
Storage & I/O
        ↓
Networking
        ↓
systemd & Services
        ↓
Logs & Text Processing
        ↓
Bash Scripting
        ↓
Performance Analysis
        ↓
Security
        ↓
Kernel & /proc
        ↓
Production Troubleshooting
        ↓
Interview Preparation
```

---

# ⚡ 60-Second Linux Cheat Sheet

## System Information

```bash
uname -a
hostname
hostnamectl
uptime
date
whoami
id
```

## CPU

```bash
lscpu
top
htop
mpstat
uptime
```

## Memory

```bash
free -h
vmstat 1
cat /proc/meminfo
```

## Disk

```bash
df -h
du -sh *
lsblk
blkid
mount
```

## Processes

```bash
ps aux
ps -ef
top
pgrep <name>
pidof <name>
kill <PID>
kill -9 <PID>
```

## Networking

```bash
ip addr
ip route
ss -tulpn
ping <host>
traceroute <host>
curl <url>
dig <domain>
nslookup <domain>
```

## Logs

```bash
journalctl
journalctl -u <service>
journalctl -f
tail -f /var/log/<file>
grep "ERROR" <file>
```

## Services

```bash
systemctl status <service>
systemctl start <service>
systemctl stop <service>
systemctl restart <service>
systemctl enable <service>
systemctl disable <service>
```

## Open Files

```bash
lsof
lsof -i :8080
lsof -p <PID>
```

---

# 🧠 The Linux Troubleshooting Mindset

Do not randomly execute commands until something looks wrong.

Start with:

```text
What is the symptom?
        ↓
What changed?
        ↓
Is the issue reproducible?
        ↓
Is it CPU?
Memory?
Disk?
Network?
Process?
Service?
Application?
        ↓
Collect evidence
        ↓
Form hypothesis
        ↓
Test hypothesis
        ↓
Fix
        ↓
Verify
        ↓
Prevent recurrence
```

For performance investigations, the **USE Method** is a useful framework:

> **Utilization → Saturation → Errors**

Check resources systematically rather than jumping between unrelated commands. Brendan Gregg documents this methodology specifically for identifying resource bottlenecks and errors.

---

# 📚 Core Topics

## 01 — Linux Basics

* Linux architecture
* Kernel vs user space
* Shell
* Filesystem hierarchy
* Absolute vs relative paths
* Environment variables
* stdin / stdout / stderr
* Pipes
* Redirection
* Wildcards
* Command substitution

---

## 02 — Files & Permissions

Understand:

```text
r = read
w = write
x = execute
```

Example:

```bash
-rwxr-xr--
```

Breakdown:

```text
Owner   Group   Others
rwx     r-x     r--
```

Important commands:

```bash
chmod
chown
chgrp
umask
getfacl
setfacl
```

---

# 03 — Processes

A process is a running instance of a program.

Useful commands:

```bash
ps
top
htop
pgrep
pkill
kill
killall
nice
renice
jobs
bg
fg
```

Important concepts:

```text
PID
PPID
Process states
Foreground/background
Signals
Zombie
Orphan
Daemon
```

---

# 04 — Signals

Common signals:

```text
SIGTERM  → request graceful termination
SIGKILL  → force termination
SIGINT   → interrupt
SIGHUP   → hangup
SIGSTOP  → stop
SIGCONT  → continue
```

Example:

```bash
kill -TERM <PID>
```

Use `SIGKILL` carefully:

```bash
kill -9 <PID>
```

It does not give the process an opportunity to clean up normally.

---

# 05 — Memory

Important concepts:

```text
Physical memory
Virtual memory
Pages
Swap
Cache
Buffers
OOM Killer
Page faults
```

Useful commands:

```bash
free -h
vmstat 1
top
ps aux --sort=-%mem
cat /proc/meminfo
```

### Quick diagnosis

```text
High memory usage
      ↓
free -h
      ↓
Is available memory low?
      ↓
Check swap
      ↓
vmstat
      ↓
Find memory-heavy processes
      ↓
Investigate application
```

---

# 06 — Storage

Useful commands:

```bash
df -h
du -sh *
lsblk
mount
findmnt
blkid
lsof
```

### Important distinction

```text
df
→ filesystem space

du
→ space consumed by files/directories
```

If:

```text
df says disk is full
du doesn't explain it
```

investigate deleted-but-open files:

```bash
lsof +L1
```

---

# 07 — Networking

Core concepts:

```text
IP
MAC
ARP
DNS
TCP
UDP
Ports
Routing
Sockets
NAT
HTTP
HTTPS
TLS
```

Useful commands:

```bash
ip addr
ip route
ss -tulpn
ping
traceroute
curl
dig
nslookup
```

### Port troubleshooting

```bash
ss -lntp
```

Check whether a service is listening.

Then:

```bash
curl localhost:<PORT>
```

Then investigate:

```text
Application
    ↓
Process
    ↓
Socket
    ↓
Firewall
    ↓
Route
    ↓
Remote server
```

---

# 08 — DNS Troubleshooting

When a hostname fails:

```bash
dig example.com
```

Check:

```text
DNS resolution
        ↓
IP address
        ↓
Routing
        ↓
TCP connectivity
        ↓
Application protocol
```

Do not assume a "website is down" when DNS is the actual problem.

---

# 09 — systemd

Important commands:

```bash
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl enable nginx
systemctl disable nginx
```

Logs:

```bash
journalctl -u nginx
journalctl -u nginx -f
journalctl -b
journalctl -p err
```

Typical service investigation:

```text
Service failed
      ↓
systemctl status
      ↓
journalctl
      ↓
Check configuration
      ↓
Check dependencies
      ↓
Check port
      ↓
Check permissions
      ↓
Restart
      ↓
Verify
```

---

# 10 — Logs

Important tools:

```bash
cat
less
tail
head
grep
awk
sed
cut
sort
uniq
wc
xargs
```

Real-world example:

```bash
grep "ERROR" application.log | tail -50
```

Count error types:

```bash
grep "ERROR" application.log \
  | awk '{print $NF}' \
  | sort \
  | uniq -c \
  | sort -nr
```

---

# 11 — Bash

Core topics:

```text
Variables
Arguments
Exit codes
Conditions
Loops
Functions
Arrays
Command substitution
Pipes
Redirection
Signals
Trap
Cron
```

Example:

```bash
#!/bin/bash

if systemctl is-active --quiet nginx; then
    echo "nginx is running"
else
    echo "nginx is NOT running"
fi
```

Always check exit codes when writing operational scripts:

```bash
echo $?
```

---

# 12 — Performance Troubleshooting

Start with a structured investigation.

## CPU

```bash
top
mpstat -P ALL 1
pidstat -u 1
```

## Memory

```bash
free -h
vmstat 1
pidstat -r 1
```

## Disk I/O

```bash
iostat -xz 1
pidstat -d 1
```

## Network

```bash
ss -s
sar -n DEV 1
ip -s link
```

## System calls

```bash
strace -p <PID>
```

## Advanced profiling

```bash
perf
```

For deeper performance work, investigate:

```text
CPU profiling
Off-CPU analysis
Flame graphs
perf events
eBPF
ftrace
```

Brendan Gregg's Linux performance material provides deeper references for these techniques.

---

# 13 — `/proc`

`/proc` is a pseudo-filesystem exposing kernel and process information to user space.

Examples:

```bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/loadavg
cat /proc/uptime
```

For a process:

```bash
cat /proc/<PID>/status
ls -l /proc/<PID>/fd
```

Understanding `/proc` makes many Linux tools easier to reason about rather than treating them as magic commands.

---

# 14 — Security

Important topics:

```text
Users
Groups
sudo
SSH
File permissions
ACL
SSH keys
Firewall
Processes
Capabilities
SELinux/AppArmor
```

Useful commands:

```bash
id
who
w
last
sudo
ssh
ss
```

Never blindly execute commands copied from the internet as root.

---

# 🚨 Production Troubleshooting Scenarios

## Scenario 1 — CPU is 100%

Start:

```bash
uptime
top
ps aux --sort=-%cpu | head
mpstat -P ALL 1
```

Questions:

* Is one process responsible?
* Is CPU evenly distributed?
* Is there high load but low CPU utilization?
* Is the process actually CPU-bound?
* Did a deployment happen recently?

---

## Scenario 2 — Server is slow

Don't immediately blame CPU.

Check:

```text
CPU
Memory
Disk I/O
Network
Load
Processes
Application
External dependencies
```

Use the USE framework where appropriate.

---

## Scenario 3 — Disk is full

```bash
df -h
du -xhd1 /
```

Then:

```bash
find /var -type f -size +1G
```

Also investigate:

```bash
lsof +L1
```

---

## Scenario 4 — Port is not reachable

```bash
ss -lntp
```

Check:

```text
Is process running?
       ↓
Is application listening?
       ↓
Is it bound to correct IP?
       ↓
Firewall?
       ↓
Routing?
       ↓
Remote connectivity?
```

---

## Scenario 5 — Service won't start

```bash
systemctl status <service>
journalctl -u <service> -n 100
```

Then check:

```text
Configuration
Permissions
Dependencies
Port conflicts
Environment variables
Disk space
Resource limits
```

---

# 🎯 Most Asked Linux Interview Questions

## Fundamentals

1. What is Linux?
2. What is the Linux kernel?
3. Kernel vs shell?
4. Linux vs Unix?
5. What happens when you execute a command?
6. What is a process?
7. What is a thread?
8. What is a daemon?
9. What is a system call?
10. What is `/proc`?

---

## Processes

11. Process vs thread?
12. What is a PID?
13. What is PPID?
14. What is a zombie process?
15. What is an orphan process?
16. What happens when you run `kill -9`?
17. SIGTERM vs SIGKILL?
18. How do you find a process?
19. How do you kill a process?
20. How do you find which process is consuming CPU?

---

## Memory

21. What is virtual memory?
22. What is swap?
23. What is paging?
24. What is a page fault?
25. What is OOM Killer?
26. How do you check memory usage?
27. Why can `free` memory appear low even when the system is healthy?
28. How would you troubleshoot high memory usage?

---

## Filesystem

29. `df` vs `du`?
30. Hard link vs symbolic link?
31. What is an inode?
32. What happens when a file is deleted?
33. How can disk space remain consumed after deleting a file?
34. What is `/etc`?
35. What is `/var`?
36. What is `/tmp`?
37. What is `/proc`?
38. What is `/dev`?

---

## Permissions

39. Explain `755`.
40. Explain `644`.
41. What does `chmod` do?
42. `chmod` vs `chown`?
43. What is `umask`?
44. What is ACL?
45. How do you troubleshoot "Permission denied"?

---

## Networking

46. TCP vs UDP?
47. What is a port?
48. What is a socket?
49. What does `ss` show?
50. `ping` vs `traceroute`?
51. What is DNS?
52. How do you troubleshoot DNS?
53. How do you check listening ports?
54. How do you find which process owns port 8080?
55. How would you troubleshoot a server that is unreachable?

---

## systemd

56. How do you start a service?
57. How do you stop a service?
58. How do you restart a service?
59. How do you enable a service at boot?
60. How do you view service logs?
61. What is `journalctl`?
62. How do you troubleshoot a failed service?

---

# 🔥 Scenario-Based Interview Questions

### Q1. Server CPU suddenly reaches 100%. What do you do?

Don't answer:

> "I will run `top`."

A stronger answer:

```text
1. Confirm the symptom.
2. Check load average.
3. Identify CPU-consuming processes.
4. Determine whether CPU usage is user/system/iowait.
5. Check per-core utilization.
6. Inspect the responsible process.
7. Check recent deployments/configuration changes.
8. Determine root cause.
9. Mitigate safely.
10. Verify recovery.
11. Document prevention.
```

---

### Q2. Server has 90% memory usage. Is that automatically a problem?

No.

Investigate:

```bash
free -h
vmstat 1
ps aux --sort=-%mem
```

Look at:

```text
available memory
swap activity
reclaimable cache
OOM events
application behavior
```

High memory utilization by itself does not establish that the system is unhealthy.

---

### Q3. Application is down but process is running.

Investigate:

```text
Process
 ↓
Listening socket
 ↓
Application logs
 ↓
Local request
 ↓
Firewall
 ↓
Network path
 ↓
Dependency
```

Useful commands:

```bash
ps
ss
curl
journalctl
grep
lsof
```

---

### Q4. Disk says 100% full but `du` doesn't explain it.

Investigate:

```bash
lsof +L1
```

A process may still have an open file descriptor for a deleted file.

---

# 🧰 Essential Commands by Problem

| Problem     | First Commands                   |
| ----------- | -------------------------------- |
| CPU         | `top`, `ps`, `mpstat`, `pidstat` |
| Memory      | `free`, `vmstat`, `ps`           |
| Disk space  | `df`, `du`                       |
| Disk I/O    | `iostat`, `pidstat`              |
| Process     | `ps`, `pgrep`, `top`             |
| Port        | `ss`, `lsof`                     |
| Network     | `ip`, `ss`, `ping`, `curl`       |
| DNS         | `dig`, `nslookup`                |
| Service     | `systemctl`, `journalctl`        |
| Logs        | `grep`, `tail`, `less`           |
| Kernel      | `dmesg`, `/proc`                 |
| Syscalls    | `strace`                         |
| Performance | `perf`, `vmstat`, `iostat`       |

---

# 🧭 Troubleshooting Decision Tree

```text
             INCIDENT
                 │
                 ▼
        What is the symptom?
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
      CPU      Memory     Network
       │         │         │
      top      free       ip
      ps       vmstat     ss
      mpstat   ps         ping
       │         │         │
       └─────────┼─────────┘
                 ▼
              Storage
                 │
             df / du
             iostat
                 │
                 ▼
              Process
                 │
             ps / lsof
                 │
                 ▼
              Service
                 │
       systemctl / journalctl
                 │
                 ▼
              Logs
                 │
        grep / awk / sed
                 │
                 ▼
            Root Cause
                 │
                 ▼
              Fix
                 │
                 ▼
             Verify
                 │
                 ▼
            Prevent
```

---

# 📖 Recommended References

### Official / Primary Documentation

* [Linux Kernel Documentation](https://docs.kernel.org/)
* [Linux man-pages](https://man7.org/linux/man-pages/)
* [GNU Coreutils](https://www.gnu.org/software/coreutils/manual/)
* [Bash Reference Manual](https://www.gnu.org/software/bash/manual/)
* [systemd Documentation](https://www.freedesktop.org/wiki/Software/systemd/)
* [OpenSSH Documentation](https://www.openssh.com/manual.html)

### Performance & Troubleshooting

* [Brendan Gregg — Linux Performance](https://www.brendangregg.com/linuxperf.html)
* [Brendan Gregg — Performance Methodology](https://www.brendangregg.com/methodology.html)
* [Brendan Gregg — USE Method](https://www.brendangregg.com/usemethod.html)
* [Brendan Gregg — perf examples](https://www.brendangregg.com/perf.html)
* [Linux `proc(5)` documentation](https://man7.org/linux/man-pages/man5/proc.5.html)

The `/proc` filesystem exposes kernel data structures and process/system information, making it an important source for understanding how Linux exposes runtime state.

---

# 📚 Recommended Book

**Systems Performance: Enterprise and the Cloud — Brendan Gregg**

Use it as a deeper reference for:

```text
CPU
Memory
Disks
Networking
Filesystems
Kernel
Performance methodology
Observability
```

The Linux USE checklist is included as an appendix in the second edition.

---

# 🧪 Hands-On Labs

This repository will eventually include practical labs for:

* High CPU
* Memory pressure
* Disk full
* Disk I/O bottleneck
* Zombie processes
* Hanging processes
* Port conflicts
* DNS failure
* Network latency
* Service failure
* Permission problems
* Log analysis
* SSH problems
* File descriptor exhaustion
* OOM conditions

---

# 🚀 Goal

The goal is not to memorize 200 Linux commands.

The goal is to be able to look at a Linux system and reason:

```text
What is happening?
       ↓
Where is it happening?
       ↓
Why is it happening?
       ↓
How can I prove it?
       ↓
How do I fix it?
       ↓
How do I prevent it?
```

> **Learn the command. Understand the system. Diagnose the problem.**

---

## 👨‍💻 Author

**Prateek**

Software Engineering • Linux • Java • SQL • Cloud • Systems

Learning by building, troubleshooting, and documenting.
