# Linux Interview Crash Course — Fresher-Friendly Detailed Edition

> A practical Linux interview guide for beginners. Every major command is explained using **what it does**, **why you need it**, **syntax/example**, and **what an interviewer may ask**.

## How to read this guide

Do not try to memorize Linux as a list of commands. Think in troubleshooting questions:

```text
Where am I?             -> pwd
What files are here?    -> ls
What's inside this file?-> cat / less
Where is the file?      -> find
Where is the error?     -> grep
Who am I?               -> whoami / id
Can I access this?      -> ls -l / chmod / chown
Is the process running? -> ps / pgrep / top
Is the service running? -> systemctl
What went wrong?        -> journalctl / logs
Is disk full?           -> df / du
Is memory exhausted?    -> free / top
Is the port listening?  -> ss
Can I reach the app?    -> curl
Can DNS resolve it?     -> nslookup / dig
```

---

## Beginner Command Dictionary

### `pwd` — Print Working Directory

**What it does:** Shows the absolute path of the directory you are currently working in.

**Why needed:** Linux terminals do not always make your current location obvious. Before copying, deleting, or creating files, `pwd` helps verify where you are.

```bash
pwd
```

Example:

```text
/home/manideep/docker-project
```

**PowerShell equivalent:** `Get-Location`

---

### `ls` — List Directory Contents

**What it does:** Shows files and directories.

**Why needed:** One of the most frequently used Linux commands. Use it to inspect a directory before working with its contents.

```bash
ls
ls -l
ls -a
ls -lh
ls -lah
```

Important flags:

```text
-l = long listing including permissions/owner/size
-a = include hidden files
-h = human-readable sizes such as KB/MB/GB
```

Example:

```bash
ls -lah /var/log
```

**PowerShell equivalent:** `Get-ChildItem`

---

### `cd` — Change Directory

**What it does:** Changes your current working directory.

**Why needed:** Used to navigate the Linux filesystem.

```bash
cd /var/log
cd ..
cd ~
cd -
```

```text
.. = parent directory
~  = your home directory
-  = previous working directory
```

**PowerShell equivalent:** `Set-Location`

---

### `mkdir` — Make Directory

**What it does:** Creates directories.

**Why needed:** Applications, scripts, logs, configuration and deployment artifacts usually need directory structures.

```bash
mkdir project
mkdir -p project/config/dev
```

`-p` creates missing parent directories.

---

### `touch` — Create/Update a File

**What it does:** Creates an empty file if it does not exist, or updates timestamps if it does.

**Why needed:** Useful for quickly creating files during scripting/testing.

```bash
touch application.log
```

---

### `cp` — Copy

**What it does:** Copies files or directories.

**Why needed:** Used for backups, configuration deployment, application files, etc.

```bash
cp config.conf config.conf.backup
cp -r website/ website-backup/
```

`-r` means recursive and is required when copying directories.

**PowerShell equivalent:** `Copy-Item`

---

### `mv` — Move or Rename

**What it does:** Moves a file/directory or renames it.

**Why needed:** Linux uses the same command for both operations.

```bash
mv old.txt new.txt
mv app.log /tmp/
```

**PowerShell equivalent:** `Move-Item`

---

### `rm` — Remove

**What it does:** Deletes files/directories.

**Why needed:** Cleanup and administration.

```bash
rm file.txt
rm -r old-directory/
rm -rf old-directory/
```

```text
-r = recursively remove directory contents
-f = force/no confirmation for missing/unwritable operands
```

**Interview warning:** `rm -rf` can be destructive. Never use it casually with `sudo`, wildcards, or uncertain paths.

**PowerShell equivalent:** `Remove-Item`

---

### `cat` — Display File Contents

**What it does:** Writes file contents to the terminal.

**Why needed:** Quick inspection of small config files, scripts and logs.

```bash
cat /etc/os-release
```

For large files, prefer `less`.

**PowerShell equivalent:** `Get-Content`

---

### `less` — Read Large Files Interactively

**What it does:** Opens text one screen at a time.

**Why needed:** Safer and easier than dumping a huge log into the terminal.

```bash
less /var/log/syslog
```

Useful keys:

```text
/ERROR = search forward for ERROR
n      = next match
q      = quit
```

---

### `head` — First Lines

**What it does:** Displays the beginning of a file.

**Why needed:** Useful for headers, CSVs and quickly inspecting file format.

```bash
head application.log
head -n 20 application.log
```

---

### `tail` — Last Lines / Live Logs

**What it does:** Shows the end of a file.

**Why needed:** New application log entries are normally appended at the end, making `tail` extremely useful in operations.

```bash
tail -n 100 application.log
```

Follow new lines continuously:

```bash
tail -f application.log
```

**Common interview question:** How do you monitor an application log in real time?

```bash
tail -f application.log
```

---

### `grep` — Search Text

**What it does:** Finds lines matching a pattern.

**Why needed:** Logs can contain thousands of lines. `grep` helps isolate errors, warnings, IDs, endpoints and messages.

```bash
grep "ERROR" application.log
grep -i "error" application.log
grep -n "ERROR" application.log
grep -r "database" /etc/myapp/
```

```text
-i = case-insensitive
-n = show line numbers
-r = recursively search directories
-v = show lines that do NOT match
-c = count matching lines
```

Example:

```bash
grep -in "connection refused" application.log
```

**PowerShell equivalent:** `Select-String`

---

### `find` — Find Files/Directories

**What it does:** Searches the filesystem using criteria such as name, type, size and modification time.

**Why needed:** You often know a filename but not where it is located.

```bash
find /var -name "application.log"
find /var/log -type f -name "*.log"
find /var -type f -size +100M
```

```text
-type f = regular files
-type d = directories
-name   = case-sensitive name
-iname  = case-insensitive name
-size   = filter by size
```

Interview distinction:

```text
find -> finds filesystem objects
grep -> searches text/content
```

---

### `|` — Pipe

**What it does:** Sends stdout from one command into another command's stdin.

**Why needed:** Linux commands are designed to be combined.

```bash
ps aux | grep nginx
```

Concept:

```text
ps aux
   |
   v
grep nginx
```

The second command filters the first command's output.

---

### `>` and `>>` — Output Redirection

**What they do:** Send command output to a file.

**Why needed:** Useful for reports, logs and scripts.

Overwrite:

```bash
echo "Deployment started" > deployment.log
```

Append:

```bash
echo "Deployment completed" >> deployment.log
```

Errors use file descriptor `2`:

```bash
command 2> errors.log
```

Both stdout and stderr:

```bash
command > output.log 2>&1
```

---

### `chmod` — Change Permissions

**What it does:** Changes read/write/execute permissions.

**Why needed:** Linux controls whether users/groups can read, modify or execute files.

```bash
chmod +x deploy.sh
chmod 755 deploy.sh
chmod 644 config.conf
```

```text
r = read    = 4
w = write   = 2
x = execute = 1
```

Therefore:

```text
7 = rwx
6 = rw-
5 = r-x
4 = r--
```

`755`:

```text
Owner  = rwx = 7
Group  = r-x = 5
Others = r-x = 5
```

---

### `chown` — Change Owner

**What it does:** Changes file/directory ownership.

**Why needed:** Applications often run under dedicated service accounts and need correct ownership.

```bash
sudo chown appuser app.conf
sudo chown appuser:appgroup app.conf
sudo chown -R appuser:appgroup /opt/myapp
```

Interview:

```text
chmod = permissions
chown = ownership
```

---

### `whoami` and `id`

`whoami` shows the effective username:

```bash
whoami
```

`id` gives more detail:

```bash
id
```

Example:

```text
uid=1000(manideep) gid=1000(manideep) groups=1000(manideep),998(docker)
```

**Why needed:** Very useful when troubleshooting permission problems.

---

### `sudo` — Run with Elevated Privileges

**What it does:** Runs an authorized command as another user, normally root.

**Why needed:** Normal users should not have unrestricted administrative access.

```bash
sudo systemctl restart nginx
```

`root` is Linux's superuser and normally has UID `0`.

---

### `ps` — Process Snapshot

**What it does:** Displays running processes.

**Why needed:** Used to determine whether an application is running and find its PID.

```bash
ps aux
ps -ef
ps aux | grep nginx
```

A **PID** is a Process ID.

---

### `pgrep` — Find PID by Process Name

**What it does:** Searches running processes by name.

**Why needed:** Cleaner than parsing `ps` when you simply need a PID.

```bash
pgrep nginx
pgrep -a nginx
```

---

### `top` — Live Process Monitor

**What it does:** Shows processes, CPU, memory, load and system activity in real time.

**Why needed:** One of the first tools to use when a Linux server becomes slow.

```bash
top
```

Important keys:

```text
P = sort by CPU
M = sort by memory
1 = per-CPU display
k = send signal to process
q = quit
```

Interview scenario:

> Server CPU is high. What do you do?

Start with:

```bash
top
```

Identify the process consuming CPU, then investigate the application/service and its logs rather than immediately killing it.

---

### `kill` — Send a Signal to a Process

**What it does:** Sends a signal to a PID.

**Why needed:** Used to request process shutdown/reload or, when necessary, force termination.

Normal termination:

```bash
kill 1234
```

Equivalent:

```bash
kill -15 1234
```

Force:

```bash
kill -9 1234
```

```text
15 = SIGTERM -> asks process to terminate gracefully
9  = SIGKILL -> kernel immediately terminates it
```

**Interview answer:** Try SIGTERM first. Use SIGKILL only when necessary because the process cannot perform graceful cleanup.

---

### `systemctl` — Manage systemd Services

**What it does:** Controls services managed by systemd.

**Why needed:** Web servers, SSH, Docker, databases and application services commonly run as services.

```bash
systemctl status nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl enable nginx
sudo systemctl disable nginx
```

Important distinction:

```text
start  = start now
enable = configure startup at boot
```

Do both:

```bash
sudo systemctl enable --now nginx
```

Typical service names:

```text
nginx
apache2 / httpd
sshd
docker
cron / crond
mysql / mysqld
postgresql
```

---

### `journalctl` — Read systemd Journal Logs

**What it does:** Reads logs collected by systemd-journald.

**Why needed:** If `systemctl status` says a service failed, logs usually explain why.

```bash
journalctl -u nginx
journalctl -u nginx -n 100
journalctl -u nginx -f
journalctl -b
```

Troubleshooting flow:

```text
systemctl status nginx
          |
          v
journalctl -u nginx
```

---

### `df` — Filesystem Free Space

**What it does:** Shows filesystem capacity and usage.

**Why needed:** Full disks are a very common production problem.

```bash
df -h
```

`-h` makes sizes human-readable.

Check inode exhaustion:

```bash
df -i
```

---

### `du` — Directory/File Disk Usage

**What it does:** Calculates space consumed by files/directories.

**Why needed:** `df` tells you a filesystem is full; `du` helps identify what is consuming the space.

```bash
du -sh /var/log
du -sh /var/log/*
```

Concept:

```text
df -> Which filesystem is full?
du -> Which directory/file is consuming it?
```

---

### `free` — Memory Usage

**What it does:** Shows RAM and swap usage.

**Why needed:** Useful when troubleshooting memory pressure.

```bash
free -h
```

Pay attention to `available` memory, not only `free`, because Linux intentionally uses memory for caches.

---

### `uptime` — Uptime and Load Average

**What it does:** Shows how long the server has been running and load averages.

```bash
uptime
```

Example:

```text
up 10 days, load average: 0.45, 0.30, 0.20
```

Load averages roughly represent runnable/uninterruptible work averaged over 1, 5 and 15 minutes. Interpret them relative to CPU count and workload.

---

### `ip addr` — Network Interfaces/IP Addresses

**What it does:** Displays interfaces and assigned IP addresses.

**Why needed:** First-level network troubleshooting.

```bash
ip addr
```

Short form:

```bash
ip a
```

---

### `ip route` — Routing Table

**What it does:** Shows how Linux decides where network traffic should go.

**Why needed:** If an IP exists but another network cannot be reached, routing is one thing to inspect.

```bash
ip route
```

---

### `ping` — Basic Network Reachability

**What it does:** Sends ICMP echo requests.

```bash
ping 8.8.8.8
ping example.com
```

**Why needed:** Helps test reachability and sometimes DNS, although firewalls can block ICMP, so a failed ping does not prove the application is unavailable.

---

### `ss` — Socket/Port Information

**What it does:** Shows network sockets and listening ports.

**Why needed:** Critical when an application says it started but users cannot connect.

```bash
ss -lntp
```

```text
-l = listening
-n = numeric
-t = TCP
-p = process
```

Check 8080:

```bash
ss -lntp | grep 8080
```

---

### `nslookup` / `dig` — DNS Troubleshooting

**What they do:** Query DNS.

**Why needed:** If a hostname fails but an IP works, DNS may be the problem.

```bash
nslookup example.com
dig example.com
```

---

### `curl` — Test HTTP/API Connectivity

**What it does:** Transfers data using protocols including HTTP/HTTPS.

**Why needed:** Extremely useful for DevOps troubleshooting of APIs, health endpoints, proxies and web applications.

```bash
curl http://localhost:8080
curl -I https://example.com
curl -v https://example.com
```

```text
-I = response headers
-v = verbose connection/request details
```

---

### `env`, `printenv`, `export` and `$PATH`

Display environment:

```bash
env
printenv
```

Set/export:

```bash
export ENVIRONMENT=dev
```

Read:

```bash
echo "$ENVIRONMENT"
echo "$PATH"
```

**Why needed:** Applications frequently receive configuration through environment variables.

`PATH` tells the shell which directories to search for executable commands.

```bash
command -v docker
```

---

### `ssh` — Secure Remote Shell

**What it does:** Connects securely to another machine.

**Why needed:** Standard Linux remote administration method.

```bash
ssh devops@10.0.0.10
```

With key:

```bash
ssh -i ~/.ssh/id_rsa devops@10.0.0.10
```

---

### `scp` — Secure Copy

**What it does:** Copies files over SSH.

```bash
scp app.conf devops@10.0.0.10:/tmp/
scp devops@10.0.0.10:/var/log/app.log .
```

**Why needed:** Useful for transferring logs/configuration/artifacts when appropriate.

---

### `tar` and `gzip` — Archive/Compress

**What they do:** `tar` groups files into an archive; gzip compresses data.

Create `.tar.gz`:

```bash
tar -czvf backup.tar.gz website/
```

Extract:

```bash
tar -xzvf backup.tar.gz
```

Common flags:

```text
c = create
x = extract
z = gzip
v = verbose
f = archive filename follows
```

---

### `crontab` — Schedule Recurring Jobs

**What it does:** Manages cron schedules for a user.

**Why needed:** Useful for recurring scripts such as cleanup, backups, reports and health checks.

```bash
crontab -l
crontab -e
```

Cron fields:

```text
MINUTE HOUR DAY-OF-MONTH MONTH DAY-OF-WEEK COMMAND
```

Every five minutes:

```cron
*/5 * * * * /opt/scripts/health-check.sh
```

Every day at 2 AM:

```cron
0 2 * * * /opt/scripts/backup.sh
```

With logging:

```cron
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

**Interview scenario:** Works manually but not in cron.

Check environment variables, `PATH`, absolute paths, permissions, working-directory assumptions and credentials.

---

## Essential Troubleshooting Mindset

### Website/application is down

Do not randomly restart things. Work through layers:

```bash
systemctl status myapp
journalctl -u myapp -n 100
ss -lntp
curl -v http://localhost:8080
ip addr
ip route
```

Think:

```text
Process/Service
      ↓
Logs
      ↓
Listening Port
      ↓
Local Connectivity
      ↓
DNS / Network / Firewall / LB
```

### Server is slow

```bash
uptime
top
free -h
df -h
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

### Disk is full

```bash
df -h
df -i
du -xhd1 /var | sort -h
find /var -type f -size +1G
```

### Permission denied

```bash
whoami
id
ls -l file
```

Then determine whether ownership or permissions are actually incorrect before changing them.

### Docker container is down

```bash
docker ps -a
docker logs --tail 100 myapp
docker inspect myapp
```

### Docker application is running but unreachable

```bash
docker ps
docker port myapp
docker logs myapp
ss -lntp
curl http://localhost:<host-port>
```

---


# Complete Interview Reference

## Index

1. [Linux Basics](#1-linux-basics)
2. [Linux Filesystem](#2-linux-filesystem)
3. [Navigation Commands](#3-navigation-commands)
4. [File and Directory Commands](#4-file-and-directory-commands)
5. [Viewing Files and Logs](#5-viewing-files-and-logs)
6. [grep and find](#6-grep-and-find)
7. [Pipes and Redirection](#7-pipes-and-redirection)
8. [Permissions and Ownership](#8-permissions-and-ownership)
9. [Users, Groups, sudo and root](#9-users-groups-sudo-and-root)
10. [Processes: ps, top, kill, nice](#10-processes-ps-top-kill-nice)
11. [Services and systemctl](#11-services-and-systemctl)
12. [journalctl and System Logs](#12-journalctl-and-system-logs)
13. [Disk and Filesystem Troubleshooting](#13-disk-and-filesystem-troubleshooting)
14. [Memory and CPU](#14-memory-and-cpu)
15. [Networking](#15-networking)
16. [Environment Variables and PATH](#16-environment-variables-and-path)
17. [Package Management](#17-package-management)
18. [SSH and SCP](#18-ssh-and-scp)
19. [tar, gzip and Archives](#19-tar-gzip-and-archives)
20. [Cron Jobs](#20-cron-jobs)
21. [Links](#21-links)
22. [Mounts](#22-mounts)
23. [curl and wget](#23-curl-and-wget)
24. [Bash Scripting Basics](#24-bash-scripting-basics)
25. [Linux + Docker Interview Commands](#25-linux--docker-interview-commands)
26. [Common Troubleshooting Scenarios](#26-common-troubleshooting-scenarios)
27. [PowerShell to Linux Mapping](#27-powershell-to-linux-mapping)
28. [Rapid Interview Questions](#28-rapid-interview-questions)
29. [Last-Minute Command Cheat Sheet](#29-last-minute-command-cheat-sheet)

---

# 1. Linux Basics

### Check the current user

```bash
whoami
```

Example output:

```text
manideep
```

### Check kernel/system information

```bash
uname -a
```

Kernel release only:

```bash
uname -r
```

### Check Linux distribution

```bash
cat /etc/os-release
```

### Check hostname

```bash
hostname
```

---

# 2. Linux Filesystem

Important directories:

| Directory | Purpose |
|---|---|
| `/` | Root of the filesystem |
| `/home` | Normal users' home directories |
| `/root` | Root user's home |
| `/etc` | System/application configuration |
| `/var` | Variable data such as logs |
| `/var/log` | Common log location |
| `/tmp` | Temporary files |
| `/usr` | Programs, libraries and shared data |
| `/bin` / `/usr/bin` | Common commands |
| `/sbin` / `/usr/sbin` | Administration commands |
| `/opt` | Optional/third-party software |
| `/mnt` | Temporary/manual mount points |
| `/dev` | Device files |
| `/proc` | Process/kernel information |

Interview question: **What is `/etc`?**

Answer: It primarily contains system-wide configuration files.

---

# 3. Navigation Commands

### `pwd`

Print current working directory.

```bash
pwd
```

Example:

```text
/home/manideep/project
```

### `ls`

```bash
ls
ls -l
ls -a
ls -la
ls -lh
```

Useful combination:

```bash
ls -lah
```

### `cd`

```bash
cd /var/log
cd ..
cd ~
cd -
```

`cd -` returns to the previous directory.

---

# 4. File and Directory Commands

### Create a directory

```bash
mkdir docker-project
mkdir -p app/config/dev
```

### Create an empty file

```bash
touch application.log
```

### Copy

```bash
cp app.conf app.conf.backup
cp -r website/ website-backup/
```

### Move or rename

```bash
mv old.txt new.txt
mv app.log /tmp/
```

### Remove

```bash
rm file.txt
rm -r directory/
rm -rf directory/
```

`rm -rf` recursively and forcibly removes content. Be extremely careful, especially with `sudo`.

### File information

```bash
file archive.tar.gz
stat application.log
```

---

# 5. Viewing Files and Logs

### `cat`

```bash
cat application.conf
```

### `less`

Better for large files:

```bash
less /var/log/syslog
```

Press `q` to exit.

### `head`

```bash
head application.log
head -n 20 application.log
```

### `tail`

```bash
tail application.log
tail -n 100 application.log
```

### Follow a live log

Very common interview command:

```bash
tail -f application.log
```

Or:

```bash
tail -n 100 -f application.log
```

---

# 6. grep and find

## `grep`

Search text.

```bash
grep "ERROR" application.log
```

Case-insensitive:

```bash
grep -i "error" application.log
```

Show line numbers:

```bash
grep -n "ERROR" application.log
```

Recursive search:

```bash
grep -r "database" /etc/
```

Exclude matching lines:

```bash
grep -v "DEBUG" application.log
```

Count matches:

```bash
grep -c "ERROR" application.log
```

Combine:

```bash
grep -in "connection refused" application.log
```

## `find`

Find a file:

```bash
find /var -name "application.log"
```

Case-insensitive:

```bash
find /var -iname "Application.log"
```

Find directories:

```bash
find /opt -type d -name "logs"
```

Find `.log` files:

```bash
find /var/log -type f -name "*.log"
```

Find files larger than 100 MB:

```bash
find /var -type f -size +100M
```

Find files modified during the last day:

```bash
find /var/log -type f -mtime -1
```

---

# 7. Pipes and Redirection

## Pipe `|`

Pass stdout from one command to another.

```bash
ps aux | grep nginx
```

```bash
cat application.log | grep ERROR
```

A simpler form of the second command is:

```bash
grep ERROR application.log
```

## `>`

Overwrite/create a file with stdout:

```bash
echo "hello" > output.txt
```

## `>>`

Append stdout:

```bash
echo "another line" >> output.txt
```

## stderr

Send errors to a file:

```bash
command 2> error.log
```

stdout and stderr separately:

```bash
command > output.log 2> error.log
```

stdout and stderr together:

```bash
command > all.log 2>&1
```

Discard output:

```bash
command > /dev/null 2>&1
```

---

# 8. Permissions and Ownership

View permissions:

```bash
ls -l
```

Example:

```text
-rwxr-xr-- 1 manideep developers 500 app.sh
```

The three permission groups are:

```text
owner | group | others
r = read
w = write
x = execute
```

Numeric values:

```text
r = 4
w = 2
x = 1
```

Common permissions:

```text
755 = rwxr-xr-x
644 = rw-r--r--
700 = rwx------
```

### `chmod`

```bash
chmod 755 deploy.sh
chmod +x deploy.sh
chmod 644 config.txt
```

### `chown`

```bash
sudo chown manideep file.txt
sudo chown manideep:developers file.txt
sudo chown -R manideep:developers /opt/app
```

### `chgrp`

```bash
sudo chgrp developers file.txt
```

Common interview question: **Difference between `chmod` and `chown`?**

- `chmod` changes permissions.
- `chown` changes ownership.

---

# 9. Users, Groups, sudo and root

Current user:

```bash
whoami
```

User/group IDs:

```bash
id
```

Logged-in users:

```bash
who
```

Create user:

```bash
sudo useradd -m devuser
```

Set password:

```bash
sudo passwd devuser
```

Create group:

```bash
sudo groupadd devops
```

Add user to supplementary group:

```bash
sudo usermod -aG devops devuser
```

Run a privileged command:

```bash
sudo systemctl restart nginx
```

Open a root shell when permitted:

```bash
sudo -i
```

Interview question: **What is root?**

`root` is the superuser, normally UID `0`, with administrative privileges.

---

# 10. Processes: ps, top, kill, nice

This is a major interview topic.

## `ps`

Current shell processes:

```bash
ps
```

Common full listing:

```bash
ps aux
```

Find nginx:

```bash
ps aux | grep nginx
```

Another useful form:

```bash
ps -ef
```

## `pgrep`

```bash
pgrep nginx
pgrep -a nginx
```

## `top`

Interactive real-time process/CPU/memory view:

```bash
top
```

Useful keys inside `top`:

```text
P    sort by CPU
M    sort by memory
k    kill a process
1    show individual CPUs
q    quit
```

### `htop`

If installed:

```bash
htop
```

It provides a friendlier interactive process view, but do not assume it exists on every server.

## `kill`

Graceful termination:

```bash
kill <PID>
```

Explicit SIGTERM:

```bash
kill -15 <PID>
```

Force termination:

```bash
kill -9 <PID>
```

Example:

```bash
ps aux | grep java
kill 1234
```

If the process refuses to terminate and you understand the impact:

```bash
kill -9 1234
```

**Interview point:** Do not start with `kill -9`. `SIGKILL` cannot be caught by the process and prevents graceful cleanup. Try `SIGTERM` first.

Common signals:

| Signal | Number | Meaning |
|---|---:|---|
| SIGHUP | 1 | Hangup/reload in some applications |
| SIGINT | 2 | Interrupt, often Ctrl+C |
| SIGTERM | 15 | Graceful termination request |
| SIGKILL | 9 | Immediate forced termination |

List signals:

```bash
kill -l
```

Kill by process name:

```bash
pkill nginx
```

## `nice` and `renice`

Start with adjusted scheduling priority:

```bash
nice -n 10 ./script.sh
```

Change an existing process:

```bash
sudo renice 10 -p 1234
```

---

# 11. Services and systemctl

Modern distributions commonly use `systemd`.

### Check a service

```bash
systemctl status nginx
```

### Start

```bash
sudo systemctl start nginx
```

### Stop

```bash
sudo systemctl stop nginx
```

### Restart

```bash
sudo systemctl restart nginx
```

### Reload configuration when supported

```bash
sudo systemctl reload nginx
```

### Start automatically at boot

```bash
sudo systemctl enable nginx
```

### Disable automatic startup

```bash
sudo systemctl disable nginx
```

### Enable and start now

```bash
sudo systemctl enable --now nginx
```

### Check whether enabled

```bash
systemctl is-enabled nginx
```

### Check whether currently active

```bash
systemctl is-active nginx
```

### List running services

```bash
systemctl list-units --type=service --state=running
```

### List failed units

```bash
systemctl --failed
```

### Typical services interviewers may mention

Depending on the server:

```text
nginx
apache2
httpd
sshd
docker
cron
crond
mysql
mysqld
postgresql
```

Examples:

```bash
systemctl status sshd
systemctl status docker
systemctl status nginx
```

Service naming differs by distribution. For example, Apache is commonly `apache2` on Ubuntu/Debian and `httpd` on RHEL-family systems.

### Interview scenario: nginx is down

```bash
systemctl status nginx
sudo systemctl start nginx
journalctl -u nginx
```

---

# 12. journalctl and System Logs

Logs are critical for troubleshooting.

### Recent system journal

```bash
journalctl
```

### Logs for a service

```bash
journalctl -u nginx
```

### Follow logs

```bash
journalctl -u nginx -f
```

### Current boot

```bash
journalctl -b
```

### Recent errors

```bash
journalctl -p err
```

### Last 100 lines for a service

```bash
journalctl -u docker -n 100
```

Common traditional log locations include:

```text
/var/log/syslog
/var/log/messages
/var/log/auth.log
/var/log/secure
```

Availability depends on the distribution and logging configuration.

---

# 13. Disk and Filesystem Troubleshooting

## `df`

Filesystem usage:

```bash
df -h
```

Inode usage:

```bash
df -i
```

## `du`

Directory usage:

```bash
du -sh /var/log
```

Sizes of immediate children:

```bash
du -sh /var/log/*
```

Useful troubleshooting pattern:

```bash
du -xhd1 /var | sort -h
```

## `lsblk`

```bash
lsblk
```

Show filesystem information:

```bash
lsblk -f
```

## Disk-full interview scenario

Start with:

```bash
df -h
df -i
```

Then identify large directories/files:

```bash
du -xhd1 /var | sort -h
find /var -type f -size +1G
```

Do not blindly delete logs or system files. Determine what is consuming the space and why.

---

# 14. Memory and CPU

Memory:

```bash
free -h
```

CPU/process usage:

```bash
top
```

Load averages / uptime:

```bash
uptime
```

Example:

```text
12:10:00 up 5 days, 2 users, load average: 0.50, 0.40, 0.30
```

Sort processes by CPU:

```bash
ps aux --sort=-%cpu | head
```

Sort by memory:

```bash
ps aux --sort=-%mem | head
```

Interview scenario: **Server is slow**

Initial checks:

```bash
uptime
top
free -h
df -h
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

Then investigate the offending process, logs, disk I/O, network, or application dependencies.

---

# 15. Networking

## IP addresses

```bash
ip addr
```

Short form:

```bash
ip a
```

## Routes

```bash
ip route
```

## Connectivity

```bash
ping 8.8.8.8
ping google.com
```

## Listening ports

Very important:

```bash
ss -lntp
```

Common flags:

```text
-l = listening
-n = numeric addresses/ports
-t = TCP
-p = process
```

Check port 8080:

```bash
ss -lntp | grep 8080
```

## DNS

```bash
nslookup example.com
```

If available:

```bash
dig example.com
```

## Hostname

```bash
hostname
hostname -I
```

Interview scenario: **Application cannot be reached**

Check:

```bash
systemctl status myapp
ss -lntp
curl http://localhost:8080
ip addr
ip route
```

Then investigate DNS, firewall/security rules, routing, load balancer, container port mappings, etc.

---

# 16. Environment Variables and PATH

Display variables:

```bash
env
printenv
```

Read one:

```bash
echo $PATH
echo $HOME
```

Set a shell variable:

```bash
NAME="Manideep"
echo "$NAME"
```

Export it to child processes:

```bash
export ENVIRONMENT=dev
echo "$ENVIRONMENT"
```

Set for one command only:

```bash
ENVIRONMENT=dev ./app.sh
```

Add a directory to `PATH` for the current shell:

```bash
export PATH="$PATH:/opt/myapp/bin"
```

Interview question: **What is PATH?**

`PATH` is a colon-separated list of directories the shell searches for executable commands.

Find a command:

```bash
which python3
command -v docker
```

---

# 17. Package Management

Ubuntu/Debian:

```bash
sudo apt update
sudo apt install nginx
sudo apt remove nginx
apt list --installed
```

RHEL/Fedora:

```bash
sudo dnf install nginx
sudo dnf remove nginx
dnf list installed
```

Older RHEL/CentOS environments may use:

```bash
yum install nginx
```

RPM packages:

```bash
rpm -qa
rpm -q nginx
```

---

# 18. SSH and SCP

Connect:

```bash
ssh user@server
```

Example:

```bash
ssh devops@10.0.0.10
```

Specific private key:

```bash
ssh -i ~/.ssh/id_rsa devops@10.0.0.10
```

Copy local file to remote server:

```bash
scp app.conf devops@10.0.0.10:/tmp/
```

Copy from remote:

```bash
scp devops@10.0.0.10:/var/log/app.log .
```

Recursive directory:

```bash
scp -r website/ devops@10.0.0.10:/opt/
```

---

# 19. tar, gzip and Archives

Create a tar archive:

```bash
tar -cvf website.tar website/
```

Extract:

```bash
tar -xvf website.tar
```

Create gzip-compressed tar archive:

```bash
tar -czvf website.tar.gz website/
```

Extract:

```bash
tar -xzvf website.tar.gz
```

List archive contents:

```bash
tar -tzf website.tar.gz
```

Compress one file:

```bash
gzip application.log
```

Decompress:

```bash
gunzip application.log.gz
```

Common tar flags:

```text
-c = create
-x = extract
-t = list
-z = gzip
-v = verbose
-f = archive filename
```

---

# 20. Cron Jobs

Cron schedules recurring commands.

Edit current user's crontab:

```bash
crontab -e
```

List jobs:

```bash
crontab -l
```

Cron format:

```text
* * * * * command
│ │ │ │ │
│ │ │ │ └── day of week (0-7, Sunday commonly 0 or 7)
│ │ │ └──── month (1-12)
│ │ └────── day of month (1-31)
│ └──────── hour (0-23)
└────────── minute (0-59)
```

### Every minute

```cron
* * * * * /opt/scripts/check.sh
```

### Every day at 2:00 AM

```cron
0 2 * * * /opt/scripts/backup.sh
```

### Every Monday at 9:00 AM

```cron
0 9 * * 1 /opt/scripts/report.sh
```

### Every 5 minutes

```cron
*/5 * * * * /opt/scripts/health-check.sh
```

### Redirect cron output

```cron
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

Use absolute paths in cron where practical because cron often has a smaller environment/PATH than your interactive shell.

Check cron service:

Ubuntu/Debian:

```bash
systemctl status cron
```

RHEL-family:

```bash
systemctl status crond
```

Interview question: **Script works manually but fails in cron. Why?**

Common causes include:

- Different/minimal `PATH`
- Missing environment variables
- Relative paths
- Different working directory
- Permissions
- Credentials unavailable to the cron environment

---

# 21. Links

## Symbolic link

```bash
ln -s /opt/app/current/app.conf /etc/myapp/app.conf
```

Inspect:

```bash
ls -l /etc/myapp/app.conf
```

A symlink references another pathname and can cross filesystems.

## Hard link

```bash
ln original.txt hardlink.txt
```

A hard link is another directory entry referring to the same inode/data. It normally cannot cross filesystems and is generally not used for directories.

---

# 22. Mounts

Show mounted filesystems:

```bash
mount
```

More readable filesystem usage:

```bash
df -h
```

Block devices:

```bash
lsblk -f
```

Mount example:

```bash
sudo mount /dev/sdb1 /mnt/data
```

Unmount:

```bash
sudo umount /mnt/data
```

Persistent mounts are commonly configured in:

```text
/etc/fstab
```

Be careful when editing `/etc/fstab`; invalid entries can affect boot/mount behavior.

---

# 23. curl and wget

## `curl`

HTTP GET:

```bash
curl https://example.com
```

Headers only:

```bash
curl -I https://example.com
```

Verbose troubleshooting:

```bash
curl -v https://example.com
```

API call:

```bash
curl -H "Accept: application/json" https://api.example.com/health
```

POST JSON:

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name":"test"}' \
  https://api.example.com/items
```

Test local service:

```bash
curl http://localhost:8080/health
```

## `wget`

Download:

```bash
wget https://example.com/file.zip
```

Choose filename:

```bash
wget -O app.zip https://example.com/file.zip
```

---

# 24. Bash Scripting Basics

Shebang:

```bash
#!/bin/bash
```

Variables:

```bash
NAME="Manideep"
echo "Hello $NAME"
```

Arguments:

```bash
echo "Script: $0"
echo "First argument: $1"
echo "All arguments: $@"
```

Exit code:

```bash
echo $?
```

By convention, `0` means success and non-zero indicates failure.

Condition:

```bash
if [ -f "/etc/os-release" ]; then
    echo "File exists"
else
    echo "File does not exist"
fi
```

Loop:

```bash
for server in web01 web02 web03
do
    echo "Checking $server"
done
```

Executable script:

```bash
chmod +x health-check.sh
./health-check.sh
```

Useful operators:

```text
-f file     file exists and is a regular file
-d dir      directory exists
-e path     path exists
-z string   string is empty
-n string   string is non-empty
-eq         numeric equal
-ne         numeric not equal
```

---

# 25. Linux + Docker Interview Commands

List containers:

```bash
docker ps
docker ps -a
```

Images:

```bash
docker image ls
```

Logs:

```bash
docker logs myapp
docker logs -f myapp
docker logs --tail 100 myapp
```

Execute inside a running container:

```bash
docker exec -it myapp bash
```

For minimal images:

```bash
docker exec -it myapp sh
```

Inspect:

```bash
docker inspect myapp
```

Processes:

```bash
docker top myapp
```

Resource usage:

```bash
docker stats
```

Port mapping:

```bash
docker port myapp
```

Host ports:

```bash
ss -lntp
```

Container stopped unexpectedly:

```bash
docker ps -a
docker logs myapp
docker inspect myapp
```

Important interview concept:

**Why does a container stop immediately?**

A container normally stays alive while its main process (PID 1) is running. If the main process exits, the container stops.

---

# 26. Common Troubleshooting Scenarios

## Scenario 1: Service is down

```bash
systemctl status nginx
journalctl -u nginx -n 100
sudo systemctl restart nginx
systemctl status nginx
```

Do not restart blindly in production without considering impact and understanding the failure.

## Scenario 2: Port is not reachable

```bash
systemctl status myapp
ss -lntp | grep 8080
curl -v http://localhost:8080
```

Then check firewall/network/security rules, DNS, routes, load balancers, or Docker port publishing as appropriate.

## Scenario 3: Disk is full

```bash
df -h
df -i
du -xhd1 /var | sort -h
find /var -type f -size +1G
```

## Scenario 4: High CPU

```bash
top
ps aux --sort=-%cpu | head
```

Identify the PID, then inspect the application/service and its logs.

## Scenario 5: High memory

```bash
free -h
top
ps aux --sort=-%mem | head
```

## Scenario 6: Process needs to be stopped

```bash
ps aux | grep myapp
kill <PID>
```

Only if graceful termination fails and force is justified:

```bash
kill -9 <PID>
```

## Scenario 7: Application logs show errors

```bash
tail -n 100 application.log
grep -in "error" application.log
tail -f application.log
```

## Scenario 8: Cannot find a file

```bash
find / -name "application.conf" 2>/dev/null
```

## Scenario 9: Permission denied

```bash
ls -l file.txt
id
```

Then determine whether permissions or ownership should legitimately be changed:

```bash
chmod 644 file.txt
sudo chown user:group file.txt
```

Avoid using `chmod 777` as a default fix.

## Scenario 10: DNS vs network problem

First test basic network connectivity:

```bash
ping 8.8.8.8
```

Then DNS:

```bash
nslookup example.com
```

Then application connectivity:

```bash
curl -v https://example.com
```

## Scenario 11: Docker application unreachable

```bash
docker ps
docker logs myapp
docker port myapp
docker inspect myapp
ss -lntp
curl http://localhost:<host-port>
```

---

# 27. PowerShell to Linux Mapping

| PowerShell | Linux |
|---|---|
| `Get-Location` | `pwd` |
| `Get-ChildItem` | `ls` |
| `Set-Location` | `cd` |
| `Get-Content` | `cat`, `less` |
| `Get-Content -Tail 10` | `tail -n 10` |
| `Get-Content -Wait` | `tail -f` |
| `New-Item -ItemType Directory` | `mkdir` |
| `New-Item file.txt` | `touch file.txt` |
| `Copy-Item` | `cp` |
| `Move-Item` | `mv` |
| `Remove-Item` | `rm` |
| `Select-String` | `grep` |
| `Get-Process` | `ps`, `top` |
| `Stop-Process` | `kill` |
| `Get-Service` | `systemctl` |
| `$env:PATH` | `$PATH` |
| `Get-ChildItem Env:` | `env` |
| `$LASTEXITCODE` | `$?` |
| `Invoke-WebRequest` | `curl`, `wget` |
| `Test-Connection` | `ping` |
| `Compress-Archive` | `tar`, `gzip` |
| Scheduled Task | `cron` / systemd timer |

---

# 28. Rapid Interview Questions

### What is the difference between `>` and `>>`?

`>` overwrites/creates a file. `>>` appends.

### What does `|` do?

It sends stdout from the command on the left to stdin of the command on the right.

```bash
ps aux | grep nginx
```

### `grep` vs `find`?

`grep` searches text/content. `find` searches filesystem paths based on name, type, size, time, etc.

### `chmod` vs `chown`?

`chmod` changes permissions. `chown` changes ownership.

### `kill` vs `kill -9`?

`kill PID` normally sends SIGTERM (15), allowing graceful shutdown. `kill -9 PID` sends SIGKILL and immediately forces termination.

### How do you check disk usage?

```bash
df -h
du -sh /var/log
```

### How do you check memory?

```bash
free -h
```

### How do you check CPU usage?

```bash
top
```

### How do you identify a process using high CPU?

```bash
top
ps aux --sort=-%cpu | head
```

### How do you check running services?

```bash
systemctl list-units --type=service --state=running
```

### How do you troubleshoot a failed service?

```bash
systemctl status <service>
journalctl -u <service>
```

### How do you check listening ports?

```bash
ss -lntp
```

### How do you monitor logs continuously?

```bash
tail -f application.log
```

or:

```bash
journalctl -u nginx -f
```

### How do you find ERROR messages?

```bash
grep -in "error" application.log
```

### How do you find a file?

```bash
find /var -name "application.log"
```

### What is cron?

A time-based job scheduler commonly used to run commands/scripts periodically.

### How do you schedule a script every five minutes?

```cron
*/5 * * * * /opt/scripts/script.sh
```

### What is `$PATH`?

A list of directories searched by the shell when resolving command names.

### How do you make a script executable?

```bash
chmod +x script.sh
```

### What does `sudo` do?

It executes an authorized command with elevated privileges, commonly as root.

### What is PID 1?

It is the first userspace process in a Linux process namespace. On many hosts it is the init system such as systemd. In a Docker container, it is the container's main process.

---

# 29. Last-Minute Command Cheat Sheet

```bash
# System
whoami
id
uname -a
cat /etc/os-release
hostname

# Navigation
pwd
ls -lah
cd /var/log
cd ..
cd ~

# Files
touch file.txt
mkdir -p app/logs
cp file.txt backup.txt
cp -r app/ app-backup/
mv old.txt new.txt
rm file.txt
rm -r directory/

# Read logs/files
cat file.txt
less file.txt
head -n 20 file.txt
tail -n 100 file.txt
tail -f application.log

# Search
grep -in "error" application.log
grep -r "text" /etc/
find /var -name "*.log"
find /var -type f -size +100M

# Permissions
ls -l
chmod +x script.sh
chmod 755 script.sh
chown user:group file.txt

# Processes
ps aux
ps -ef
pgrep -a nginx
top
kill PID
kill -15 PID
kill -9 PID

# Services
systemctl status nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl enable nginx
systemctl --failed

# Logs
journalctl -u nginx
journalctl -u nginx -f
journalctl -u nginx -n 100

# Disk
df -h
df -i
du -sh /var/log
lsblk -f

# Memory / CPU
free -h
uptime
top
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head

# Network
ip addr
ip route
ping example.com
ss -lntp
nslookup example.com
curl -v http://localhost:8080

# Environment
env
printenv
echo $PATH
export ENVIRONMENT=dev
command -v docker

# SSH
ssh user@server
scp file.txt user@server:/tmp/

# Archives
tar -czvf backup.tar.gz directory/
tar -xzvf backup.tar.gz

# Cron
crontab -l
crontab -e

# Docker
docker ps
docker ps -a
docker image ls
docker logs -f container
docker exec -it container bash
docker inspect container
docker stats
docker port container
```

---

## Final Interview Strategy

Do not try to pretend you have years of Linux administration experience. A strong answer when you are new to a command is:

> "I haven't used that command extensively yet, but I understand the Linux concept. I would first inspect the current state, check the relevant logs, and verify the documentation before making a production change."

For troubleshooting questions, explain your **sequence**, not just a command.

For example, if an application is unreachable:

```text
1. Is the process/service running?
        ↓
2. What do the application/system logs say?
        ↓
3. Is the expected port listening?
        ↓
4. Does localhost connectivity work?
        ↓
5. Are DNS/routing/firewall/security rules correct?
        ↓
6. If containerized, is Docker publishing the expected port?
```

That troubleshooting mindset is often more valuable in an interview than memorizing every Linux command.
