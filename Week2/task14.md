# TASK 14 – LINUX SECURITY AND PERMISSIONS AUDIT

## OBJECTIVE

The objective of this task is to perform a basic Linux security and permissions audit by identifying:

- Files with SUID permission
- Files with SGID permission
- World-writable files
- Empty files
- Empty directories
- Broken symbolic links

The findings are reviewed to understand their security implications. No files are modified or deleted during the audit.

---

## ENVIRONMENT

- Operating system: Ubuntu
- Audit type: Read-only filesystem and permissions audit
- Audit location: Root filesystem
- Report directory: `~/task14-audit/reports`

---

# IMPLEMENTATION

## 1. Create the Audit Directory

```bash
mkdir -p ~/task14-audit/reports
cd ~/task14-audit
```

This directory stores the audit output separately from system files.

---

## 2. Identify SUID Files

```bash
sudo find / \
  -xdev \
  -type f \
  -perm -4000 \
  -printf '%M %u:%g %p\n' \
  2>/dev/null | tee reports/suid-files.txt
```

### Explanation

The SUID permission allows an executable to run with the privileges of its file owner rather than the user who started it.

The command options mean:

- `/` – start searching from the root filesystem
- `-xdev` – remain on the same filesystem
- `-type f` – search regular files
- `-perm -4000` – find files with SUID permission
- `-printf` – display permissions, owner, group, and path
- `2>/dev/null` – hide non-critical permission errors
- `tee` – display and save the output

### Security Implication

SUID files may be necessary for legitimate administrative operations. However, an unexpected or vulnerable root-owned SUID executable could allow privilege escalation.

SUID files should be compared with known Ubuntu packages and investigated if they are unfamiliar.

---

## 3. Identify SGID Files

```bash
sudo find / \
  -xdev \
  -type f \
  -perm -2000 \
  -printf '%M %u:%g %p\n' \
  2>/dev/null | tee reports/sgid-files.txt
```

### Explanation

The SGID permission causes an executable to run with the privileges of the file’s group.

- `-perm -2000` searches for SGID permission.

### Security Implication

SGID files may allow users to perform operations using additional group privileges. Unexpected SGID executables could expose resources belonging to a privileged group.

The owner, group, package source, and intended purpose should be verified.

---

## 4. Identify World-Writable Files

```bash
sudo find / \
  -xdev \
  -type f \
  -perm -0002 \
  -printf '%M %u:%g %p\n' \
  2>/dev/null | tee reports/world-writable-files.txt
```

### Explanation

A world-writable file can be modified by any local user.

- `-perm -0002` checks whether the write bit is set for “others.”

### Security Implication

World-writable files may allow unauthorized users to:

- Modify application data
- Alter configuration
- Replace scripts
- Inject malicious content
- Tamper with logs

Such permissions should be removed unless they are explicitly required.

Example permission correction:

```bash
sudo chmod o-w /path/to/file
```

Permissions must not be changed until the file’s purpose has been confirmed.

---

## 5. Identify Empty Files

```bash
sudo find / \
  -xdev \
  -type f \
  -empty \
  -printf '%M %u:%g %p\n' \
  2>/dev/null | tee reports/empty-files.txt
```

### Explanation

- `-type f` searches regular files.
- `-empty` finds files containing no data.

### Security Implication

Empty files are not automatically security vulnerabilities. They may be:

- Lock files
- Marker files
- Placeholder files
- Empty logs
- Incomplete configuration files
- Abandoned files

Each empty file must be reviewed before deletion.

---

## 6. Identify Empty Directories

```bash
sudo find / \
  -xdev \
  -type d \
  -empty \
  -printf '%M %u:%g %p\n' \
  2>/dev/null | tee reports/empty-directories.txt
```

### Explanation

- `-type d` searches directories.
- `-empty` identifies directories containing no entries.

### Security Implication

Empty directories may indicate abandoned software, incomplete removal, or unused storage paths. However, some applications require empty directories at startup.

The directory owner, package, and application requirements should be checked before removal.

---

## 7. Identify Broken Symbolic Links

```bash
sudo find / \
  -xtype l \
  -printf '%p -> %l\n' \
  2>/dev/null | tee reports/broken-symbolic-links.txt
```

### Explanation

A symbolic link is a reference to another file or directory.

- `-xtype l` identifies symbolic links whose target does not exist.
- `%p` prints the link path.
- `%l` prints its intended target.

### Security Implication

Broken symbolic links may indicate:

- Removed files
- Incomplete software uninstallations
- Incorrect paths
- Failed upgrades
- Missing mounted storage
- Stale application configuration

They may cause applications, backups, or automation scripts to fail.

A broken link should only be removed or repaired after confirming its expected target.

---

# AUDIT SUMMARY

The number of findings was calculated using:

```bash
echo "SUID files: $(wc -l < reports/suid-files.txt)"
echo "SGID files: $(wc -l < reports/sgid-files.txt)"
echo "World-writable files: $(wc -l < reports/world-writable-files.txt)"
echo "Empty files: $(wc -l < reports/empty-files.txt)"
echo "Empty directories: $(wc -l < reports/empty-directories.txt)"
echo "Broken symbolic links: $(wc -l < reports/broken-symbolic-links.txt)"
```

The summary was saved using:

```bash
{
  echo "Linux Security Audit Summary"
  echo "Generated: $(date)"
  echo
  echo "SUID files: $(wc -l < reports/suid-files.txt)"
  echo "SGID files: $(wc -l < reports/sgid-files.txt)"
  echo "World-writable files: $(wc -l < reports/world-writable-files.txt)"
  echo "Empty files: $(wc -l < reports/empty-files.txt)"
  echo "Empty directories: $(wc -l < reports/empty-directories.txt)"
  echo "Broken symbolic links: $(wc -l < reports/broken-symbolic-links.txt)"
} | tee reports/audit-summary.txt
```

Actual counts should be copied from the generated `audit-summary.txt` file.

---

# FINDINGS

| Category | Number found | Risk level | Observation |
|---|---:|---|---|
| SUID files | Add actual count | Medium/High | Review unexpected executables, especially root-owned files. |
| SGID files | Add actual count | Medium | Confirm group privileges and package ownership. |
| World-writable files | Add actual count | High if unexpected | Any local user may modify these files. |
| Empty files | Add actual count | Low/Informational | Verify whether they are required placeholders or stale files. |
| Empty directories | Add actual count | Low/Informational | Check whether applications or packages require them. |
| Broken symbolic links | Add actual count | Low/Medium | May indicate stale configuration or missing targets. |

The risk level depends on the actual file, owner, path, package, and intended purpose. A finding is not automatically a vulnerability.

---

# INVESTIGATION COMMANDS

The following commands can be used to inspect suspicious findings:

```bash
stat /path/to/file
```

Displays file type, permissions, ownership, timestamps, and size.

```bash
file /path/to/file
```

Identifies the file type.

```bash
dpkg -S /path/to/file
```

Identifies the Ubuntu package that owns the file.

```bash
readlink -f /path/to/link
```

Attempts to resolve a symbolic link target.

---

# SECURITY RECOMMENDATIONS

1. Review unfamiliar SUID and SGID files.
2. Confirm whether each privileged executable belongs to an installed Ubuntu package.
3. Remove world-write permission from files that do not require it.
4. Do not delete empty files or directories without confirming their purpose.
5. Repair or remove broken links only after identifying their expected targets.
6. Repeat the audit regularly.
7. Compare future results with a known-good baseline.
8. Investigate newly introduced privileged files immediately.
9. Keep Ubuntu and installed packages updated.
10. Store audit reports securely because filesystem paths may reveal system details.

---

# COMMANDS USED

```bash
mkdir -p ~/task14-audit/reports
cd ~/task14-audit

sudo find / -xdev -type f -perm -4000 \
-printf '%M %u:%g %p\n' 2>/dev/null \
| tee reports/suid-files.txt

sudo find / -xdev -type f -perm -2000 \
-printf '%M %u:%g %p\n' 2>/dev/null \
| tee reports/sgid-files.txt

sudo find / -xdev -type f -perm -0002 \
-printf '%M %u:%g %p\n' 2>/dev/null \
| tee reports/world-writable-files.txt

sudo find / -xdev -type f -empty \
-printf '%M %u:%g %p\n' 2>/dev/null \
| tee reports/empty-files.txt

sudo find / -xdev -type d -empty \
-printf '%M %u:%g %p\n' 2>/dev/null \
| tee reports/empty-directories.txt

sudo find / -xtype l \
-printf '%p -> %l\n' 2>/dev/null \
| tee reports/broken-symbolic-links.txt
```

---

# OUTPUT FILES

The audit created the following reports:

```text
reports/
├── audit-summary.txt
├── suid-files.txt
├── sgid-files.txt
├── world-writable-files.txt
├── empty-files.txt
├── empty-directories.txt
└── broken-symbolic-links.txt
```

---

# SCREENSHOTS

Screenshots are stored in:

```text
Week3/Screenshots/Task14/
```

Recommended screenshots:

1. `01-audit-directory.png`
2. `02-suid-files.png`
3. `03-sgid-files.png`
4. `04-world-writable-files.png`
5. `05-empty-files.png`
6. `06-empty-directories.png`
7. `07-broken-links.png`
8. `08-audit-summary.png`

---

# ERRORS ENCOUNTERED

## Permission Denied Messages

### Error

Some directories could not be inspected by a normal user.

### Resolution

The commands were run with `sudo`. Non-critical error output was redirected using:

```bash
2>/dev/null
```

## Large Amount of Output

### Problem

Some audit categories generated many results.

### Resolution

The output was saved to report files using `tee` and reviewed using:

```bash
less reports/filename.txt
```

## Broken Links in Virtual Filesystems

### Problem

Temporary system paths may change while the audit is running.

### Resolution

Results were reviewed individually and were not automatically deleted.

---

# LEARNING SUMMARY

Through this task, I learned how to:

- Identify SUID and SGID executables
- Find world-writable regular files
- Locate empty files and directories
- Detect broken symbolic links
- Save command output to audit reports
- Understand security implications of privileged permissions
- Distinguish findings from confirmed vulnerabilities
- Investigate ownership and package origin before remediation
- Conduct a read-only security audit without altering system files
