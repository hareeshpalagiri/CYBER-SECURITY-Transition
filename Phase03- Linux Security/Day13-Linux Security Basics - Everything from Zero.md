# 🔵 PHASE 3 — DAY 11
# Linux Security Basics — Everything From Zero
### *SecureCorp Ltd. | Understanding Linux Through a Security Lens*

---

## 🧭 DAY 11 MINDSET

```
YOUR LINUX EXPERIENCE RIGHT NOW:
─────────────────────────────────────────────────────────
You have Kali Linux installed.
You may have run a few commands.
But Linux internals — users, permissions, processes —
are not yet fully clear.

THAT IS PERFECTLY FINE.
Most Windows admins transitioning to security
feel the same way about Linux.

WHY LINUX MATTERS FOR SOC ANALYSTS:
─────────────────────────────────────────────────────────
→ 90% of web servers run Linux
→ Most cloud infrastructure runs Linux (Azure VMs)
→ Attack tools (Kali, Metasploit) run on Linux
→ Docker containers are Linux-based
→ Most SIEM/log infrastructure runs on Linux
→ Attackers love Linux because it is powerful and quiet

SecureCorp has:
→ WEB01 — Linux web server (Apache)
→ Kali Linux — Navi's attack/defence workstation
→ Azure VMs — Ubuntu-based ML servers for Gani
→ Log collectors — Linux syslog forwarders

BY END OF PHASE 3:
You will read Linux logs, detect attacks,
harden Linux servers, and understand privilege
escalation — the core Linux security skills
every SOC Analyst must have.
─────────────────────────────────────────────────────────
```

---

## 📌 SECTION 1 of 6
## Linux vs Windows — The Mental Model Shift

---

### Everything is a File

```
THE MOST IMPORTANT LINUX CONCEPT:
─────────────────────────────────────────────────────────
In Windows: Things are objects with GUIs
  → Users managed in Active Directory GUI
  → Settings managed in Registry (binary format)
  → Services managed in Services.msc
  → Everything is compartmentalised

In Linux: EVERYTHING is a file
  → Users defined in /etc/passwd (plain text file)
  → Passwords stored in /etc/shadow (plain text file)
  → Network config in /etc/network/interfaces (text file)
  → Service config in /etc/servicename/ (text files)
  → Hardware devices appear as files in /dev/
  → Running processes visible as files in /proc/
  → System settings in /etc/ (all text files)

WHY THIS MATTERS FOR SECURITY:
  → You can READ configuration with any text tool
  → You can MODIFY behaviour by editing files
  → Attackers read sensitive files to find credentials
  → Defenders check file contents to detect tampering
  → Log analysis = reading text files in /var/log/
  → Everything is auditable because everything is text

SECURECORP EXAMPLE:
  On Windows DC01:
  → Open Active Directory Users and Computers GUI
  → Click through menus to see users

  On Linux WEB01:
  → cat /etc/passwd
  → Instantly see all users in plain text
  → No GUI needed
  → Scripts can process it automatically
```

### The Linux Filesystem — Where Everything Lives

```
LINUX DIRECTORY STRUCTURE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
/                    Root — top of everything
│
├── /etc/            Configuration files (CRITICAL for security)
│   ├── passwd       User account definitions
│   ├── shadow       Password hashes (protected)
│   ├── group        Group definitions
│   ├── sudoers      Who can run commands as root
│   ├── ssh/         SSH server configuration
│   ├── cron.d/      Scheduled tasks
│   └── hosts        Local DNS entries
│
├── /var/            Variable data (changes constantly)
│   ├── log/         ALL LOG FILES ARE HERE
│   │   ├── auth.log         Authentication events
│   │   ├── syslog           General system events
│   │   ├── apache2/         Web server logs
│   │   └── audit/audit.log  Security audit log
│   └── www/html/    Web server files
│
├── /home/           Regular user home directories
│   ├── gani/        Gani's home directory
│   ├── priya/       Priya's home directory
│   └── navi/        Navi's home directory
│
├── /root/           Root (admin) home directory
│                    ONLY root can access this
│
├── /bin/            Essential commands (ls, cat, cp)
├── /sbin/           System commands (root-only tools)
├── /usr/            User programs and utilities
├── /tmp/            Temporary files (world-writable!)
│                    ATTACKERS LOVE /tmp/ — stays between reboots
│                    Files here can be executed
│
├── /proc/           Running processes (virtual filesystem)
│   ├── /proc/1/     Process with ID 1 (init/systemd)
│   └── /proc/[pid]/ Every running process has a folder
│                    Read process memory, cmdline, env vars
│
└── /dev/            Device files
    ├── sda          First hard disk
    ├── null         Null device (discard output)
    └── random       Random number generator

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECURITY HOTSPOTS — NAVI CHECKS THESE FIRST:
  /etc/passwd    → Who has user accounts?
  /etc/shadow    → Are password hashes strong?
  /etc/sudoers   → Who can escalate to root?
  /etc/cron*     → What scheduled tasks exist?
  /tmp/          → Any suspicious files or scripts?
  /var/log/      → What do the logs say?
  /home/*/       → Suspicious files in user dirs?
  /root/         → What has root been doing?
  ~/.ssh/        → Authorised SSH keys (backdoors!)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📌 SECTION 2 of 6
## Linux Users and Groups — The Identity System

---

### Users in Linux — The /etc/passwd File

```
WHAT IS /etc/passwd:
Every user account on a Linux system is defined here.
It is a plain text file. Anyone can READ it.
(The actual password hashes are in /etc/shadow)

VIEW IT:
cat /etc/passwd

SAMPLE OUTPUT ON WEB01:
─────────────────────────────────────────────────────────
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
hareesh:x:1000:1000:Hareesh Kumar:/home/hareesh:/bin/bash
navi:x:1001:1001:Navi SOC:/home/navi:/bin/bash
gani:x:1002:1002:Gani ML:/home/gani:/bin/bash
apache-svc:x:1003:1003:Apache Service:/var/www:/usr/sbin/nologin
─────────────────────────────────────────────────────────

EACH LINE HAS 7 FIELDS SEPARATED BY COLONS:
Field 1: Username         → hareesh
Field 2: Password         → x (means: look in /etc/shadow)
Field 3: UID (User ID)    → 1000 (unique number for this user)
Field 4: GID (Group ID)   → 1000 (primary group)
Field 5: Comment/GECOS    → Hareesh Kumar (full name)
Field 6: Home directory   → /home/hareesh
Field 7: Login shell      → /bin/bash

SECURITY ANALYSIS OF /etc/passwd:
─────────────────────────────────────────────────────────
LOOK FOR:

UID 0 = ROOT PRIVILEGES:
  root:x:0:0:...
  Any account with UID 0 = full root privileges
  There should be ONLY ONE UID 0 account (root)
  
  If you see:
  hacker:x:0:0::/root:/bin/bash
  = BACKDOOR ACCOUNT with root access
  = Attacker created this to maintain access

  Check command:
  awk -F: '($3 == "0") {print}' /etc/passwd
  Should return ONLY the root line.

SHELL FIELD ANALYSIS:
  /bin/bash or /bin/sh = Can log in interactively
  /usr/sbin/nologin    = Service account, cannot login
  /bin/false           = Cannot login at all

  Service accounts (www-data, daemon) should have nologin.
  If www-data has /bin/bash = security misconfiguration.
  Attacker who compromises Apache → can get a real shell.

  Check service accounts:
  grep -v nologin /etc/passwd | grep -v false
  Shows accounts that CAN log in interactively.

NEW ACCOUNTS RECENTLY CREATED:
  ls -lt /home/
  Lists home directories by modification time.
  New directory = new user account.
  Was this user supposed to be created?
```

### The /etc/shadow File — Where Passwords Live

```
WHAT IS /etc/shadow:
Stores password hashes for all users.
ONLY root can read this file (unlike /etc/passwd).
This is what attackers try to access.

VIEW IT (requires root/sudo):
sudo cat /etc/shadow

SAMPLE OUTPUT:
─────────────────────────────────────────────────────────
root:$6$rounds=5000$xyz...$[long hash]:19000:0:99999:7:::
hareesh:$6$rounds=5000$abc...$[long hash]:19100:0:90:7:::
navi:$6$rounds=5000$def...$[long hash]:19200:0:90:7:::
gani:!:19300:0:90:7:::
daemon:*:19000:0:99999:7:::
─────────────────────────────────────────────────────────

EACH LINE HAS 9 FIELDS:
Field 1: Username
Field 2: Password hash (or special character)
Field 3: Last password change (days since Jan 1 1970)
Field 4: Minimum days between changes (0 = change anytime)
Field 5: Maximum password age (90 = must change every 90 days)
Field 6: Warning days (7 = warn 7 days before expiry)
Field 7: Inactive days (blank = no inactivity lockout)
Field 8: Account expiry date (blank = never expires)
Field 9: Reserved

PASSWORD FIELD SPECIAL VALUES:
  $6$...  = SHA-512 hash (strong, good)
  $5$...  = SHA-256 hash (acceptable)
  $1$...  = MD5 hash (WEAK — should not exist)
  $2y$... = bcrypt hash (strong, good)
  *       = Account locked, no password login allowed
  !       = Account locked (password login disabled)
            (Gani's! above = password login disabled)
  !!      = Password never set (new account)
  (empty) = NO PASSWORD REQUIRED = CRITICAL VULNERABILITY

SECURITY CHECKS ON /etc/shadow:
─────────────────────────────────────────────────────────
CHECK 1 — Empty passwords (critical):
  sudo awk -F: '($2 == "") {print $1}' /etc/shadow
  Any output = account with NO password = attacker can login
  as this user with no credentials at all

CHECK 2 — MD5 hashes (weak):
  sudo grep '$1$' /etc/shadow
  MD5 hashes are crackable very quickly
  Should force password reset for these accounts

CHECK 3 — Password never expires:
  sudo awk -F: '($5 == "99999" || $5 == "") {print $1}' /etc/shadow
  99999 or empty = password never expires
  Service accounts OK, human accounts should rotate

CHECK 4 — Can shadow file be read by non-root?
  ls -la /etc/shadow
  Should show: -rw-r----- root shadow
  If world-readable: chmod 640 /etc/shadow immediately
```

### The /etc/group File — Group Management

```
WHAT IS /etc/group:
Defines groups and their members.
Groups control access to shared resources.

cat /etc/group

SAMPLE OUTPUT:
─────────────────────────────────────────────────────────
root:x:0:
sudo:x:27:hareesh
adm:x:4:hareesh,navi
docker:x:999:gani,priya
www-data:x:33:
securecorp-admin:x:1010:hareesh
securecorp-dev:x:1011:priya,gani
─────────────────────────────────────────────────────────

FORMAT: groupname:password:GID:members

CRITICAL GROUPS FROM SECURITY PERSPECTIVE:
─────────────────────────────────────────────────────────
sudo group (or wheel on RHEL):
  Members can run ANY command as root using sudo
  This is equivalent to Domain Admins in Active Directory
  
  Check: grep '^sudo:' /etc/group
  Should contain: ONLY authorised admins (Hareesh, Navi)
  If you see gani or any unexpected user = escalation risk

adm group:
  Members can read most log files in /var/log/
  Useful for security monitoring (Navi should be here)
  But: should not include regular users

docker group:
  Members can run Docker containers
  CRITICAL: Docker group = effective root access
  docker run -v /:/mnt --rm -it alpine chroot /mnt sh
  Above command gives root shell to the host = privilege escalation
  Docker group members = effectively root
  
  Check: grep docker /etc/group
  Gani being in docker = dangerous if Gani's account compromised

shadow group:
  Members can read /etc/shadow
  Should be: EMPTY or only specific system processes
  Any human in shadow group = can steal all password hashes

SECURITY CHECK — WHO CAN BECOME ROOT:
  grep -E '^sudo|^wheel|^admin' /etc/group
  This shows all accounts with sudo access.
  Verify every single one is authorised.
```

### The /etc/sudoers File — The Keys to the Kingdom

```
WHAT IS /etc/sudoers:
Defines who can run which commands as root (or other users).
This is the most security-critical configuration file
on any Linux system.

NEVER edit directly — use visudo:
  sudo visudo

SAMPLE CONTENTS:
─────────────────────────────────────────────────────────
# Root can run anything as anyone
root    ALL=(ALL:ALL) ALL

# Members of sudo group can run anything with password
%sudo   ALL=(ALL:ALL) ALL

# Hareesh can run anything with password
hareesh ALL=(ALL:ALL) ALL

# Navi can only run specific security tools
navi    ALL=(ALL) /usr/bin/auditctl, /usr/bin/ausearch

# Gani can run ML scripts without password (BAD!)
gani    ALL=(ALL) NOPASSWD: /opt/ml/train.py

# Apache service restart only (good practice)
www-data ALL=(root) NOPASSWD: /usr/bin/systemctl restart apache2
─────────────────────────────────────────────────────────

SUDOERS FORMAT:
WHO  WHERE=(AS_WHOM) COMMAND

WHO:      username or %groupname
WHERE:    ALL (any host) or specific hostname
AS_WHOM:  (ALL) = any user, (root) = only root
COMMAND:  ALL = everything, or specific path

DANGEROUS SUDOERS CONFIGURATIONS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
DANGER 1 — NOPASSWD ALL:
  gani ALL=(ALL) NOPASSWD: ALL
  
  Gani can run ANY command as root WITHOUT a password.
  If Gani's account is compromised:
  → Attacker runs: sudo bash
  → Gets root shell immediately
  → No password barrier at all

DANGER 2 — Wildcard in command:
  hareesh ALL=(ALL) /usr/bin/vi *
  
  Hareesh can run vi on ANY file as root.
  vi has a shell escape: :!bash
  → Opens a root shell from within vi
  → Hareesh has full root access via vi escape
  
  Other dangerous commands with shell escape:
  vi, vim, nano, less, more, man, awk, python, perl,
  find, nmap, tar, zip, bash, sh, python3

DANGER 3 — Editing with sudo:
  hareesh ALL=(ALL) /usr/bin/chmod
  
  Can change permissions on /etc/shadow
  → chmod 777 /etc/shadow → anyone reads password hashes

DANGER 4 — sudoers.d/ directory not reviewed:
  ls /etc/sudoers.d/
  Additional sudoers rules in this directory.
  Attackers add files here for persistence.
  A file like /etc/sudoers.d/backdoor with:
  hacker ALL=(ALL) NOPASSWD: ALL
  = backdoor that survives regular audits

SECURITY AUDIT OF SUDOERS:
  sudo cat /etc/sudoers
  sudo ls -la /etc/sudoers.d/
  sudo cat /etc/sudoers.d/*
  
  Question every line:
  → Who is this person/service?
  → Do they NEED this level of access?
  → Can we restrict the command to something specific?
  → Is NOPASSWD justified?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📌 SECTION 3 of 6
## Linux File Permissions — Reading and Understanding

---

### The Permission System — Explained Simply

```
EVERY FILE AND DIRECTORY HAS THREE PERMISSION SETS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Run: ls -la /etc/shadow

Output:
-rw-r----- 1 root shadow 1234 Apr 1 2026 /etc/shadow

BREAKING DOWN THE PERMISSION STRING:
-  rw-  r--  ---
│   │    │    │
│   │    │    └── Other (everyone else): no permissions
│   │    └─────── Group (shadow group): read only
│   └──────────── Owner (root): read and write
└──────────────── File type (- = file, d = directory)

THREE PERMISSION TYPES:
  r = read    (value: 4)  Can view the file contents
  w = write   (value: 2)  Can modify the file
  x = execute (value: 1)  Can run as program (files)
                          Can enter directory (directories)
  - = no permission (value: 0)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NUMERIC (OCTAL) PERMISSIONS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Add the values: r=4, w=2, x=1, -=0

rw-  =  4+2+0  = 6
r--  =  4+0+0  = 4
---  =  0+0+0  = 0

So -rw-r----- = 640

Common permission values:
  777 = rwxrwxrwx = EVERYONE can read/write/execute
        ← DANGEROUS: never for sensitive files
  755 = rwxr-xr-x = Owner full, others read+execute
        ← Normal for directories and executables
  644 = rw-r--r-- = Owner read/write, others read only
        ← Normal for regular files
  640 = rw-r----- = Owner read/write, group read, others nothing
        ← Good for sensitive files (/etc/shadow)
  600 = rw------- = Only owner can read/write
        ← SSH private keys must be 600
  000 = ----------= Nobody can access
        ← Extreme restriction

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SECURECORP WEB01 — PERMISSION CHECK:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

ls -la /etc/

-rw-r--r--  root root  /etc/passwd      644  ✅ Normal
-rw-r-----  root shadow /etc/shadow     640  ✅ Correct
-rw-r--r--  root root  /etc/group       644  ✅ Normal
-rw-r-----  root root  /etc/sudoers     440  ✅ Correct
-rw-r--r--  root root  /etc/ssh/sshd_config 644 ✅ Normal

BAD EXAMPLES:
-rw-rw-rw-  root root  /etc/passwd      666  ❌ Anyone can write!
-rw-r--r--  root root  /etc/shadow      644  ❌ Anyone can read hashes!
-rwxrwxrwx  root root  /etc/sudoers     777  ❌ Catastrophic!
```

### Changing Permissions — The Commands

```
CHMOD — Change File Permissions:
─────────────────────────────────────────────────────────
# Numeric method:
chmod 640 /etc/shadow    # rw-r----- (correct for shadow)
chmod 644 /etc/passwd    # rw-r--r-- (correct for passwd)
chmod 600 ~/.ssh/id_rsa  # rw------- (correct for SSH key)
chmod 755 /usr/bin/script.sh  # rwxr-xr-x (executable)

# Symbolic method:
chmod u+x script.sh    # Add execute for owner
chmod g-w /etc/shadow  # Remove write from group
chmod o-r /etc/shadow  # Remove read from others
chmod a-x /tmp/danger  # Remove execute from all (a=all)

# Recursive (for directories):
chmod -R 750 /var/securecorp/  # Apply to all files inside

CHOWN — Change File Owner:
─────────────────────────────────────────────────────────
# Change owner:
chown root /etc/shadow         # Owner = root
chown hareesh file.txt         # Owner = hareesh

# Change owner and group:
chown root:shadow /etc/shadow  # Owner=root, Group=shadow
chown www-data:www-data /var/www/html/

# Recursive:
chown -R www-data:www-data /var/www/html/

SECURITY COMMANDS — FIND RISKY PERMISSIONS:
─────────────────────────────────────────────────────────
# Find world-writable files (anyone can modify):
find / -perm -002 -type f 2>/dev/null
# These files can be modified by any user
# Attackers use these to plant backdoors

# Find world-writable directories:
find / -perm -002 -type d 2>/dev/null
# Attackers drop malicious files in these directories

# Find files with no owner (orphaned):
find / -nouser -o -nogroup 2>/dev/null
# May indicate deleted user left files behind
# Or attacker created files outside normal user context

# Find recently modified files in /etc/:
find /etc -mtime -1 2>/dev/null
# Files modified in last 24 hours
# Was config changed legitimately?
```

### Special Permissions — SUID, SGID, Sticky Bit

```
THESE ARE THE MOST SECURITY-CRITICAL PERMISSIONS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SUID — Set User ID (runs as file OWNER not as executor):
─────────────────────────────────────────────────────────
Normal execution:
  Gani runs a script → script runs as Gani

With SUID bit set:
  Gani runs the script → script runs as the FILE OWNER
  If owner is root → script runs as ROOT
  Even though GANI ran it!

REAL EXAMPLE — /usr/bin/passwd:
  -rwsr-xr-x root root /usr/bin/passwd
    ↑ s = SUID bit

  When you run passwd to change your own password:
  Your user needs to write to /etc/shadow (root-only file)
  SUID makes passwd run as root → can write to shadow
  But passwd only allows changing YOUR OWN password
  (the program logic enforces this)

DANGER — SUID ON BASH OR OTHER SHELLS:
  If attacker plants: chmod 4755 /bin/bash
  Then ANY user running /bin/bash -p gets a ROOT SHELL
  
  This is one of the most common persistence techniques.

HOW ATTACKERS ABUSE SUID:
  Find any SUID binary:
  find / -perm -4000 -type f 2>/dev/null

  Look up on GTFOBins (gtfobins.github.io):
  Many legitimate SUID programs have known
  privilege escalation methods.
  
  Examples:
  /usr/bin/find with SUID → find . -exec /bin/bash -p \;
  /usr/bin/vim with SUID  → vim escapes to root shell
  /usr/bin/python3 SUID   → python3 -c "import os;os.system('/bin/bash')"

DEFENSIVE CHECK — FIND ALL SUID FILES:
  find / -perm -u=s -type f 2>/dev/null
  
  Compare output to a known-good baseline.
  Any NEW SUID file not on baseline = investigate immediately.
  
  LEGITIMATE SUID files on Ubuntu:
  /usr/bin/passwd
  /usr/bin/sudo
  /usr/bin/pkexec
  /bin/mount
  /bin/umount
  /bin/su
  Any others = suspicious

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SGID — Set Group ID:
  Similar to SUID but runs as the file's GROUP.
  Used for shared directories where group access is needed.
  Less dangerous than SUID but still review.

STICKY BIT — Prevents deletion by non-owners:
  Used on /tmp/ and shared directories.
  Even if /tmp is world-writable:
  Gani can only delete HIS OWN files in /tmp
  Cannot delete Hareesh's files (sticky bit protection)

  ls -la /tmp
  drwxrwxrwt root root /tmp
            ↑ t = sticky bit

  Check /tmp is correctly set:
  stat /tmp | grep "Access:"
  Should show: (1777/drwxrwxrwt)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📌 SECTION 4 of 6
## Essential Linux Commands for SOC Analysts

---

### Navigation and File Operations

```
NAVIGATION:
─────────────────────────────────────────────────────────
pwd                  # Print current directory
                     # /home/navi

cd /var/log          # Change to /var/log
cd ..                # Go up one level
cd ~                 # Go to home directory
cd -                 # Go to previous directory

ls                   # List files
ls -la               # List with permissions + hidden files
ls -lh               # Human-readable file sizes
ls -lt               # Sort by modification time (newest first)
ls -ltr              # Sort by time reversed (oldest first)

VIEWING FILES:
─────────────────────────────────────────────────────────
cat /etc/passwd           # Display entire file
cat -n /etc/passwd        # With line numbers

head -20 /var/log/auth.log  # First 20 lines
tail -20 /var/log/auth.log  # Last 20 lines
tail -f /var/log/auth.log   # Follow in real time (live)
                             # Navi uses this to watch
                             # logs as events happen

less /var/log/syslog      # Scroll through file
                           # q = quit, /search = search
                           # n = next match, G = end

SEARCHING IN FILES:
─────────────────────────────────────────────────────────
grep "Failed" /var/log/auth.log     # Find failed logins
grep -i "error" /var/log/syslog     # Case-insensitive
grep -n "sudo" /var/log/auth.log    # Show line numbers
grep -v "nologin" /etc/passwd       # Lines NOT containing
grep -r "password" /etc/           # Recursive search
grep -E "Failed|Invalid" auth.log  # Multiple patterns

# Count matching lines:
grep -c "Failed" /var/log/auth.log

# Show lines before and after match (context):
grep -A 3 "Failed" auth.log  # 3 lines After match
grep -B 3 "Failed" auth.log  # 3 lines Before match
grep -C 3 "Failed" auth.log  # 3 lines around match

AWK — Process structured text:
─────────────────────────────────────────────────────────
# Print specific field from /etc/passwd:
awk -F: '{print $1}' /etc/passwd     # Just usernames
awk -F: '{print $1,$3}' /etc/passwd  # Username + UID
awk -F: '($3 == 0)' /etc/passwd      # Find UID 0 accounts
awk -F: '($7 !~ /nologin|false/)' /etc/passwd  # Login shells

# Count occurrences:
awk '{print $1}' /var/log/auth.log | sort | uniq -c | sort -rn
# Shows most frequent source IPs in logs
```

### Process and Service Management

```
VIEWING PROCESSES:
─────────────────────────────────────────────────────────
ps aux                    # All running processes
ps aux | grep apache      # Find specific process
ps aux | grep -v grep     # Exclude grep itself

top                       # Interactive process monitor
                          # q = quit
                          # k = kill a process
                          # 1 = show individual CPUs

htop                      # Better version of top
                          # Install: apt install htop

# Show process tree (parent-child relationships):
pstree -p
pstree -u                 # Show which user runs each process

# Security use: Find unexpected processes:
ps aux | awk '$11 !~ /^\[/'  # Exclude kernel threads
# Look for: Unknown process names, processes in /tmp
# Reverse shells often appear as /bin/sh or /tmp/backdoor

NETWORK CONNECTIONS:
─────────────────────────────────────────────────────────
ss -tulpn             # All listening ports + process names
                      # t=TCP, u=UDP, l=listening, p=process, n=numeric
netstat -tulpn        # Older equivalent (if ss not available)

ss -tnp               # All established TCP connections
ss -tnp | grep ESTAB  # Only established (active) connections

# Security use: Find unexpected network connections:
ss -tulpn | grep -v ":22\|:80\|:443\|:53"
# Shows ports that shouldn't be listening
# Reverse shell might show /tmp/backdoor listening on 4444

# Find which process owns a connection:
ss -tulpn | grep ":4444"
# If something is listening on 4444 → likely malware

SERVICES:
─────────────────────────────────────────────────────────
systemctl status apache2         # Check service status
systemctl list-units --type=service  # All services
systemctl list-units --type=service --state=running  # Active only
systemctl is-active apache2      # Just active/inactive

# Start/stop/restart:
systemctl start apache2
systemctl stop apache2
systemctl restart apache2
systemctl reload apache2  # Reload config without full restart

# Enable/disable on boot:
systemctl enable apache2    # Start on boot
systemctl disable apache2   # Don't start on boot

# Security: Find all enabled services:
systemctl list-unit-files --type=service --state=enabled
# Review each — is it necessary? Disable unused services.
```

### Package Management

```
UBUNTU/DEBIAN (SecureCorp's Linux):
─────────────────────────────────────────────────────────
apt update                    # Update package list
apt upgrade                   # Upgrade all packages
apt install nmap              # Install a package
apt remove nmap               # Remove package
apt purge nmap                # Remove + config files
apt list --installed          # All installed packages
apt list --upgradeable        # Packages needing update

# Security: Check for outdated packages:
apt list --upgradeable 2>/dev/null
# Any security updates? Apply immediately.

# Find when packages were installed (for forensics):
cat /var/log/dpkg.log | grep "install"
# Shows all package installations with timestamps
# Attacker installed something at 3 AM? Shows here.

# Check package integrity (detect tampering):
dpkg --verify
# Compares installed files against package checksums
# Modified system binaries will show up here
```

---

## 📌 SECTION 5 of 6
## Linux in SecureCorp — Real Scenarios

---

### Scenario 1 — Hareesh Checks WEB01 After an Alert

```
ALERT: Unusual outbound connection from WEB01 at 2 AM

Hareesh SSHes into WEB01:
ssh hareesh@192.168.1.40

STEP 1 — Check running processes:
ps aux --sort=-%cpu | head -20
# Any unknown process using high CPU?
# Is something in /tmp running?

ps aux | grep tmp
# If you see: /tmp/x or /tmp/.hidden/malware
# = Malware in /tmp — common attacker technique

STEP 2 — Check network connections:
ss -tnp | grep ESTAB
# What connections are currently active?

ss -tulpn | grep -v "22\|80\|443"
# Any unexpected listening ports?
# Apache should be on 80/443, SSH on 22
# Anything else = investigate

STEP 3 — Check who is logged in right now:
w
# Shows logged in users + what they're running
# Output:
# USER  TTY  FROM            LOGIN@  IDLE  WHAT
# hareesh pts/0 192.168.1.10 09:00   0:00  bash
# IF YOU SEE AN UNKNOWN USER = ACTIVE INTRUDER

who
last        # Recent login history
lastb       # Failed login attempts

STEP 4 — Check recent file changes:
find /var/www -mtime -1 -type f 2>/dev/null
# Files modified in web directory in last 24 hours
# Modified PHP file = possible webshell planted

find /tmp -type f 2>/dev/null
# Any files in /tmp?
# /tmp/.systemd-update or /tmp/x = suspicious

STEP 5 — Check scheduled tasks:
crontab -l              # Root's cron jobs
crontab -u hareesh -l   # Hareesh's cron jobs
ls -la /etc/cron*       # System-wide cron jobs
cat /etc/crontab
ls /etc/cron.d/

# Attacker persistence via cron:
# */5 * * * * /tmp/.update.sh
# = Runs /tmp/.update.sh every 5 minutes as root
# = Persistent reverse shell even after reboot

STEP 6 — Check authentication logs:
tail -100 /var/log/auth.log
grep "Failed" /var/log/auth.log | tail -20
grep "Accepted" /var/log/auth.log | tail -20
# Who successfully logged in recently?
# From which IP?
# At what time?
```

### Scenario 2 — Navi Investigates Suspicious SSH Activity

```
ALERT: Multiple SSH failures from external IP targeting WEB01

Navi opens /var/log/auth.log:

# Real-time monitoring:
tail -f /var/log/auth.log

# Sample suspicious output:
Apr 1 02:14:01 web01 sshd: Failed password for root from 185.220.101.45 port 54321
Apr 1 02:14:03 web01 sshd: Failed password for root from 185.220.101.45 port 54322
Apr 1 02:14:05 web01 sshd: Failed password for admin from 185.220.101.45 port 54323
Apr 1 02:14:07 web01 sshd: Failed password for ubuntu from 185.220.101.45 port 54324
Apr 1 02:14:09 web01 sshd: Failed password for gani from 185.220.101.45 port 54325
Apr 1 02:14:45 web01 sshd: Accepted password for gani from 185.220.101.45 port 54389
# ^^ SUCCESS after spray

NAVI'S ANALYSIS:
─────────────────────────────────────────────────────────
# Count failures per source IP:
grep "Failed" /var/log/auth.log | \
  awk '{print $(NF-3)}' | sort | uniq -c | sort -rn

# Output:
# 847  185.220.101.45   ← this IP = brute force
#   3  192.168.1.0      ← local network normal
#   1  10.0.0.1         ← single failure normal

# Find successful logins:
grep "Accepted" /var/log/auth.log
# Apr 1 02:14:45: gani logged in from 185.220.101.45
# SAME IP as attacker = password spray succeeded

# Check what Gani did after login:
grep "gani" /var/log/auth.log | grep -A 20 "02:14:45"

# Also check bash history (if attacker didn't clear it):
cat /home/gani/.bash_history
# Shows commands run in Gani's session

NAVI'S IMMEDIATE RESPONSE:
  1. Block 185.220.101.45 at firewall:
     iptables -A INPUT -s 185.220.101.45 -j DROP
     OR: ufw deny from 185.220.101.45

  2. Kill active session if still connected:
     who        # Find Gani's TTY (e.g. pts/1)
     pkill -u gani  # Kill all Gani's processes
     # OR:
     kill -9 [session PID]

  3. Lock Gani's account immediately:
     usermod -L gani   # Lock (-L adds ! to password)
     passwd -l gani    # Alternative method

  4. Force password reset (after investigation):
     passwd gani       # Set new password
     usermod -U gani   # Unlock account

  5. Check for persistence:
     crontab -u gani -l       # Did attacker add cron job?
     cat /home/gani/.ssh/authorized_keys
     # Did attacker add their SSH key for persistent access?
```

---

## 📌 SECTION 6 of 6
## Critical Linux Config Files for Security

---

### Complete Reference — Security-Critical Files

```
FILE: /etc/ssh/sshd_config
PURPOSE: SSH server configuration
SECURITY IMPORTANCE: Very High
─────────────────────────────────────────────────────────
# View it:
cat /etc/ssh/sshd_config

# KEY SETTINGS TO CHECK:

PermitRootLogin no        ← Root cannot SSH directly
                            (MUST be no in production)
                            If yes: attacker can brute
                            force root directly

PasswordAuthentication no ← Force key-based auth only
                            Much stronger than passwords
                            If yes + weak passwords = risk

PubkeyAuthentication yes  ← Allow SSH keys (good)

Port 22                   ← Default port, often targeted
                            Consider changing to 2222 or
                            random port (security by
                            obscurity — not a real defence
                            but reduces noise in logs)

MaxAuthTries 3            ← Lockout after 3 failed attempts
                            Default is 6 — reduce it

AllowUsers hareesh navi   ← Only these users can SSH
                            If not set: anyone with an
                            account can try to SSH

PermitEmptyPasswords no   ← Never allow empty password SSH
                            (should already be no)

X11Forwarding no          ← Disable X11 (rarely needed)
                            Reduces attack surface

# After changes, reload SSH:
systemctl reload sshd
# Do NOT restart — you might lose your current session

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FILE: /home/[user]/.ssh/authorized_keys
PURPOSE: SSH public keys allowed to login as this user
SECURITY IMPORTANCE: Critical
─────────────────────────────────────────────────────────
# Check all users' authorized_keys:
cat /home/hareesh/.ssh/authorized_keys
cat /home/gani/.ssh/authorized_keys
cat /root/.ssh/authorized_keys  # Root's SSH keys!

# What attackers do:
# Add their own public key here
# Gives permanent SSH access even after password change
# Very common persistence mechanism

# Format of authorized_keys:
# ssh-rsa AAAAB3NzaC1yc2E... attacker@evil.com
# Everything after the key = comment (shows who added it)

# Check for unauthorized keys:
find /home -name "authorized_keys" -exec cat {} \;
# Review every key — do you recognise all of them?
# Unknown key = attacker backdoor = remove immediately

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FILE: /etc/hosts
PURPOSE: Local DNS resolution
─────────────────────────────────────────────────────────
cat /etc/hosts

# Normal:
127.0.0.1   localhost
192.168.1.10 dc01.securecorp.local

# Suspicious:
192.168.1.99 updates.microsoft.com  ← DNS poisoning!
# Attacker redirects legitimate domains to their server
# Traffic to "Microsoft" goes to attacker's machine

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FILE: /proc/[pid]/
PURPOSE: Running process information
─────────────────────────────────────────────────────────
# Find suspicious process and inspect it:
ps aux | grep suspicious-name
# Note the PID (process ID)

# Read the exact command that started it:
cat /proc/[PID]/cmdline | tr '\0' ' '

# Read environment variables (may contain secrets):
cat /proc/[PID]/environ | tr '\0' '\n'

# See open files and network connections:
ls -la /proc/[PID]/fd     # Open file descriptors
cat /proc/[PID]/net/tcp   # Network connections

# Find deleted files still in use (malware technique):
ls -la /proc/*/exe 2>/dev/null | grep deleted
# If malware deletes itself but keeps running:
# /proc/12345/exe → /tmp/malware (deleted)
# The file is gone but process continues!
# Copy it out for analysis:
cp /proc/12345/exe /tmp/malware-sample

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FILE: /var/log/auth.log (Ubuntu) or /var/log/secure (RHEL)
PURPOSE: Authentication events
─────────────────────────────────────────────────────────
KEY PATTERNS TO LOOK FOR:

# Successful SSH logins (who and from where):
grep "Accepted" /var/log/auth.log
# Apr 1 09:00 web01 sshd: Accepted publickey for hareesh
# from 192.168.1.10 port 52341 ssh2

# Failed SSH attempts (brute force indicator):
grep "Failed password" /var/log/auth.log
grep "Invalid user" /var/log/auth.log  # Non-existent usernames

# Sudo usage (who escalated to root and when):
grep "sudo:" /var/log/auth.log
# Apr 1 10:30 web01 sudo: hareesh : TTY=pts/0 ;
# PWD=/home/hareesh ; USER=root ; COMMAND=/usr/bin/apt update
# Shows exactly what command was run as root

# Failed sudo attempts (privilege escalation attempts):
grep "sudo.*FAILED\|sudo.*incorrect password" /var/log/auth.log

# PAM authentication failures:
grep "pam_unix.*authentication failure" /var/log/auth.log

# New user sessions:
grep "session opened" /var/log/auth.log

# Su to another user:
grep "su\[" /var/log/auth.log
```

---

## 🧪 DAY 11 LABS

```
All labs on YOUR KALI LINUX — already installed!

LAB 1 — Explore the Linux Filesystem (20 min)
─────────────────────────────────────────────────────────
Open terminal on Kali and run each command:

# 1. View user accounts:
cat /etc/passwd
# How many users? Which have /bin/bash shell?
# Note your own username

# 2. Check groups:
cat /etc/group
# Are you in the sudo group?
grep sudo /etc/group

# 3. Check your own permissions:
id
# Shows your UID, GID, and group memberships

# 4. View file permissions:
ls -la /etc/passwd /etc/shadow /etc/sudoers
# Note the permissions on each file

# 5. Check shadow file (requires sudo on Kali):
sudo cat /etc/shadow
# Look at the hash format ($6$ = SHA-512)

LAB 2 — Process and Network Investigation (20 min)
─────────────────────────────────────────────────────────
# 1. See all running processes:
ps aux | head -20

# 2. Check listening services:
ss -tulpn
# What is listening on your Kali machine?

# 3. Check who is logged in:
w
who
last | head -10

# 4. Find SUID files:
find / -perm -u=s -type f 2>/dev/null
# List all SUID files
# Are there any in /tmp? (Suspicious!)

# 5. Check cron jobs:
crontab -l
cat /etc/crontab
ls /etc/cron.d/

LAB 3 — Log Analysis (20 min)
─────────────────────────────────────────────────────────
# 1. Check authentication log:
sudo tail -50 /var/log/auth.log
# Any SSH events? Sudo events?

# 2. Generate a log event — try sudo:
sudo ls /root
# Now check the log:
sudo grep "sudo" /var/log/auth.log | tail -5
# Can you see YOUR sudo command in the logs?

# 3. Check what packages were recently installed:
cat /var/log/dpkg.log | tail -20

# 4. Practice grep on logs:
sudo grep "Failed" /var/log/auth.log
sudo grep "Accepted" /var/log/auth.log
sudo grep "session opened" /var/log/auth.log

LAB 4 — Permission Exercises (20 min)
─────────────────────────────────────────────────────────
# Create test files and practice permissions:

# Create a test directory:
mkdir /tmp/security-lab
cd /tmp/security-lab

# Create test files:
echo "sensitive data" > secret.txt
echo "public data" > public.txt

# Check default permissions:
ls -la

# Make secret.txt only readable by owner:
chmod 600 secret.txt
ls -la secret.txt

# Make public.txt readable by everyone:
chmod 644 public.txt
ls -la public.txt

# Try to read as different concept:
# (check if chmod worked correctly)
sudo cat /etc/shadow   # Only root can read
cat /etc/passwd        # Anyone can read

# Practice finding world-writable files:
find /tmp -perm -002 -type f 2>/dev/null
```

---

## 📋 DAY 11 — COMPLETE NOTES

```
╔══════════════════════════════════════════════════════════╗
║         DAY 11 — LINUX SECURITY BASICS                  ║
║                  SecureCorp Ltd.                         ║
╚══════════════════════════════════════════════════════════╝

LINUX vs WINDOWS — KEY DIFFERENCES:
─────────────────────────────────────────────────────────
Everything is a file in Linux:
  Users → /etc/passwd
  Passwords → /etc/shadow
  Network → /etc/network/interfaces
  Services → /etc/ config directories
  Logs → /var/log/

Security benefit: Everything is auditable text
Security risk: Reading files = reading config

CRITICAL DIRECTORIES:
─────────────────────────────────────────────────────────
/etc/           All configuration files
/var/log/       ALL log files
/home/          User home directories
/root/          Root's home (root access only)
/tmp/           Temp files — world-writable — attacker fav
/proc/          Running processes (virtual, live)
/etc/cron*/     Scheduled tasks

CRITICAL FILES FOR SECURITY:
─────────────────────────────────────────────────────────
/etc/passwd    User accounts — anyone can read
               Fields: user:x:UID:GID:comment:home:shell
               Security: UID 0 = root, check for extras
                         /bin/bash = can login interactively

/etc/shadow    Password hashes — root only
               $6$ = SHA-512 (good)
               $1$ = MD5 (weak — rotate immediately)
               * or ! = account locked
               Empty = NO password (critical vulnerability)

/etc/group     Groups and members
               sudo group = root equivalent
               docker group = effective root
               Check for unexpected members

/etc/sudoers   Who can run root commands
               (via sudo visudo to edit)
               NOPASSWD: ALL = dangerous — no password needed
               Wildcard commands = shell escape risk
               Check /etc/sudoers.d/ for hidden entries

/etc/ssh/sshd_config   SSH server settings
               PermitRootLogin no (must be no)
               PasswordAuthentication no (use keys)
               MaxAuthTries 3 (reduce from default 6)
               AllowUsers [specific users only]

~/.ssh/authorized_keys  SSH public keys allowed in
               Attacker persistence: add their key here
               Check: any unknown keys = backdoor

PERMISSIONS:
─────────────────────────────────────────────────────────
Format: [type][owner][group][other]
Example: -rw-r----- = file, owner rw, group r, others none

r=4, w=2, x=1, -=0
644 = rw-r--r-- (normal files)
640 = rw-r----- (sensitive files like shadow)
755 = rwxr-xr-x (directories and executables)
600 = rw------- (SSH private keys — MUST be this)
777 = rwxrwxrwx (DANGEROUS — never for sensitive files)

Commands:
  chmod 640 /etc/shadow
  chown root:shadow /etc/shadow
  find / -perm -002 -type f   # World-writable files

SPECIAL PERMISSIONS:
─────────────────────────────────────────────────────────
SUID (s in owner execute position):
  File runs as OWNER not as executor
  If owner=root: ANY user runs it as root
  find / -perm -u=s -type f 2>/dev/null
  Known SUID should be: passwd, sudo, su, mount
  Unknown SUID = investigate immediately

SGID (s in group execute position):
  Runs as group owner
  Less dangerous than SUID

Sticky bit (t in other execute position):
  Only file owner can delete their own files
  Used on /tmp — check: stat /tmp → should be 1777

USERS AND IDENTITY:
─────────────────────────────────────────────────────────
id                    Your UID, GID, groups
whoami                Current username
w                     Who is logged in + what they're doing
who                   Simple logged-in list
last                  Recent login history
lastb                 Failed login attempts

User management:
  useradd gani         Create user
  usermod -L gani      Lock account (add ! to hash)
  usermod -U gani      Unlock account
  passwd gani          Change password
  userdel -r gani      Delete user + home directory

PROCESSES AND NETWORK:
─────────────────────────────────────────────────────────
ps aux               All processes (user, PID, command)
ps aux | grep suspicious  Find specific process
top / htop           Interactive process monitor
pstree -p            Process tree with PIDs

ss -tulpn            All listening ports + process
ss -tnp              Active connections
ss -tnp | grep ESTAB Only established connections

Attacker indicators:
  Process in /tmp     = malware
  Listening on unusual port = backdoor
  Deleted exe still running = stealth malware

ESSENTIAL LOG COMMANDS:
─────────────────────────────────────────────────────────
tail -f /var/log/auth.log          Real-time monitoring
grep "Failed" /var/log/auth.log    Find failures
grep "Accepted" /var/log/auth.log  Successful logins
grep "sudo:" /var/log/auth.log     Root escalations

# Count failed attempts per IP:
grep "Failed" /var/log/auth.log | \
  awk '{print $(NF-3)}' | sort | uniq -c | sort -rn

# Find brute force → success:
grep "Accepted" /var/log/auth.log
# Check if the IP appears in Failed logs too

SECURITY INVESTIGATION CHECKLIST:
─────────────────────────────────────────────────────────
When investigating suspicious Linux activity:
□ ps aux — any unknown processes?
□ ss -tulpn — any unexpected listening ports?
□ w / who — any unexpected active sessions?
□ last — suspicious recent logins?
□ find /tmp -type f — any files in /tmp?
□ find / -perm -u=s -type f — unexpected SUID files?
□ crontab -l / /etc/crontab — unexpected scheduled jobs?
□ cat ~/.ssh/authorized_keys — unknown SSH keys?
□ grep "Accepted" /var/log/auth.log — successful logins?
□ cat /etc/passwd — unknown user accounts?
□ awk -F: '($3==0)' /etc/passwd — multiple UID 0?

LABS COMPLETED:
─────────────────────────────────────────────────────────
□ Explored /etc/passwd, /etc/shadow, /etc/group
□ Checked sudo group membership
□ Viewed file permissions with ls -la
□ Found SUID files with find command
□ Checked cron jobs
□ Read auth.log and grep'd for events
□ Generated sudo event → verified in logs
□ Practiced chmod / permission changes
```

---

> 💡 **Architect's Truth for Day 11:** *"Every Windows admin I have trained who thought Linux was difficult had the same moment — they ran cat /etc/passwd and saw every user account in plain text. Then cat /etc/shadow and saw all the password hashes. Then cat /etc/sudoers and saw exactly who had root access. In Windows you need GUI tools and Active Directory to see this information. In Linux everything is a text file you can read with one command. Once that clicks — Linux stops being intimidating and starts being powerful. You now have that foundation. Day 12 we harden it."*

---

**Day 12 tomorrow — Linux Hardening.** We take WEB01 and harden it properly:
SSH hardening, firewall with UFW, fail2ban, disabling dangerous services, and CIS benchmark basics. Everything step by step with real commands. 💪🐧
