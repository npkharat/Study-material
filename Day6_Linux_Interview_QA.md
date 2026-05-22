# Day 6: Linux

---

## Table of Contents
1. [Core Concepts (Q1–Q20)](#1-core-concepts)
2. [File System & Permissions (Q21–Q35)](#2-file-system--permissions)
3. [Process Management (Q36–Q48)](#3-process-management)
4. [Networking (Q49–Q62)](#4-networking)
5. [Shell Scripting (Q63–Q75)](#5-shell-scripting)
6. [System Administration (Q76–Q90)](#6-system-administration)
7. [Advanced & Real-World (Q91–Q110)](#7-advanced--real-world)

---

## 1. Core Concepts

**Q1. What is Linux and why is it used in DevOps?**
> Linux is an open-source Unix-like operating system. Used in DevOps because:
> - Free and open-source
> - Highly stable and secure
> - Most servers, containers (Docker), and cloud instances run Linux
> - Powerful command-line tools for automation
> - Supports scripting (Bash, Python)

---

**Q2. What is the difference between Linux distributions?**
> | Distribution | Base | Package Manager | Use Case |
> |---|---|---|---|
> | Ubuntu | Debian | apt | Servers, desktops, cloud |
> | CentOS/RHEL | RedHat | yum/dnf | Enterprise servers |
> | Amazon Linux | RHEL/Fedora | yum/dnf | AWS EC2 |
> | Alpine | Independent | apk | Docker containers |
> | Debian | - | apt | Stable servers |

---

**Q3. What is the Linux Kernel?**
> The kernel is the core of the OS. It manages:
> - Hardware (CPU, memory, disk, network)
> - Process scheduling
> - System calls
> - Device drivers
> Everything else (shell, applications) runs on top of the kernel.

---

**Q4. What is a Shell?**
> A Shell is a command-line interface to interact with the OS. Common shells:
> - **bash** (Bourne Again Shell) — most common on Linux
> - **sh** (Bourne Shell) — POSIX standard
> - **zsh** — improved bash (default on Mac)
> - **fish** — user-friendly
> Check current shell: `echo $SHELL`

---

**Q5. What is the difference between `su` and `sudo`?**
> - `su` — Switch User. `su -` switches to root. Requires root password.
> - `sudo` — Execute ONE command as root. Requires YOUR password. Logged in `/var/log/auth.log`.
> `sudo` is safer — gives temporary privilege without full root access.

---

**Q6. What is the Linux boot process?**
> ```
> Power ON
>   → BIOS/UEFI (hardware check)
>   → Bootloader (GRUB) - loads kernel
>   → Kernel - initializes hardware, mounts root filesystem
>   → init/systemd (PID 1) - starts all services
>   → Login prompt / Shell
> ```

---

**Q7. What is systemd?**
> systemd is the init system (PID 1) on modern Linux. It:
> - Starts and manages services (daemons)
> - Handles dependencies between services
> - Mounts filesystems
> - Manages logging (journald)
```bash
systemctl start nginx       # start service
systemctl stop nginx        # stop service
systemctl restart nginx     # restart
systemctl reload nginx      # reload config (no restart)
systemctl enable nginx      # start on boot
systemctl disable nginx     # don't start on boot
systemctl status nginx      # check status
systemctl list-units        # list all services
journalctl -u nginx         # view service logs
journalctl -f               # follow system logs
```

---

**Q8. What is the difference between `/bin`, `/sbin`, `/usr/bin`, `/usr/local/bin`?**
> - `/bin` — Essential user binaries (ls, cp, mv, bash)
> - `/sbin` — System binaries for root (fdisk, iptables, ifconfig)
> - `/usr/bin` — Non-essential user binaries (git, python, vim)
> - `/usr/local/bin` — Locally installed software (your custom scripts, tools)
> - `/opt` — Optional/third-party software

---

**Q9. What is the Linux filesystem hierarchy?**
> | Directory | Purpose |
> |---|---|
> | `/` | Root of filesystem |
> | `/home` | User home directories |
> | `/root` | Root user's home |
> | `/etc` | Configuration files |
> | `/var` | Variable data (logs, caches, databases) |
> | `/tmp` | Temporary files (cleared on reboot) |
> | `/proc` | Virtual filesystem - kernel/process info |
> | `/sys` | Virtual filesystem - hardware info |
> | `/dev` | Device files |
> | `/mnt` | Mount points |
> | `/var/log` | System logs |

---

**Q10. What is an inode?**
> An inode is a data structure that stores metadata about a file:
> - File size, permissions, ownership
> - Timestamps (created, modified, accessed)
> - Pointer to data blocks on disk
> - Does NOT store filename (directory maps name → inode)
```bash
ls -i file.txt       # show inode number
stat file.txt        # detailed inode info
df -i                # inode usage per filesystem
```

---

**Q11. What is the difference between hard link and soft (symbolic) link?**
> - **Hard Link:** Another name pointing to the same inode. Same file, different name. Still works if original is deleted. Can't cross filesystems.
> - **Soft Link (Symlink):** Pointer to the filename (like a shortcut). Breaks if original is deleted. Can cross filesystems.
```bash
ln file.txt hardlink.txt        # hard link
ln -s file.txt symlink.txt      # symbolic link
ls -la symlink.txt              # shows -> file.txt
```

---

**Q12. What is the `/proc` filesystem?**
> `/proc` is a virtual filesystem that exposes kernel and process information as files:
```bash
cat /proc/cpuinfo       # CPU information
cat /proc/meminfo       # memory information
cat /proc/version       # kernel version
cat /proc/uptime        # system uptime
cat /proc/1234/status   # info about process 1234
cat /proc/net/tcp       # network connections
```

---

**Q13. What is swap space in Linux?**
> Swap is disk space used as "overflow" memory when RAM is full. The kernel moves inactive pages from RAM to swap.
```bash
swapon --show       # show swap usage
free -h             # show RAM and swap
mkswap /dev/sdb1    # create swap on partition
swapon /dev/sdb1    # enable swap
```
> In production: Avoid heavy swap usage — it's much slower than RAM. For containers: often disable swap.

---

**Q14. What is the difference between a process and a thread?**
> - **Process:** Independent program with its own memory space. Created with `fork()`.
> - **Thread:** Lightweight unit of execution within a process. Shares memory with other threads in same process.
> Multiple threads = faster (parallel) but share memory = need synchronization.

---

**Q15. What are Linux runlevels / targets?**
> Runlevels define the system state (old SysV init). Systemd uses "targets":
> | Runlevel | Systemd Target | Description |
> |---|---|---|
> | 0 | poweroff.target | Shutdown |
> | 1 | rescue.target | Single user/rescue mode |
> | 3 | multi-user.target | Multi-user, no GUI |
> | 5 | graphical.target | Multi-user with GUI |
> | 6 | reboot.target | Reboot |
```bash
systemctl get-default           # show current target
systemctl set-default multi-user.target
systemctl isolate rescue.target # switch target now
```

---

**Q16. What is `ulimit` in Linux?**
> ulimit controls resource limits for processes per user:
```bash
ulimit -a            # show all limits
ulimit -n 65536      # max open files (temporary)
ulimit -u 4096       # max user processes

# Permanent limits in /etc/security/limits.conf
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf
```
> Important for: High-traffic web servers, databases that need many open connections.

---

**Q17. What is a zombie process?**
> A zombie process has finished execution but still has an entry in the process table (parent hasn't read its exit status with `wait()`).
```bash
ps aux | grep Z     # find zombie processes (Z state)
```
> Zombies waste PID slots. Fix: Fix the parent process to call `wait()`, or kill the parent (init will adopt and reap).

---

**Q18. What is an orphan process?**
> An orphan process's parent has died. Linux automatically re-parents it to PID 1 (init/systemd), which eventually reaps it. Usually not a problem.

---

**Q19. What is the difference between `kill`, `kill -9`, and `kill -15`?**
> - `kill PID` or `kill -15 PID` — sends SIGTERM. Process can catch it and shutdown gracefully.
> - `kill -9 PID` — sends SIGKILL. Process is immediately terminated by kernel. Cannot be caught or ignored.
> - `kill -1 PID` — sends SIGHUP. Reload config (for daemons).
> Always try SIGTERM first, use SIGKILL as last resort.

---

**Q20. What is `cron` and how does it work?**
> cron is a job scheduler. Jobs defined in crontab files run at scheduled times.
```bash
crontab -e          # edit current user's crontab
crontab -l          # list crontab entries
crontab -r          # remove crontab

# Format: minute hour day month weekday command
# * * * * * /path/to/script.sh

0 2 * * *    /backup.sh          # every day at 2:00 AM
*/5 * * * *  /health-check.sh    # every 5 minutes
0 0 * * 0    /weekly-report.sh   # every Sunday midnight
0 9-17 * * 1-5 /business.sh     # 9AM-5PM, Mon-Fri
```

---

## 2. File System & Permissions

**Q21. Explain Linux file permissions.**
> Every file has permissions for: Owner (u), Group (g), Others (o)
> Each can have: Read (r=4), Write (w=2), Execute (x=1)
```bash
ls -la file.txt
# -rwxr-xr-- 1 alice devops 1234 Jan 1 10:00 file.txt
#  ^^^       owner=alice, group=devops
#  rwx = owner: read+write+execute
#  r-x = group: read+execute
#  r-- = others: read only
```

---

**Q22. How do you change file permissions?**
```bash
# Symbolic mode
chmod u+x script.sh        # add execute for owner
chmod g-w file.txt         # remove write for group
chmod o=r file.txt         # set others to read-only
chmod a+x script.sh        # add execute for all

# Octal mode (easier to remember)
chmod 755 script.sh        # rwxr-xr-x
chmod 644 file.txt         # rw-r--r--
chmod 600 private.key      # rw------- (private key!)
chmod 777 file.txt         # rwxrwxrwx (dangerous!)

# Common permissions
# 755 = owner:rwx group:rx others:rx (scripts, dirs)
# 644 = owner:rw group:r others:r (files)
# 600 = owner:rw (SSH private keys)
# 400 = owner:r (read-only keys)
```

---

**Q23. What is `chown` and `chgrp`?**
```bash
chown alice file.txt              # change owner
chown alice:devops file.txt       # change owner and group
chown -R alice:devops /app/       # recursive
chgrp devops file.txt             # change group only
```

---

**Q24. What are SUID, SGID, and Sticky Bit?**
> - **SUID (4000):** Execute file as the file's OWNER (e.g., `/usr/bin/passwd` runs as root)
> - **SGID (2000):** Execute as GROUP. On directory: new files inherit group.
> - **Sticky Bit (1000):** On directory: only file owner can delete their files (e.g., `/tmp`)
```bash
chmod u+s file      # set SUID
chmod g+s dir       # set SGID
chmod +t /tmp       # set sticky bit
ls -la /tmp         # shows drwxrwxrwt (t = sticky bit)
ls -la /usr/bin/passwd  # shows -rwsr-xr-x (s = SUID)
```

---

**Q25. What is `umask`?**
> umask sets default permissions for newly created files by subtracting from 666 (files) or 777 (directories).
```bash
umask            # show current umask (usually 022)
umask 022        # new files: 644 (666-022), dirs: 755 (777-022)
umask 027        # new files: 640, dirs: 750 (more restrictive)
```

---

**Q26. How do you find files in Linux?**
```bash
find / -name "*.log"                    # find by name
find /var/log -name "*.log" -mtime +7  # logs older than 7 days
find . -type f -size +100M             # files larger than 100MB
find . -type d -name "node_modules"    # find directories
find . -perm 777                        # find files with 777 permissions
find . -user alice                      # files owned by alice
find . -name "*.tmp" -delete           # find and delete
find . -name "*.py" -exec chmod 644 {} \;  # find and execute command
```

---

**Q27. What is `grep` and how do you use it?**
```bash
grep "error" app.log                   # search in file
grep -i "error" app.log               # case-insensitive
grep -r "TODO" /app/src/              # recursive search
grep -n "error" app.log               # show line numbers
grep -v "DEBUG" app.log               # invert (exclude DEBUG)
grep -c "error" app.log               # count matching lines
grep -A 3 "error" app.log            # 3 lines After match
grep -B 3 "error" app.log            # 3 lines Before match
grep -E "error|warning" app.log      # extended regex (OR)
grep -l "TODO" *.py                   # only filenames
grep "^[0-9]" file.txt               # lines starting with digit
ps aux | grep nginx                   # search process list
```

---

**Q28. Explain `sed` command.**
```bash
# Stream Editor - search, replace, transform text
sed 's/old/new/' file.txt           # replace first occurrence per line
sed 's/old/new/g' file.txt          # replace ALL occurrences
sed -i 's/old/new/g' file.txt       # in-place edit (modify file!)
sed -i.bak 's/old/new/g' file.txt   # in-place with backup

sed '3d' file.txt                    # delete line 3
sed '/pattern/d' file.txt           # delete lines matching pattern
sed -n '5,10p' file.txt             # print lines 5-10
sed 's/^/  /' file.txt             # add 2 spaces at start of each line

# Real-world use: update config
sed -i 's/MAX_CONNECTIONS=100/MAX_CONNECTIONS=500/' /etc/myapp.conf
```

---

**Q29. Explain `awk` command.**
```bash
# Pattern scanning and processing language
awk '{print $1}' file.txt           # print first column
awk '{print $1, $3}' file.txt       # print columns 1 and 3
awk -F: '{print $1}' /etc/passwd    # use : as delimiter, print usernames
awk '{sum += $1} END {print sum}' numbers.txt  # sum column

# Conditional
awk '$3 > 100 {print $0}' data.txt  # print lines where col 3 > 100
awk '/error/ {print NR, $0}' app.log  # line number + matching lines

# Process output
ps aux | awk '{print $2, $11}'      # PID and command
df -h | awk 'NR>1 {print $5, $6}'  # disk usage % and mount
```

---

**Q30. What are the important log files in Linux?**
```bash
/var/log/syslog         # General system logs (Ubuntu)
/var/log/messages       # General system logs (CentOS)
/var/log/auth.log       # Authentication logs (sudo, ssh)
/var/log/kern.log       # Kernel logs
/var/log/dmesg          # Boot and hardware messages
/var/log/nginx/         # Nginx access and error logs
/var/log/apache2/       # Apache logs
/var/log/mysql/         # MySQL logs
/var/log/cron.log       # Cron job logs

# View logs
tail -f /var/log/syslog              # follow real-time
tail -n 100 /var/log/auth.log        # last 100 lines
grep "Failed" /var/log/auth.log      # failed logins
journalctl -u nginx --since "1 hour ago"
```

---

**Q31. What is `tar` and how do you use it?**
```bash
# Create archive
tar -czf archive.tar.gz /path/to/dir/    # create gzip compressed tar
tar -cjf archive.tar.bz2 /path/        # create bzip2 compressed tar
tar -cf archive.tar /path/              # create uncompressed tar

# Extract archive
tar -xzf archive.tar.gz                 # extract gzip tar
tar -xzf archive.tar.gz -C /dest/      # extract to specific dir
tar -xf archive.tar                     # extract uncompressed

# List contents
tar -tzf archive.tar.gz                 # list files without extracting

# Flags: c=create, x=extract, z=gzip, j=bzip2, f=file, v=verbose, t=list
```

---

**Q32. How do you monitor disk usage?**
```bash
df -h                   # disk space per filesystem
df -i                   # inode usage
du -sh /var/log/        # size of directory
du -sh /*               # size of each top-level directory
du -ah /var/ | sort -rh | head -20  # top 20 largest files/dirs
lsblk                   # list block devices
fdisk -l                # list disk partitions
```

---

**Q33. How do you check and manage file descriptors?**
```bash
lsof -p 1234            # files opened by process 1234
lsof -u alice           # files opened by user alice
lsof -i :80             # processes using port 80
lsof /var/log/app.log   # processes using this file
cat /proc/sys/fs/file-max  # system-wide max open files
```

---

**Q34. What is `rsync` and how is it used?**
```bash
# Sync files efficiently (only changed files)
rsync -av src/ dest/                   # local sync
rsync -avz src/ user@remote:/dest/    # to remote (with compression)
rsync -avz --delete src/ dest/        # mirror (delete extra files in dest)
rsync -avz --exclude='*.log' src/ dest/  # exclude files
rsync --dry-run -av src/ dest/        # preview without making changes

# Common use: backup
rsync -avz --delete /data/ backup-server:/backups/data/
```

---

**Q35. What is `ln` and how is it different from `cp`?**
> - `cp` creates a complete duplicate (uses more disk space)
> - `ln` creates a hard link (same data, no extra space)
> - `ln -s` creates a symlink (just a pointer)
```bash
cp file.txt copy.txt        # duplicate - uses disk space
ln file.txt hardlink.txt    # hard link - same data, no extra space
ln -s /etc/nginx/ nginx     # symlink to directory
```

---

## 3. Process Management

**Q36. How do you view running processes?**
```bash
ps aux                      # all processes (BSD style)
ps -ef                      # all processes (UNIX style)
ps aux | grep nginx         # find specific process
top                         # interactive process viewer
htop                        # better interactive viewer
pgrep nginx                 # get PID of process by name
pidof nginx                 # get PID(s) of process
```

---

**Q37. Explain `top` command output.**
```
top - 14:23:05 up 10 days,  2:34,  3 users,  load average: 0.52, 0.58, 0.65
Tasks: 245 total,   1 running, 244 sleeping
%Cpu(s):  5.2 us,  1.3 sy,  0.0 ni, 93.2 id,  0.2 wa
MiB Mem :  15951.4 total,   1234.5 free,   8765.2 used,   5951.7 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.

  PID USER  PR  NI    VIRT    RES    SHR S  %CPU  %MEM  TIME+  COMMAND
 1234 nginx 20   0  123456  45678  12345 S   2.3   0.3  1:23.45 nginx
```
> - `load average`: 1/5/15 minute averages. > number of CPUs = overloaded
> - `us` = user CPU, `sy` = system CPU, `id` = idle, `wa` = waiting for I/O
> - `RES` = actual RAM used (Resident Set Size)

---

**Q38. How do you run a process in background?**
```bash
command &               # run in background
nohup command &         # run in background, immune to hangup (HUP)
nohup command > output.log 2>&1 &  # with log file

# Job control
jobs                    # list background jobs
fg %1                   # bring job 1 to foreground
bg %1                   # send to background
Ctrl+Z                  # suspend current process
Ctrl+C                  # terminate current process
```

---

**Q39. What is `screen` and `tmux`?**
> Both allow terminal multiplexing — run multiple terminals in one session, detach and reattach later.
```bash
# screen
screen              # start session
screen -S mysession # named session
Ctrl+A, D           # detach
screen -r mysession # reattach
screen -ls          # list sessions

# tmux (more modern)
tmux                # start session
tmux new -s mysession
Ctrl+B, D           # detach
tmux attach -t mysession
tmux ls             # list sessions
```
> Critical for DevOps: SSH into server, start long process in tmux, detach, log out safely.

---

**Q40. What is `strace`?**
```bash
strace -p 1234           # trace system calls of running process
strace command           # trace system calls of new command
strace -e trace=open ls  # only trace 'open' syscalls
strace -o output.txt ls  # save to file
```
> Used for debugging: See what files a process opens, what syscalls it makes. Very useful for troubleshooting "Permission denied" or missing file errors.

---

**Q41. What is `lsof`?**
```bash
lsof                     # all open files
lsof -i :8080           # what's using port 8080
lsof -u alice           # files opened by user alice
lsof -p 1234            # files opened by PID 1234
lsof +D /var/log/       # all files in directory
```
> "lsof" = List Open Files. In Linux, everything is a file — including network connections, pipes.

---

**Q42. How do you check memory usage?**
```bash
free -h                 # RAM and swap usage
cat /proc/meminfo       # detailed memory info
vmstat 1 5              # memory stats every 1 second, 5 times
top                     # interactive (press M to sort by memory)
ps aux --sort=-%mem | head  # top memory-using processes
```

---

**Q43. What is the `nice` and `renice` command?**
```bash
nice -n 19 command      # run with lowest priority (19 = least nice)
nice -n -20 command     # run with highest priority (needs root)
renice -n 10 -p 1234    # change priority of running process
renice -n 5 -u alice    # change priority of all alice's processes

# Priority range: -20 (highest) to 19 (lowest), default: 0
```

---

**Q44. What is `iotop` and `iostat`?**
```bash
iotop                   # I/O usage by process (like top for disk)
iostat                  # disk I/O statistics
iostat -x 1 5           # extended stats, every 1 sec, 5 times
iostat -d sda           # stats for specific disk

# Key metrics in iostat:
# %util = how busy the disk is (>80% = bottleneck)
# await = average I/O wait time (ms)
```

---

**Q45. What is a daemon process?**
> A daemon is a background service process that:
> - Has no controlling terminal
> - Usually starts at boot
> - Runs continuously
> - Name often ends in 'd': `nginx`, `sshd`, `crond`, `dockerd`
> Managed by systemd with `systemctl`.

---

**Q46. What is `pstree`?**
```bash
pstree              # show processes as tree
pstree -p           # show with PIDs
pstree alice        # tree for user alice's processes
```
> Shows parent-child relationships between processes. Useful for understanding process hierarchy.

---

**Q47. What is `/dev/null`?**
> `/dev/null` is a special file that discards everything written to it. Read from it = get empty output.
```bash
command > /dev/null        # discard stdout
command 2> /dev/null       # discard stderr
command > /dev/null 2>&1   # discard both stdout and stderr
```
> Used in cron jobs and scripts to suppress unwanted output.

---

**Q48. How do you check CPU information?**
```bash
cat /proc/cpuinfo           # detailed CPU info
nproc                       # number of processing units
lscpu                       # CPU architecture info
mpstat                      # per-CPU statistics
mpstat -P ALL 1 5           # all CPUs, every 1 sec, 5 times
top                         # press 1 to see per-CPU usage
```

---

## 4. Networking

**Q49. How do you check network configuration?**
```bash
ip addr show            # show IP addresses (modern)
ip addr show eth0       # specific interface
ifconfig                # older command (deprecated)
ip link show            # show interfaces
ip route show           # show routing table
route -n                # show routing table (older)
```

---

**Q50. How do you check open ports and connections?**
```bash
ss -tlnp                # TCP listening ports with process (modern)
ss -tulnp               # TCP + UDP listening ports
netstat -tlnp           # older (netstat is deprecated)
netstat -an             # all connections
lsof -i :80             # what's using port 80
lsof -i TCP             # all TCP connections
```

---

**Q51. How do you troubleshoot network connectivity?**
```bash
ping google.com             # test connectivity
ping -c 4 google.com        # 4 packets only
traceroute google.com       # trace route (hops)
tracepath google.com        # similar, no root needed
mtr google.com              # combines ping + traceroute (real-time)
nslookup google.com         # DNS lookup
dig google.com              # detailed DNS lookup
dig +short google.com       # just the IP
curl -v http://example.com  # test HTTP with details
telnet host 80              # test TCP port connectivity
nc -zv host 80              # test port (netcat)
```

---

**Q52. What is `netcat` (`nc`)?**
```bash
nc -zv hostname 80          # test if port 80 is open
nc -zv hostname 1-1024      # scan port range
nc -l 8080                  # listen on port 8080
echo "hello" | nc host 80   # send data to port

# File transfer
nc -l 9999 > file.txt       # receive file
nc hostname 9999 < file.txt  # send file
```

---

**Q53. How do you manage firewall rules with iptables?**
```bash
iptables -L                             # list all rules
iptables -L -n -v                       # detailed with packet counts
iptables -A INPUT -p tcp --dport 80 -j ACCEPT   # allow port 80
iptables -A INPUT -p tcp --dport 22 -j ACCEPT   # allow SSH
iptables -A INPUT -j DROP              # drop everything else

# Save rules
iptables-save > /etc/iptables/rules.v4

# Modern alternative: ufw (Uncomplicated Firewall)
ufw allow 80/tcp
ufw allow 22/tcp
ufw enable
ufw status
```

---

**Q54. How do you use `curl` for HTTP testing?**
```bash
curl http://example.com                  # GET request
curl -X POST http://api/endpoint         # POST request
curl -X POST -d '{"key":"value"}' \
     -H "Content-Type: application/json" \
     http://api/endpoint
curl -I http://example.com               # headers only
curl -v http://example.com               # verbose (show headers)
curl -o output.html http://example.com   # save to file
curl -u user:pass http://example.com     # basic auth
curl -k https://self-signed.example.com  # ignore SSL errors
curl -w "%{http_code}" http://example.com  # show status code
curl --max-time 10 http://example.com    # timeout after 10s
```

---

**Q55. What is SSH and how do you use it securely?**
```bash
ssh user@hostname                    # basic SSH
ssh -i ~/.ssh/key.pem user@host      # with private key
ssh -p 2222 user@host               # custom port
ssh -L 8080:localhost:80 user@host   # local port forwarding
ssh -R 8080:localhost:80 user@host   # remote port forwarding
ssh -J jumphost user@target          # jump through bastion host

# SSH config file (~/.ssh/config)
Host production
    HostName 10.0.1.100
    User ubuntu
    IdentityFile ~/.ssh/prod-key.pem
    Port 22

ssh production  # use alias
```

---

**Q56. What is SSH port forwarding / tunneling?**
> **Local Port Forwarding:** Access a remote service through a local port
```bash
ssh -L 5432:db-private:5432 bastion-host
# Now: psql -h localhost -p 5432  (connects to private DB via bastion)
```
> **Remote Port Forwarding:** Expose local service to remote server
```bash
ssh -R 8080:localhost:3000 remote-server
# Remote server's port 8080 → your local port 3000
```

---

**Q57. How do you configure SSH key-based authentication?**
```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "your@email.com"
ssh-keygen -t ed25519 -C "your@email.com"  # more secure

# Copy public key to server
ssh-copy-id user@server
# Or manually:
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# Fix permissions (required!)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_rsa        # private key
chmod 644 ~/.ssh/id_rsa.pub    # public key
```

---

**Q58. What is `/etc/hosts` file?**
> Local DNS resolution file. Maps hostnames to IPs without DNS query.
```bash
cat /etc/hosts
# 127.0.0.1   localhost
# 192.168.1.10 myserver
# 10.0.0.5    db-server db

echo "192.168.1.100 myapp.local" >> /etc/hosts
ping myapp.local  # resolves to 192.168.1.100
```
> Checked BEFORE DNS. Great for local development, overriding DNS in testing.

---

**Q59. What is `tcpdump`?**
```bash
tcpdump -i eth0                    # capture all traffic on eth0
tcpdump -i eth0 port 80           # capture port 80 traffic
tcpdump -i eth0 host 10.0.0.1    # traffic to/from specific host
tcpdump -i eth0 -w capture.pcap   # save to file (open in Wireshark)
tcpdump -i eth0 -n                # don't resolve hostnames
tcpdump -i eth0 'tcp and port 443 and host 10.0.0.1'
```
> Use for: Debugging network issues, capturing traffic for analysis.

---

**Q60. What is DNS and how does resolution work?**
> ```
> Browser asks: What is the IP for google.com?
>   → Check /etc/hosts (local)
>   → Check local DNS cache
>   → Ask configured DNS resolver (/etc/resolv.conf)
>   → Resolver asks Root DNS server
>   → Root says: ask .com nameserver
>   → .com says: ask google.com nameserver
>   → google.com nameserver returns IP: 142.250.x.x
>   → Browser connects to that IP
> ```
```bash
cat /etc/resolv.conf    # configured DNS servers
dig google.com          # DNS lookup with details
dig @8.8.8.8 google.com # use specific DNS server (Google DNS)
nslookup google.com     # simple DNS lookup
host google.com         # simple DNS lookup
```

---

**Q61. How do you set a static IP address in Linux?**
```bash
# Ubuntu/Debian - netplan (/etc/netplan/01-netcfg.yaml)
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

sudo netplan apply

# CentOS/RHEL - /etc/sysconfig/network-scripts/ifcfg-eth0
BOOTPROTO=static
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
```

---

**Q62. What is `ip route` and how do you add a route?**
```bash
ip route show                               # show routing table
ip route add 192.168.2.0/24 via 10.0.0.1  # add route
ip route del 192.168.2.0/24               # delete route
ip route add default via 192.168.1.1      # add default gateway

# Persistent routes: add to /etc/network/interfaces or netplan
```

---

## 5. Shell Scripting

**Q63. Write a basic Bash script with error handling.**
```bash
#!/bin/bash
set -euo pipefail
# -e: exit on error
# -u: error on undefined variable
# -o pipefail: pipe fails if any command fails

LOG_FILE="/var/log/myscript.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

cleanup() {
    log "Script interrupted, cleaning up..."
    # cleanup actions here
}
trap cleanup EXIT INT TERM

log "Script started"

if [ -z "${1:-}" ]; then
    log "ERROR: No argument provided"
    echo "Usage: $0 <environment>"
    exit 1
fi

ENVIRONMENT="$1"
log "Deploying to: $ENVIRONMENT"
```

---

**Q64. What are the common Bash conditionals?**
```bash
# File tests
if [ -f file.txt ]; then echo "file exists"; fi
if [ -d /tmp/mydir ]; then echo "directory exists"; fi
if [ -r file.txt ]; then echo "file is readable"; fi
if [ -x script.sh ]; then echo "file is executable"; fi
if [ -s file.txt ]; then echo "file is non-empty"; fi
if [ ! -f file.txt ]; then echo "file does NOT exist"; fi

# String tests
if [ -z "$var" ]; then echo "variable is empty"; fi
if [ -n "$var" ]; then echo "variable is not empty"; fi
if [ "$var" = "hello" ]; then echo "equal"; fi
if [ "$var" != "hello" ]; then echo "not equal"; fi

# Number tests
if [ "$num" -eq 5 ]; then echo "equal to 5"; fi
if [ "$num" -gt 5 ]; then echo "greater than 5"; fi
if [ "$num" -lt 5 ]; then echo "less than 5"; fi
if [ "$num" -ge 5 ]; then echo "greater than or equal"; fi

# Combined
if [ -f file.txt ] && [ -r file.txt ]; then echo "exists and readable"; fi
if [ "$a" = "x" ] || [ "$b" = "y" ]; then echo "a=x or b=y"; fi
```

---

**Q65. How do you write loops in Bash?**
```bash
# For loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

for file in *.log; do
    echo "Processing: $file"
    gzip "$file"
done

for i in {1..10}; do echo $i; done

for ((i=0; i<5; i++)); do echo $i; done

# While loop
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < input.txt

# Until loop (opposite of while)
until [ -f ready.flag ]; do
    echo "Waiting..."
    sleep 5
done
```

---

**Q66. What are special variables in Bash?**
```bash
$0          # script name
$1, $2...$9 # arguments
$@          # all arguments as separate words
$*          # all arguments as one word
$#          # number of arguments
$?          # exit code of last command (0=success)
$$          # current process PID
$!          # PID of last background process
$LINENO     # current line number

# Example
./script.sh arg1 arg2
# $0 = ./script.sh
# $1 = arg1, $2 = arg2
# $# = 2
```

---

**Q67. How do you handle errors in shell scripts?**
```bash
#!/bin/bash
set -e          # Exit on any error

# Check command success
if ! command -v docker &>/dev/null; then
    echo "Docker not installed"
    exit 1
fi

# Trap errors
trap 'echo "Error on line $LINENO"; exit 1' ERR

# Check exit code
tar -czf backup.tar.gz /data
if [ $? -ne 0 ]; then
    echo "Backup failed!"
    exit 1
fi

# Or using ||
mkdir /tmp/mydir || { echo "mkdir failed"; exit 1; }
```

---

**Q68. What are arrays in Bash?**
```bash
# Declare array
fruits=("apple" "banana" "cherry")
servers=("web-01" "web-02" "db-01")

# Access elements
echo ${fruits[0]}        # apple
echo ${fruits[@]}        # all elements
echo ${#fruits[@]}       # length = 3

# Loop array
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Add element
fruits+=("mango")

# Associative array (dictionary)
declare -A config
config[host]="localhost"
config[port]="5432"
echo ${config[host]}
```

---

**Q69. How do you do string manipulation in Bash?**
```bash
str="Hello, World!"

echo ${#str}                    # length = 13
echo ${str:7:5}                 # substring "World"
echo ${str/World/Linux}         # replace first
echo ${str//l/L}                # replace all
echo ${str^^}                   # uppercase
echo ${str,,}                   # lowercase

# Remove prefix/suffix
filename="myfile.tar.gz"
echo ${filename%.*}             # myfile.tar (remove last extension)
echo ${filename%%.*}            # myfile (remove all extensions)
echo ${filename#*.}             # tar.gz (remove first prefix)
echo ${filename##*.}            # gz (remove all prefix)

# Default value
echo ${var:-"default"}          # use "default" if var is unset
echo ${var:="default"}          # assign "default" if var is unset
```

---

**Q70. Write a script to check if a service is running and restart if not.**
```bash
#!/bin/bash
set -euo pipefail

SERVICE="nginx"
LOG="/var/log/service-monitor.log"

check_and_restart() {
    local service=$1
    if ! systemctl is-active --quiet "$service"; then
        echo "[$(date)] $service is down, restarting..." >> "$LOG"
        systemctl restart "$service"
        if systemctl is-active --quiet "$service"; then
            echo "[$(date)] $service restarted successfully" >> "$LOG"
        else
            echo "[$(date)] FAILED to restart $service!" >> "$LOG"
            # Send alert (email, Slack, etc.)
        fi
    else
        echo "[$(date)] $service is running OK" >> "$LOG"
    fi
}

check_and_restart "$SERVICE"
```

---

**Q71. Write a script for log rotation.**
```bash
#!/bin/bash
LOG_DIR="/var/log/myapp"
MAX_DAYS=30
MAX_SIZE="100M"

# Delete logs older than 30 days
find "$LOG_DIR" -name "*.log" -mtime +$MAX_DAYS -delete

# Compress logs older than 7 days
find "$LOG_DIR" -name "*.log" -mtime +7 -exec gzip {} \;

# Report
echo "Log rotation complete. Current disk usage:"
du -sh "$LOG_DIR"
```

---

**Q72. What is process substitution in Bash?**
```bash
# Compare output of two commands
diff <(ls dir1/) <(ls dir2/)

# Read from command output as file
while read -r line; do
    echo "Server: $line"
done < <(aws ec2 describe-instances --query 'Reservations[].Instances[].PublicIpAddress' --output text)
```

---

**Q73. What is `xargs`?**
```bash
# Build command from stdin
echo "file1 file2 file3" | xargs rm
find . -name "*.log" | xargs rm
find . -name "*.txt" | xargs -I {} cp {} /backup/

# Parallel execution
cat servers.txt | xargs -P 5 -I {} ssh {} "sudo apt update"
# -P 5 = run 5 in parallel
# -I {} = replace {} with each line
```

---

**Q74. What are here documents (heredoc)?**
```bash
# Write multi-line text to file
cat > /etc/myapp/config.conf << 'EOF'
server {
    listen 80;
    server_name myapp.com;
    root /var/www/html;
}
EOF

# Or with variable expansion
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  APP_ENV: ${ENVIRONMENT}
EOF
```

---

**Q75. Write a script to deploy an application.**
```bash
#!/bin/bash
set -euo pipefail

APP_NAME="myapp"
DEPLOY_DIR="/var/www/${APP_NAME}"
BACKUP_DIR="/var/backups/${APP_NAME}"
GIT_REPO="https://github.com/org/myapp.git"
BRANCH="${1:-main}"

log() { echo "[$(date '+%H:%M:%S')] $1"; }

# Create backup
log "Creating backup..."
mkdir -p "$BACKUP_DIR"
[ -d "$DEPLOY_DIR" ] && \
    tar -czf "${BACKUP_DIR}/backup-$(date +%Y%m%d-%H%M%S).tar.gz" "$DEPLOY_DIR"

# Deploy
log "Deploying branch: $BRANCH"
if [ -d "$DEPLOY_DIR/.git" ]; then
    cd "$DEPLOY_DIR"
    git fetch origin
    git checkout "$BRANCH"
    git pull origin "$BRANCH"
else
    git clone -b "$BRANCH" "$GIT_REPO" "$DEPLOY_DIR"
    cd "$DEPLOY_DIR"
fi

# Install dependencies
log "Installing dependencies..."
npm install --production

# Restart service
log "Restarting service..."
systemctl restart "$APP_NAME"
sleep 3

if systemctl is-active --quiet "$APP_NAME"; then
    log "Deployment successful!"
else
    log "Deployment failed! Rolling back..."
    # restore from backup
    exit 1
fi
```

---

## 6. System Administration

**Q76. How do you manage users and groups?**
```bash
# Users
useradd -m -s /bin/bash alice         # create user with home dir
useradd -m -G sudo alice              # add to sudo group at creation
passwd alice                           # set password
usermod -aG docker alice              # add to docker group
usermod -s /bin/bash alice            # change shell
userdel -r alice                      # delete user and home dir

# Groups
groupadd devops                        # create group
groupdel devops                        # delete group
gpasswd -a alice devops               # add user to group
gpasswd -d alice devops               # remove user from group

# View
id alice                               # show user's UID, GID, groups
cat /etc/passwd                        # all users
cat /etc/group                         # all groups
who                                    # logged-in users
w                                      # logged-in users with activity
last                                   # login history
```

---

**Q77. How do you manage packages in Ubuntu/Debian?**
```bash
apt update                            # update package list
apt upgrade                           # upgrade all packages
apt install nginx                     # install package
apt remove nginx                      # remove package
apt purge nginx                       # remove + config files
apt autoremove                        # remove unused dependencies
apt search nginx                      # search packages
apt show nginx                        # package info
dpkg -l | grep nginx                  # list installed packages
dpkg -l nginx                         # check if package installed

# Non-interactive install (for scripts)
DEBIAN_FRONTEND=noninteractive apt-get install -y nginx
```

---

**Q78. How do you manage packages in CentOS/RHEL?**
```bash
yum update                            # update all
yum install nginx                     # install
yum remove nginx                      # remove
yum search nginx                      # search
yum info nginx                        # info
yum list installed                    # list installed
rpm -qa | grep nginx                  # query installed RPMs

# Modern: dnf (RHEL 8+)
dnf update
dnf install nginx
dnf module list                       # list modules (AppStreams)
```

---

**Q79. How do you monitor system resources?**
```bash
# CPU
top / htop
mpstat 1              # per-CPU stats
sar -u 1 5           # CPU utilization (sar from sysstat)

# Memory
free -h
vmstat 1 5
sar -r 1 5

# Disk I/O
iotop
iostat -x 1 5
sar -d 1 5

# Network
iftop                 # network bandwidth by connection
nethogs               # network usage by process
sar -n DEV 1 5        # network device stats

# All-in-one
dstat                 # combines cpu, disk, net, memory
glances               # Python-based system monitor
```

---

**Q80. What is `sysctl` and how is it used?**
```bash
sysctl -a                              # list all kernel parameters
sysctl net.ipv4.ip_forward             # check value
sysctl -w net.ipv4.ip_forward=1       # set temporarily

# Permanent (in /etc/sysctl.conf or /etc/sysctl.d/)
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p                              # apply changes

# Common tuning parameters
net.core.somaxconn = 65535           # max connection queue
net.ipv4.tcp_max_syn_backlog = 65535
vm.swappiness = 10                   # reduce swap usage
fs.file-max = 2097152                # max open files
```

---

**Q81. How do you set up a cron job to rotate logs daily?**
```bash
# Edit crontab
crontab -e

# Add:
0 0 * * * /usr/local/bin/log-rotate.sh >> /var/log/log-rotate.log 2>&1

# Or use /etc/cron.daily/ - scripts here run daily
cp log-rotate.sh /etc/cron.daily/
chmod +x /etc/cron.daily/log-rotate.sh
```

---

**Q82. What is `journalctl` and how do you use it?**
```bash
journalctl                             # all logs
journalctl -f                          # follow (like tail -f)
journalctl -u nginx                    # service logs
journalctl -u nginx -f                 # follow service logs
journalctl --since "1 hour ago"        # recent logs
journalctl --since "2024-01-01" --until "2024-01-02"
journalctl -p err                      # error level and above
journalctl -p err -u docker            # errors for docker service
journalctl -n 100                      # last 100 lines
journalctl --disk-usage                # how much disk logs use
journalctl --vacuum-size=1G            # keep only 1GB of logs
```

---

**Q83. What is `/etc/fstab`?**
> File Systems Table — defines how filesystems are mounted at boot.
```bash
cat /etc/fstab
# device          mountpoint  type    options     dump  pass
# /dev/sda1       /           ext4    defaults    0     1
# /dev/sdb1       /data       ext4    defaults    0     2
# tmpfs           /tmp        tmpfs   defaults    0     0

# Mount NFS
# server:/share   /mnt/share  nfs     defaults    0     0

# Test fstab entry without rebooting
mount -a    # mount all fstab entries

# Add S3-mounted storage (s3fs)
# my-bucket /mnt/s3 fuse.s3fs defaults,_netdev 0 0
```

---

**Q84. How do you manage system time and timezone?**
```bash
date                               # show current date/time
timedatectl                        # show time settings
timedatectl list-timezones         # list available timezones
timedatectl set-timezone Asia/Kolkata  # set timezone
timedatectl set-ntp true           # enable NTP sync

# NTP synchronization
systemctl status chronyd           # CentOS NTP service
systemctl status systemd-timesyncd # Ubuntu NTP service

chronyc tracking                   # NTP sync status
```

---

**Q85. How do you manage systemd services (write a unit file)?**
```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target
Requires=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node /opt/myapp/server.js
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
Environment=NODE_ENV=production
Environment=PORT=3000
EnvironmentFile=/opt/myapp/.env

[Install]
WantedBy=multi-user.target
```
```bash
systemctl daemon-reload            # reload after creating/editing unit file
systemctl enable myapp             # enable at boot
systemctl start myapp              # start now
```

---

## 7. Advanced & Real-World

**Q86. How do you troubleshoot "disk full" on a server?**
```bash
# Find what's using space
df -h                              # disk usage overview
du -sh /*                          # top-level directories
du -ah /var/ | sort -rh | head -20 # largest files in /var

# Common culprits
ls -lh /var/log/                   # log files
docker system df                   # Docker using space
journalctl --disk-usage            # systemd journal
du -sh /var/lib/docker/            # Docker layers
find / -name "*.log" -size +1G    # large log files

# Quick fixes
docker system prune -a             # clean Docker
journalctl --vacuum-size=500M      # limit journal size
find /var/log -name "*.log.gz" -mtime +30 -delete  # old compressed logs
```

---

**Q87. How do you troubleshoot "too many open files" error?**
```bash
# Check current limits
ulimit -n                          # current user's limit
cat /proc/sys/fs/file-max         # system-wide limit

# Find who's using too many
lsof | wc -l                      # total open files
lsof -u nginx | wc -l             # nginx open files
cat /proc/$(pgrep nginx | head -1)/limits  # nginx process limits

# Fix temporarily
ulimit -n 65536

# Fix permanently for a service
# In systemd unit file:
[Service]
LimitNOFILE=65536

# System-wide in /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
```

---

**Q88. How do you check if a port is open on a remote server?**
```bash
# From your machine
nc -zv remote-host 80             # netcat
telnet remote-host 80             # telnet
curl -v telnet://remote-host:80   # curl
nmap -p 80 remote-host            # nmap (if installed)

# Check if port is listening on the server itself
ss -tlnp | grep :80
netstat -tlnp | grep :80
```

---

**Q89. How do you analyse Linux performance issues?**
> Use the USE Method: **Utilization, Saturation, Errors** for each resource:
```bash
# CPU
mpstat 1                   # CPU utilization
sar -u 1 5                # historical CPU data
top (press 1)              # per-CPU breakdown

# Memory
free -h                    # RAM/swap usage
vmstat 1 5                # memory stats + swap in/out
sar -r 1 5

# Disk
iostat -x 1 5             # %util (busy?), await (slow?)
iotop                      # what process doing I/O

# Network
sar -n DEV 1 5            # bytes/packets per interface
nethogs                    # bandwidth per process
ss -s                      # socket statistics summary
```

---

**Q90. What is SELinux and AppArmor?**
> Both are Mandatory Access Control (MAC) systems — extra security layer beyond standard permissions.
> - **SELinux** (CentOS/RHEL): Labeling-based. Complex but powerful.
>   ```bash
>   getenforce              # Enforcing/Permissive/Disabled
>   setenforce 0            # set Permissive temporarily
>   sestatus                # SELinux status
>   audit2allow             # generate policy from denials
>   ```
> - **AppArmor** (Ubuntu): Profile-based. Simpler.
>   ```bash
>   aa-status               # show profile status
>   aa-complain nginx       # set to complain mode
>   aa-enforce nginx        # set to enforce mode
>   ```

---

**Q91. How do you set up passwordless sudo for a user?**
```bash
# Add to /etc/sudoers (use visudo - validates syntax)
visudo

# Add line:
alice ALL=(ALL) NOPASSWD: ALL   # all commands, no password

# Or specific commands only
alice ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx, /usr/bin/docker

# Better: create /etc/sudoers.d/alice
echo "alice ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/alice
chmod 440 /etc/sudoers.d/alice
```

---

**Q92. How do you secure SSH configuration?**
```bash
# /etc/ssh/sshd_config best practices
PermitRootLogin no              # never SSH as root
PasswordAuthentication no       # only key-based auth
PubkeyAuthentication yes
MaxAuthTries 3                  # limit auth attempts
AllowUsers alice bob            # whitelist users
Port 2222                       # change default port
ClientAliveInterval 300         # disconnect idle sessions
ClientAliveCountMax 2

# After changes
systemctl restart sshd
```

---

**Q93. What is `strace` vs `ltrace`?**
> - `strace` — traces **system calls** (kernel-level calls): file open, network, fork, exec
> - `ltrace` — traces **library calls** (user-level): libc functions like malloc, fopen, printf
```bash
strace ls /tmp              # see syscalls made by ls
ltrace ls /tmp              # see library calls made by ls
strace -p 1234              # attach to running process
```

---

**Q94. How do you check and fix filesystem errors?**
```bash
fsck /dev/sdb1              # check filesystem (unmount first!)
fsck -y /dev/sdb1           # auto-fix errors

e2fsck /dev/sdb1            # for ext2/3/4 filesystems
xfs_repair /dev/sdb1        # for XFS

# Check without repair
fsck -n /dev/sdb1

# Force fsck on next boot
touch /forcefsck
shutdown -r now
```

---

**Q95. How do you mount an NFS share?**
```bash
# Install NFS client
apt install nfs-common

# Mount temporarily
mount -t nfs server:/share /mnt/nfs

# Mount permanently (/etc/fstab)
server:/share   /mnt/nfs    nfs    defaults,_netdev    0   0

# Mount options
mount -t nfs -o ro server:/share /mnt/nfs    # read-only
mount -t nfs -o noatime server:/share /mnt   # no access time (performance)
```

---

**Q96. What is `sar` command?**
```bash
sar -u 1 5          # CPU: 1 second interval, 5 times
sar -r 1 5          # Memory usage
sar -d 1 5          # Disk I/O
sar -n DEV 1 5      # Network
sar -q 1 5          # Load average + run queue
sar -b 1 5          # I/O statistics

# Historical data
sar -u              # today's CPU usage
sar -u -f /var/log/sysstat/sa01  # specific date file
```
> `sar` from sysstat package. Collects and stores performance data. Great for post-mortem analysis.

---

**Q97. How do you use `tmux` for DevOps work?**
```bash
tmux new -s deploy          # new session named 'deploy'
Ctrl+B, c                   # new window
Ctrl+B, n / p               # next/previous window
Ctrl+B, "                   # split horizontal
Ctrl+B, %                   # split vertical
Ctrl+B, arrow keys          # move between panes
Ctrl+B, z                   # zoom pane (toggle full screen)
Ctrl+B, d                   # detach (leave running)
tmux attach -t deploy       # reattach

# Useful for: long deployments, monitoring multiple servers
# Real usage: tmux with 3 panes: logs | app | ssh
```

---

**Q98. How do you find and kill a process using a specific port?**
```bash
# Find PID using port
lsof -i :8080 -t             # just the PID
ss -tlnp | grep :8080
fuser 8080/tcp               # show PID

# Kill it
kill $(lsof -i :8080 -t)
fuser -k 8080/tcp            # kill using fuser directly
```

---

**Q99. What is the Linux load average?**
```bash
uptime
# 14:23:05 up 10 days, load average: 1.52, 1.35, 1.21
#                                     1min  5min  15min
```
> Load average = average number of processes wanting CPU (running + waiting).
> - Load = number of CPUs → 100% utilized (not bad)
> - Load > number of CPUs → overloaded
> - 4 CPU machine: load of 4.0 = 100% busy. Load of 8.0 = 200% busy (processes waiting).

---

**Q100. Real-world scenario: Server is slow, what do you check?**
```bash
# Step 1: Overview
uptime                  # load average - is it high?
top                     # CPU, memory, load overview
free -h                 # memory - is it low?

# Step 2: CPU
top → press P           # sort by CPU usage
ps aux --sort=-%cpu | head  # top CPU processes
mpstat 1 5              # CPU breakdown (user/system/iowait)

# Step 3: Memory
free -h                 # low free memory? high swap?
top → press M           # sort by memory
vmstat 1 5              # si/so = swap in/out (should be 0)

# Step 4: Disk I/O
iostat -x 1 5           # %util > 80% = bottleneck
iotop                   # which process doing I/O

# Step 5: Network
ss -s                   # socket stats
nethogs                 # bandwidth by process
netstat -an | grep ESTABLISHED | wc -l  # connection count

# Step 6: Application logs
journalctl -u myapp -n 100
tail -f /var/log/myapp/error.log
```

---

**Q101–Q110: Quick-fire questions.**

**Q101. What is `/etc/profile` vs `~/.bashrc` vs `~/.bash_profile`?**
> - `/etc/profile` — system-wide, runs for all users at LOGIN shell
> - `~/.bash_profile` or `~/.profile` — user-specific, runs at LOGIN shell
> - `~/.bashrc` — user-specific, runs for every INTERACTIVE non-login shell (new terminal tab)
> Typically: `~/.bash_profile` sources `~/.bashrc`

**Q102. What is the difference between `>` and `>>`?**
> - `>` — Redirect stdout, OVERWRITE file
> - `>>` — Redirect stdout, APPEND to file
> - `2>` — Redirect stderr
> - `2>&1` — Redirect stderr to stdout
> - `&>` — Redirect both stdout and stderr

**Q103. What does `chmod +x` do?**
> Adds execute permission to a file (for owner, group, and others). Required to run a script: `chmod +x script.sh && ./script.sh`

**Q104. What is `which` vs `whereis` vs `type`?**
> - `which python` — shows path of executable in PATH
> - `whereis python` — shows binary, source, and man page locations
> - `type python` — shows if it's alias, builtin, or external command

**Q105. What is `env` command?**
> `env` prints all environment variables. `env VAR=value command` runs command with extra env var. `env -i command` runs with clean environment.

**Q106. What is the difference between `cut` and `awk`?**
> - `cut` — simple field extraction by delimiter or character position
> - `awk` — full programming language for complex text processing
> `cut -d: -f1 /etc/passwd` vs `awk -F: '{print $1}' /etc/passwd` — both print usernames

**Q107. What is `watch` command?**
> Runs a command repeatedly and shows output: `watch -n 2 df -h` — shows disk usage every 2 seconds. `watch -d` highlights changes.

**Q108. How do you check the last boot time?**
> `who -b` or `uptime -s` or `last reboot | head -1`

**Q109. What is `dd` command and when is it used?**
> `dd` copies data at block level:
> - Create bootable USB: `dd if=ubuntu.iso of=/dev/sdb bs=4M`
> - Backup disk: `dd if=/dev/sda of=/backup/disk.img`
> - Test disk speed: `dd if=/dev/zero of=/tmp/test bs=1G count=1`

**Q110. What is the difference between `/dev/sda` and `/dev/sda1`?**
> - `/dev/sda` — the entire disk
> - `/dev/sda1` — the first partition on that disk
> You format and mount partitions (`/dev/sda1`), not entire disks (usually).

---
---


