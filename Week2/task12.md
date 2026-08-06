# Task 12 – Backup Automation

## Objective

The objective of this task is to create a reusable Bash script that accepts a directory as input, creates a compressed backup with a timestamped filename, stores the backup in a separate directory, deletes backups older than seven days, and records every operation in a log file.

The script also validates user input and handles errors gracefully.

---

# Requirements

The script:

- Accepts a directory as input
- Creates a compressed `.tar.gz` backup
- Appends the current timestamp to the filename
- Stores backups in a separate directory
- Automatically deletes backups older than seven days
- Logs backup operations
- Handles missing, invalid, and unreadable input

---

# Commands Used

## Create a Test Source Directory

```bash
mkdir -p ~/task12_source/documents
echo "Backup automation test file" > ~/task12_source/file1.txt
echo "Linux backup task" > ~/task12_source/documents/file2.txt
```

## Create the Backup Directory

```bash
mkdir -p ~/task12_backups
```

## Create the Script

```bash
nano ~/backup_automation.sh
```

## Make the Script Executable

```bash
chmod +x ~/backup_automation.sh
```

## Run the Script

```bash
~/backup_automation.sh ~/task12_source
```

## Verify the Backup

```bash
ls -lh ~/task12_backups
```

## View Archive Contents

```bash
tar -tzf "$(ls -t ~/task12_backups/*.tar.gz | head -n 1)"
```

## View the Log

```bash
cat ~/task12_backups/backup.log
```

---

# Script

```bash
#!/bin/bash

set -o pipefail

BACKUP_DIR="$HOME/task12_backups"
LOG_FILE="$BACKUP_DIR/backup.log"

log_message() {
    local message="$1"
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $message" | tee -a "$LOG_FILE"
}

if [ "$#" -ne 1 ]; then
    echo "Error: Please provide exactly one source directory."
    echo "Usage: $0 /path/to/source_directory"
    exit 1
fi

SOURCE_DIR="$1"

if [ ! -e "$SOURCE_DIR" ]; then
    echo "Error: The path '$SOURCE_DIR' does not exist."
    exit 2
fi

if [ ! -d "$SOURCE_DIR" ]; then
    echo "Error: '$SOURCE_DIR' is not a directory."
    exit 3
fi

if [ ! -r "$SOURCE_DIR" ]; then
    echo "Error: The directory '$SOURCE_DIR' is not readable."
    exit 4
fi

if ! mkdir -p "$BACKUP_DIR"; then
    echo "Error: Unable to create backup directory '$BACKUP_DIR'."
    exit 5
fi

SOURCE_DIR="$(realpath "$SOURCE_DIR")"
SOURCE_NAME="$(basename "$SOURCE_DIR")"
TIMESTAMP="$(date '+%Y%m%d_%H%M%S')"
BACKUP_FILE="$BACKUP_DIR/${SOURCE_NAME}_${TIMESTAMP}.tar.gz"

log_message "Backup started for '$SOURCE_DIR'."

if tar -czf "$BACKUP_FILE" \
    -C "$(dirname "$SOURCE_DIR")" \
    "$SOURCE_NAME"; then

    BACKUP_SIZE="$(du -h "$BACKUP_FILE" | cut -f1)"
    log_message "Backup completed successfully: '$BACKUP_FILE' ($BACKUP_SIZE)."
else
    log_message "ERROR: Backup creation failed for '$SOURCE_DIR'."
    rm -f "$BACKUP_FILE"
    exit 6
fi

DELETED_COUNT="$(
    find "$BACKUP_DIR" \
        -maxdepth 1 \
        -type f \
        -name '*.tar.gz' \
        -mtime +7 \
        -print \
        -delete |
    wc -l
)"

log_message "Removed $DELETED_COUNT backup file(s) older than seven days."

echo
echo "Backup operation completed."
echo "Backup file: $BACKUP_FILE"
echo "Log file: $LOG_FILE"

exit 0
```

---

# Explanation

## Input Handling

The source directory is supplied as the first command-line argument:

```bash
~/backup_automation.sh ~/task12_source
```

The `$#` variable checks how many arguments were supplied, while `$1` contains the directory path.

## Validation

The script checks whether:

- Exactly one argument was supplied
- The path exists
- The path is a directory
- The directory is readable
- The backup destination can be created

Each failure produces a clear error and a nonzero exit status.

## Timestamped Filename

The command:

```bash
date '+%Y%m%d_%H%M%S'
```

creates a timestamp such as:

```text
20260806_111510
```

The completed filename becomes:

```text
task12_source_20260806_111510.tar.gz
```

## Compressed Backup

The command:

```bash
tar -czf
```

creates a gzip-compressed tar archive.

The `-C` option makes the archive contain the source folder name instead of the complete absolute path.

## Logging

The `log_message` function records the date, time, and status of every operation in:

```text
~/task12_backups/backup.log
```

The `tee -a` command displays the message and appends it to the log.

## Seven-Day Retention

The command:

```bash
find "$BACKUP_DIR" -type f -name '*.tar.gz' -mtime +7 -delete
```

finds and deletes backup archives older than seven days.

Only `.tar.gz` files inside the backup directory are removed.

---

# Output

Example successful output:

```text
2026-08-06 11:15:10 - Backup started for '/home/ravneeth/task12_source'.
2026-08-06 11:15:10 - Backup completed successfully: '/home/ravneeth/task12_backups/task12_source_20260806_111510.tar.gz' (4.0K).
2026-08-06 11:15:10 - Removed 0 backup file(s) older than seven days.

Backup operation completed.
Backup file: /home/ravneeth/task12_backups/task12_source_20260806_111510.tar.gz
Log file: /home/ravneeth/task12_backups/backup.log
```

---

# Error Handling

## Missing Input

Command:

```bash
~/backup_automation.sh
```

Output:

```text
Error: Please provide exactly one source directory.
Usage: /home/ravneeth/backup_automation.sh /path/to/source_directory
```

## Invalid Path

Command:

```bash
~/backup_automation.sh ~/directory_that_does_not_exist
```

Output:

```text
Error: The path '/home/ravneeth/directory_that_does_not_exist' does not exist.
```

## File Instead of Directory

Command:

```bash
~/backup_automation.sh /etc/hosts
```

Output:

```text
Error: '/etc/hosts' is not a directory.
```

---

# Verification

The backup directory was checked using:

```bash
ls -lh ~/task12_backups
```

The archive contents were verified using:

```bash
tar -tzf "$(ls -t ~/task12_backups/*.tar.gz | head -n 1)"
```

The log file was checked using:

```bash
cat ~/task12_backups/backup.log
```

The old-backup cleanup was tested by creating a test archive with a modification time older than seven days.

---

# Screenshots

The following screenshots were captured:

1. Source directory
2. Backup directory
3. Script permissions
4. Script content
5. Successful backup
6. Backup file
7. Archive contents
8. Backup log
9. Missing-input error
10. Invalid-path error
11. File-input error
12. Old backup before cleanup
13. Old backup removed

Screenshots are stored in:

```text
Week3/Screenshots/Task12/
```

---

# Errors Encountered and Resolution

## Permission Denied

### Error

```text
bash: backup_automation.sh: Permission denied
```

### Resolution

```bash
chmod +x ~/backup_automation.sh
```

## Source Directory Not Found

### Error

```text
Error: The path does not exist.
```

### Resolution

Verified and corrected the source directory path.

## Archive Creation Failed

### Possible Reason

The source directory was unreadable or the backup destination did not have sufficient space.

### Resolution

Verified permissions and available disk space:

```bash
ls -ld ~/task12_source
df -h
```

---

# Learning Summary

Through this task, I learned how to:

- Accept command-line arguments in Bash
- Validate directories and permissions
- Create compressed backups using `tar`
- Generate timestamped filenames
- Store backups separately from source data
- Log successful and failed operations
- Delete old backups automatically using `find`
- Test file retention without waiting seven days
- Use exit codes and clear error messages
- Schedule recurring backups using Cron
