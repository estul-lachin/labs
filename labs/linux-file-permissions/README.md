# linux-file-permissions

## Concepts

A small print shop keeps its price list, order forms and invoice archive on one Linux server. The owner
wants each file readable by the right people only, and suspects the previous admin left a few things
open. Every file answers two questions: *who owns it* and *what may each class of user do with it*.

**Reading `ls -l`**: `-rwxr-x--- 1 pia printers ... price-list.txt`

- Character 1 is the type: `-` file, `d` directory, `l` symlink.
- Characters 2–4, 5–7 and 8–10 are the rights of the **owner**, the **group** and **others**.
  `r` read, `w` write, `x` execute (for a directory: may enter it). A `-` means not granted.
- Then the owner (`pia`) and the group (`printers`). A user who is neither gets the "others" rights.

**chmod, octal form**: r = 4, w = 2, x = 1, added up per class. `740` is owner `rwx` (4+2+1), group
`r--` (4), others `---` (0). The octal form **replaces** the whole mode.

**chmod, symbolic form**: `who` (`u` `g` `o` `a`), an operator (`+` add, `-` remove, `=` set exactly),
and the rights. It **only changes what I name**: `chmod g-w` removes group write and leaves every other
bit alone, which is why it's safer when I don't know the current mode.

**chown / chgrp**: `chown user file`, `chgrp group file`, or both at once with `chown user:group file`.
`-R` applies it to a whole directory tree. Giving files away needs root.

**Special bits** (a fourth octal digit in front):

| Bit | Octal | Shown as | Effect |
|---|---|---|---|
| SUID | 4000 | `s` in the owner's x slot | An executable runs as its **owner** (that's how `passwd` can edit /etc/shadow) |
| SGID | 2000 | `s` in the group's x slot | Runs as the file's group; on a directory, new files inherit its group |
| Sticky | 1000 | `t` in the others' x slot | In a directory, only a file's owner (or root) may delete or rename it, e.g. `/tmp` (`1777`) |

`find -perm /4000` matches files with *any* of those bits set (`/2000` SGID, `/1000` sticky).

**Why it matters for incident response**: a SUID copy of a shell in a writable directory is a root
backdoor; a world-writable cron file lets anyone run commands as root; a binary owned by an
unprivileged account can be swapped out by that account; a `/tmp` without the sticky bit lets users
delete each other's files.

## Tasks

Start the lab with `./lab up linux-file-permissions`, open a shell with `./lab shell linux-file-permissions`.
The shell is `root`. The users `pia` and `otto` (a member of group `printers`) and `guest` already exist.

1. **Create and inspect.** Create the empty file `/srv/print/price-list.txt` and look at it with
   `ls -l`. Write its permission string to `/root/answers.txt` as `INITIAL_PERMS=<like -rwx------>`.
2. **Octal mode.** Using numbers, give `price-list.txt` full rights for the owner, read-only for the
   group and nothing for others.
3. **Symbolic mode.** `/srv/print/order-form.txt` is currently `751`. Using the symbolic form, in one
   command: remove execute from the owner, set the group to exactly read+write, add read for others.
   Everything else must stay as it was.
4. **Ownership.** Make `price-list.txt` belong to user `pia` and group `printers` (try `chown` and
   `chgrp` separately). Then give the whole `/srv/print/archive` tree to `pia:printers` in one command.
   Afterwards `otto` should be able to read the price list and `guest` should not.
5. **Special bits.** Append three more lines to `/root/answers.txt`:

   ```
   PASSWD_SPECIAL=<suid, sgid or sticky: the special bit on /usr/bin/passwd>
   TMP_SPECIAL=<suid, sgid or sticky: the bit a normal /tmp carries>
   SGID_FILE=<the path of any file under /usr/bin that has the SGID bit>
   ```

6. **Permission audit.** Four things on this server are wrong. Find them and write
   `/root/findings.txt`:

   ```
   SUID_BACKDOOR=<a SUID file that shouldn't exist>
   WORLD_WRITABLE=<a world-writable file under /etc>
   BAD_OWNER=<a program in /usr/local/bin not owned by root>
   NO_STICKY=<a world-writable directory that is missing its sticky bit>
   ```

   Then fix them:
   - Remove the backdoor; the real SUID binaries must keep their bit.
   - Set the cron file to `644`.
   - Give the program to `root:root`.
   - Restore `/tmp` to `1777`.

Check your work from the host: `./lab check linux-file-permissions`

<details>
<summary>Hints</summary>

- `ls -l <file>`, `ls -ld <dir>`, `stat -c '%A %a %U:%G' <file>`
- `chmod 740 ...`, `chmod u-x,g=rw,o+r ...`
- `chown pia ...`, `chgrp printers ...`, `chown -R pia:printers ...`
- Test access as another user: `su -s /bin/sh otto -c 'cat /srv/print/price-list.txt'`
- `find / -xdev -perm /4000 -type f 2>/dev/null`, `find /etc /root -perm /002 -type f 2>/dev/null`,
  `find / -xdev -type d -perm -0002 2>/dev/null`

</details>

## My notes

<!-- Screenshots: ./lab shot linux-file-permissions ~/Screenshots/<file>.png <name>, then link it as
     ![what it shows](screenshots/<name>.png). Prefer code blocks for terminal output. -->

# My Notes – Linux File Permissions (Debian 12)

## `ls -l`

Check file and directory permissions.

```bash
ls -l filename
ls -ld directory
ls -la
```

Example:

```text
-rwxr-xr-- 1 root developers 4096 Nov 15 16:25 myfile
drwxr-xr-x 4 root root       4096 Nov 15 16:25 archive
```

Permission breakdown:

```text
-rwxr-xr--
 ||| ||| |||
 ||| ||| +++--- Others
 ||| +++------- Group
 +++----------- Owner
 |
 +------------- File type
```

- `-` = Regular file
- `d` = Directory
- `l` = Symbolic link
- `r` = Read
- `w` = Write
- `x` = Execute

For directories:
- `r` = List directory contents
- `w` = Create, delete or rename entries (normally requires `x` too)
- `x` = Enter/traverse the directory

## `stat -c '%A %a %U:%G' <file>`

Check the permission mode, owner and group.

```bash
stat -c '%A %a %U:%G' filename
```

Example:

```text
-rwxr----- 740 root:developers
```

- `%A` = Permissions in symbolic format
- `%a` = Permissions in numeric format
- `%U` = Owner username
- `%G` = Group name

Useful when I need to quickly verify file permissions and ownership.

## `chmod`

Change file or directory permissions.

```bash
chmod 755 filename
chmod 644 filename
chmod 700 filename
```

Numeric permission values:

```text
Read (r)    = 4
Write (w)   = 2
Execute (x) = 1
```

Add the values together:

| Number | Permission |
|---|---|
| 0 | `---` |
| 1 | `--x` |
| 2 | `-w-` |
| 3 | `-wx` |
| 4 | `r--` |
| 5 | `r-x` |
| 6 | `rw-` |
| 7 | `rwx` |

The three digits represent:

```text
Owner : Group : Others
```

Examples:

```bash
chmod 777 filename   # rwxrwxrwx
chmod 755 filename   # rwxr-xr-x
chmod 740 filename   # rwxr-----
chmod 644 filename   # rw-r--r--
chmod 600 filename   # rw-------
```

Common permission modes:

| Mode | Typical use |
|---|---|
| 644 | Normal files |
| 755 | Directories and executable scripts |
| 700 | Private directories |
| 600 | Private files / SSH private keys |
| 640 | Restricted configuration files |
| 777 | Everyone can read, write and execute (usually unsafe) |

Can also use symbolic permissions:

```bash
chmod u+x filename   # Add execute for owner
chmod g+w filename   # Add write for group
chmod o-r filename   # Remove read for others
chmod a+r filename   # Add read for everyone
```

Recursive permission changes:

```bash
chmod -R 755 folder
```

Be careful with `-R`, especially when changing permissions on system directories.

## `chown` and `chgrp`

Change file owner and group.

Change owner only:

```bash
chown user filename
```

Change group only:

```bash
chgrp group filename
```

Or:

```bash
chown :group filename
```

Change both owner and group:

```bash
chown user:group filename
```

Example:

```bash
chown pia:printers filename
```

Change ownership of a folder and everything under it:

```bash
chown -R user:group folder
```

Example:

```bash
chown -R pia:printers archive
```

Important:
- `chown` changes ownership, not permissions.
- `chmod` changes permissions, not ownership.
- Use `sudo` if root privileges are required.

Verify:

```bash
ls -l filename
stat -c '%U:%G %a %A' filename
```

## SUID vs SGID vs Sticky Bit

These are special permission bits in Linux.

### 1. SUID (Set User ID)

Numeric value: `4000`

When an executable has SUID set, it runs with the effective user ID of the file owner.

Example:

```bash
ls -l /usr/bin/passwd
```

Typical output:

```text
-rwsr-xr-x 1 root root 72056 May 30 2024 /usr/bin/passwd
```

```text
-rwsr-xr-x
   ^
   s = SUID
```

Why does `passwd` need SUID?

Normal users cannot directly modify `/etc/shadow`, but they need to be able to change their own passwords.

The SUID bit allows the `passwd` program to perform the required operation with the effective privileges of its owner (normally root).

### 2. SGID (Set Group ID)

Numeric value: `2000`

For executable files:
- The program runs with the effective group ID of the file's group.

For directories:
- New files and subdirectories normally inherit the directory's group.

Example:

```bash
mkdir shared
chown root:printers shared
chmod 2775 shared
```

Check:

```bash
ls -ld shared
```

Expected:

```text
drwxrwsr-x root printers shared
```

The `s` in the group execute position means SGID.

### 3. Sticky Bit

Numeric value: `1000`

Usually used on shared directories such as `/tmp`.

Example:

```bash
ls -ld /tmp
```

Normal output:

```text
drwxrwxrwt 1 root root 4096 Nov 15 16:25 /tmp
```

The `t` at the end indicates Sticky Bit.

It prevents ordinary users from deleting or renaming files belonging to other users in a shared directory (unless they own the directory or have sufficient privileges).

Set Sticky Bit:

```bash
chmod +t /tmp
```

Or set the full permission mode:

```bash
chmod 1777 /tmp
```

### Special permission summary

| Special bit | Numeric value | Symbol | Main purpose |
|---|---:|---|---|
| SUID | 4000 | `s` | Use file owner's effective identity |
| SGID | 2000 | `s` | Use file group's effective identity / directory group inheritance |
| Sticky | 1000 | `t` | Protect files in shared directories |

Set special permissions:

```bash
chmod 4755 filename   # SUID
chmod 2755 directory  # SGID
chmod 1777 directory  # Sticky Bit
```

Find files with special permissions:

```bash
find / -type f -perm -4000 2>/dev/null  # SUID
find / -type f -perm -2000 2>/dev/null  # SGID
find / -type d -perm -1000 2>/dev/null  # Sticky Bit
```

Note: If the execute permission is missing, special bits may appear as uppercase `S` or `T` instead of lowercase `s` or `t`.

## Permission Audit

Useful commands when checking for incorrect permissions or ownership.

### 1. Find unexpected SUID files

```bash
find / -type f -perm -4000 2>/dev/null
```

Not every SUID file is suspicious. Compare the results with expected system binaries.

### 2. Find world-writable files under `/etc`

```bash
find /etc -type f -perm -0002 -ls
```

`-0002` means others have write permission.

World-writable configuration files can be a security risk.

### 3. Find programs not owned by root

```bash
find /usr/local/bin -type f ! -user root -ls
```

Check whether executable files are owned by the expected user.

### 4. Find world-writable directories without Sticky Bit

```bash
find / -type d -perm -0002 ! -perm -1000 -ls 2>/dev/null
```

A normal `/tmp` should have Sticky Bit enabled.

### 5. Save audit findings

Example:

```bash
cat > /root/findings.txt <<'EOF'
SUID_BACKDOOR=/var/tmp/.helper
WORLD_WRITABLE=/etc/cron.d/cleanup
BAD_OWNER=/usr/local/bin/print-backup
NO_STICKY=/tmp
EOF
```

Check the result:

```bash
cat /root/findings.txt
```

## Permission Remediation

After completing a permission audit, the next step is to fix the issues found during the inspection.

The exact commands depend on the affected files and directories.

### Common remediation commands

```bash
# Remove a suspicious file
sudo rm <suspicious-file>

# Change file permissions
sudo chmod 644 <configuration-file>

# Change file ownership and group
sudo chown root:root <program-file>

# Restore standard shared directory permissions
sudo chmod 1777 <temporary-directory>
```

### Verify the changes

Always verify the permissions and ownership after making changes.

```bash
# Check file permissions
ls -l <file>

# Check directory permissions
ls -ld <directory>

# Check owner, group and numeric mode
stat -c '%A %a %U:%G' <file>

# Check whether a file still exists
test -e <file> && echo "File exists" || echo "File not found"
```

### Important notes

- Do not remove legitimate system SUID or SGID binaries.
- Always verify a suspicious file before deleting it.
- Avoid using recursive permission changes unless necessary.
- Keep system configuration files under the appropriate ownership.
- Be careful when changing permissions on shared directories.

**My takeaway:** Finding a permission issue is only the first step. Always verify the affected file, understand why the permission is incorrect, apply the smallest necessary change, and check the result afterward.

## Related

- [hello-world](../hello-world/): octal permissions on a single file
- [linux-server-setup](../linux-server-setup/): users, groups and a `770` shared directory
- `man chmod`, `man chown`, `man chgrp`, `man find` (section `-perm`)
