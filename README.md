# 🔐 Linux File Permissions Lab

> Hands-on security lab simulating real-world Linux user management, group policies, and file permission hardening.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Skills Demonstrated](#skills-demonstrated)
- [Lab Environment](#lab-environment)
- [Scenarios](#scenarios)
  - [Scenario 1 — User & Group Management](#scenario-1--user--group-management)
  - [Scenario 2 — File Permission Hardening](#scenario-2--file-permission-hardening)
  - [Scenario 3 — Permission Auditing](#scenario-3--permission-auditing)
  - [Scenario 4 — Special Permissions (SUID/SGID/Sticky Bit)](#scenario-4--special-permissions-suidsgidsticky-bit)
- [Key Commands Reference](#key-commands-reference)
- [Lessons Learned](#lessons-learned)
- [How to Reproduce This Lab](#how-to-reproduce-this-lab)

---

## Overview

This lab simulates a small company IT environment where a new sysadmin (me) is tasked with securing a Linux server. The server had poor permission hygiene — world-writable files, users with too many privileges, and no group structure.

The goal: **lock it down** following the principle of least privilege.

---

## Skills Demonstrated

| Skill | Tools Used |
|---|---|
| User creation and management | `useradd`, `usermod`, `userdel`, `passwd` |
| Group management | `groupadd`, `groupmod`, `gpasswd` |
| File permissions | `chmod`, `chown`, `chgrp` |
| Permission auditing | `find`, `ls -la`, `stat` |
| Special permissions | SUID, SGID, Sticky Bit |
| Reading permission bits | Symbolic (`rwx`) and octal (`755`) notation |

---

## Lab Environment

- **OS:** Ubuntu 22.04 LTS (can be reproduced on any Linux distro)
- **Setup:** Local VM or any cloud instance (AWS EC2 free tier works)
- **User:** Commands run as `root` or via `sudo`

---

## Scenarios

### Scenario 1 — User & Group Management

**Problem:** The company has 3 departments (dev, marketing, finance) but everyone shares the same user group. No separation of access.

**Solution:** Create department groups and assign users accordingly.

```bash
# Create department groups
sudo groupadd dev
sudo groupadd marketing
sudo groupadd finance

# Create users for each department
sudo useradd -m -s /bin/bash -G dev alice
sudo useradd -m -s /bin/bash -G dev bob
sudo useradd -m -s /bin/bash -G marketing carol
sudo useradd -m -s /bin/bash -G finance david

# Set passwords
sudo passwd alice
sudo passwd bob
sudo passwd carol
sudo passwd david

# Verify group memberships
cat /etc/group | grep -E "dev|marketing|finance"
id alice
```

**Result:**
```
dev:x:1001:alice,bob
marketing:x:1002:carol
finance:x:1003:david
```

---

### Scenario 2 — File Permission Hardening

**Problem:** Sensitive directories are world-readable. The `/finance/reports` folder is accessible by all users — a major security risk.

**Solution:** Apply strict permissions so only the finance group can read those files.

```bash
# Create the directory structure
sudo mkdir -p /company/dev
sudo mkdir -p /company/marketing
sudo mkdir -p /company/finance/reports

# Create some sample files
sudo touch /company/finance/reports/Q1_budget.txt
sudo touch /company/finance/reports/salaries.txt

# Assign ownership to correct groups
sudo chown -R root:finance /company/finance
sudo chown -R root:dev /company/dev
sudo chown -R root:marketing /company/marketing

# Set permissions
# Owner: read/write/execute | Group: read/execute | Others: NO ACCESS
sudo chmod -R 750 /company/finance
sudo chmod -R 750 /company/dev
sudo chmod -R 750 /company/marketing

# Verify
ls -la /company/
ls -la /company/finance/reports/
```

**Before vs After:**

| Directory | Before | After |
|---|---|---|
| `/company/finance` | `drwxrwxrwx` (777) | `drwxr-x---` (750) |
| `/company/dev` | `drwxrwxrwx` (777) | `drwxr-x---` (750) |
| `/company/marketing` | `drwxrwxrwx` (777) | `drwxr-x---` (750) |

**Test — verify access control works:**
```bash
# Switch to alice (dev user) and try to access finance folder
su - alice
ls /company/finance/reports/
# Expected output: Permission denied ✅
```

---

### Scenario 3 — Permission Auditing

**Problem:** We don't know what dangerous files exist on the system — world-writable files or files with no owner are a common attack vector.

**Solution:** Run an audit to find and fix dangerous permissions.

```bash
# Find all world-writable files (dangerous!)
sudo find / -type f -perm -o+w 2>/dev/null

# Find world-writable directories
sudo find / -type d -perm -o+w 2>/dev/null

# Find files with no owner (orphaned files — security risk)
sudo find / -nouser -o -nogroup 2>/dev/null

# Find files with SUID bit set (can be used for privilege escalation)
sudo find / -perm /4000 -type f 2>/dev/null

# Fix a world-writable file found during audit
sudo chmod o-w /path/to/dangerous/file

# Generate a full audit report and save it
sudo find / -type f -perm -o+w 2>/dev/null > audit_report.txt
echo "Audit completed: $(date)" >> audit_report.txt
cat audit_report.txt
```

**Sample audit output:**
```
/tmp/old_script.sh
/var/log/app/debug.log
/home/bob/shared_notes.txt

Audit completed: Mon Jan 1 10:00:00 UTC 2024
```

---

### Scenario 4 — Special Permissions (SUID/SGID/Sticky Bit)

**Problem:** The team needs a shared folder where all devs can write files, but no one can delete each other's files. Also, scripts run by any user should execute with the owner's privileges.

**Solution:** Use SGID on the shared folder and Sticky Bit for protection.

```bash
# Create a shared dev folder
sudo mkdir /company/dev/shared
sudo chown root:dev /company/dev/shared

# SGID: new files created inside inherit the group (dev)
# This means all files created here automatically belong to the dev group
sudo chmod g+s /company/dev/shared

# Sticky Bit: users can only delete their OWN files
# (Same behaviour as /tmp on any Linux system)
sudo chmod +t /company/dev/shared

# Final permissions should look like this:
ls -la /company/dev/
# drwxrwsr-t  root dev  shared

# Verify SGID is working
su - alice
touch /company/dev/shared/alice_notes.txt
ls -la /company/dev/shared/
# -rw-r--r-- alice dev alice_notes.txt  ← group is 'dev' automatically ✅

# Verify Sticky Bit: bob cannot delete alice's file
su - bob
rm /company/dev/shared/alice_notes.txt
# Expected: rm: cannot remove: Operation not permitted ✅
```

**Understanding the special bits:**

| Bit | Symbol | On Files | On Directories |
|---|---|---|---|
| SUID | `s` on owner execute | Runs as file owner | No effect |
| SGID | `s` on group execute | Runs as file group | New files inherit group |
| Sticky | `t` on other execute | No modern effect | Only owner can delete |

---

## Key Commands Reference

```bash
# --- PERMISSIONS ---
chmod 755 file          # rwxr-xr-x (octal)
chmod u+x file          # add execute for owner (symbolic)
chmod g-w file          # remove write for group
chmod o=r file          # set others to read-only
chmod -R 750 /folder    # recursive: apply to all contents

# --- OWNERSHIP ---
chown user file         # change owner
chown user:group file   # change owner and group
chown -R user:group /folder  # recursive
chgrp group file        # change group only

# --- READING PERMISSIONS ---
ls -la                  # long listing with permissions
stat file               # detailed file info
id username             # show user's groups

# --- FINDING FILES ---
find / -perm 777        # find files with exact permissions
find / -perm -o+w       # find world-writable files
find / -perm /4000      # find SUID files
find / -nouser          # find orphaned files

# --- USER MANAGEMENT ---
useradd -m -s /bin/bash username   # create user with home + shell
usermod -aG groupname username     # add user to group
userdel -r username                # delete user + home directory
passwd username                    # set password

# --- GROUP MANAGEMENT ---
groupadd groupname      # create group
groupdel groupname      # delete group
gpasswd -a user group   # add user to group
gpasswd -d user group   # remove user from group
```

---

## Lessons Learned

1. **Least privilege is non-negotiable.** Files should be accessible only to those who need them. Default `777` permissions are almost always wrong.

2. **World-writable files are a major risk.** Any user (or malicious process) can modify them. Regular audits with `find` catch these before attackers do.

3. **Groups are your best friend.** Instead of managing permissions per user, group users by role and assign permissions to the group. Scales much better.

4. **SGID on shared directories solves collaboration problems.** Without it, files in a shared folder inherit the creator's primary group — which breaks group-based access.

5. **Sticky Bit prevents accidental (or malicious) deletion.** Essential for any shared workspace.

6. **Orphaned files are sneaky.** When a user is deleted without `-r`, their files remain on disk with no owner. These are invisible to normal permission checks and can be exploited.

---

## How to Reproduce This Lab

You can run this entire lab on any Linux machine:

**Option A — Local VM (recommended)**
1. Download [Ubuntu Server 22.04](https://ubuntu.com/download/server)
2. Run it in VirtualBox or VMware (both free)
3. Clone this repo and run the scripts in order

**Option B — Free cloud instance**
1. Create a free AWS account
2. Launch a t2.micro EC2 instance with Ubuntu
3. SSH in and run the scripts

```bash
# Clone this repo
git clone https://github.com/YOUR_USERNAME/linux-permissions-lab
cd linux-permissions-lab

# Run scenarios in order
bash scripts/01_user_setup.sh
bash scripts/02_permission_hardening.sh
bash scripts/03_audit.sh
bash scripts/04_special_permissions.sh
```

---

*This lab was built as part of my cybersecurity and Linux administration portfolio. All scenarios are based on real-world misconfigurations commonly found in enterprise environments.*
