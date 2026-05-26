# Linux File Permissions — Cheat Sheet

## How to Read Permission Bits

```
-  r w x  r w x  r w x
^  ^^^    ^^^    ^^^
|   |      |      |
|   |      |      └── Others (everyone else)
|   |      └───────── Group
|   └──────────────── Owner (user)
└──────────────────── File type (- = file, d = directory, l = symlink)
```

## Permission Values (Octal)

| Symbol | Octal | Meaning |
|--------|-------|---------|
| `r` | 4 | Read |
| `w` | 2 | Write |
| `x` | 1 | Execute |
| `-` | 0 | No permission |

## Common Octal Combos

| Octal | Symbolic | Who can do what |
|-------|----------|-----------------|
| `777` | `rwxrwxrwx` | Everyone can read, write, execute — **dangerous** |
| `755` | `rwxr-xr-x` | Owner full access, others can read/run only |
| `750` | `rwxr-x---` | Owner full, group read/run, others nothing |
| `700` | `rwx------` | Owner only |
| `644` | `rw-r--r--` | Owner read/write, others read only |
| `600` | `rw-------` | Owner read/write only — good for private keys |
| `400` | `r--------` | Owner read only — good for backups |

## chmod Examples

```bash
chmod 755 script.sh         # standard script permissions
chmod 600 ~/.ssh/id_rsa     # SSH private key (must be this!)
chmod 644 index.html        # web file
chmod -R 750 /secure/dir    # recursive, secure directory
chmod u+x file              # add execute for owner
chmod g-w file              # remove write from group
chmod o= file               # remove all permissions from others
chmod a+r file              # add read for ALL (user/group/others)
```

## Special Permission Bits

```bash
chmod u+s file    # Set SUID  → file runs as owner
chmod g+s dir     # Set SGID  → new files inherit group
chmod +t dir      # Set Sticky Bit → only owner can delete files

# In octal, add a 4th digit at the front:
chmod 4755 file   # SUID + rwxr-xr-x
chmod 2750 dir    # SGID + rwxr-x---
chmod 1777 /tmp   # Sticky + rwxrwxrwx (how /tmp looks)
```

## Recognizing Special Bits in ls -la

```
drwxrwsr-t  root dev  shared/
       ^  ^
       |  └── Sticky Bit (t)
       └───── SGID (s in group execute position)

-rwsr-xr-x  root root  /usr/bin/passwd
   ^
   └── SUID (s in owner execute position)
```

## chown / chgrp

```bash
chown alice file              # change owner to alice
chown alice:dev file          # change owner AND group
chown -R alice:dev /folder    # recursive
chgrp dev file                # change group only
```

## Useful find Commands for Auditing

```bash
find / -perm 777 -type f              # world-writable files
find / -perm -o+w -type f             # files writable by others
find / -perm /4000 -type f            # SUID files
find / -perm /2000 -type f            # SGID files
find / -nouser 2>/dev/null            # orphaned files (no owner)
find / -nogroup 2>/dev/null           # files with no group
```

## The Principle of Least Privilege

> Give every user, process, and file only the minimum permissions needed to do its job — nothing more.

- Avoid `777` always
- Prefer `750` for directories shared by a group
- Use `600` or `400` for sensitive files (keys, configs, credentials)
- Regular audits catch permission drift before attackers do
