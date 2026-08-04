# TASK 8 – BASH AUTOMATION

## OBJECTIVE

The objective of this task is to develop a reusable Bash script that analyses a directory. The script accepts a directory path as input, counts files and subdirectories, calculates the total directory size, finds the largest file, and saves all results to a timestamped report file.

The script also validates user input and displays meaningful error messages when invalid information is supplied.

---

# COMMANDS USED

## CREATE THE SCRIPT

```bash
sudo nano /usr/local/bin/directory_report.sh
```

## MAKE THE SCRIPT EXECUTABLE

```bash
sudo chmod +x /usr/local/bin/directory_report.sh
```

## VERIFY SCRIPT PERMISSIONS

```bash
ls -l /usr/local/bin/directory_report.sh
```

## CREATE A TEST DIRECTORY

```bash
mkdir -p ~/task8_test/{documents,images,backups}
```

## CREATE SAMPLE FILES

```bash
echo "Linux automation task" > ~/task8_test/readme.txt
echo "Sample document" > ~/task8_test/documents/document1.txt
echo "Another document" > ~/task8_test/documents/document2.txt
dd if=/dev/zero of=~/task8_test/images/image1.dat bs=1K count=100 status=none
dd if=/dev/zero of=~/task8_test/backups/backup.dat bs=1K count=500 status=none
```

## RUN THE SCRIPT

```bash
/usr/local/bin/directory_report.sh ~/task8_test
```

## DISPLAY THE REPORT

```bash
cat "$(ls -t directory_report_*.txt | head -n 1)"
```

---

# SCRIPT

```bash
#!/bin/bash

if [ "$#" -ne 1 ]; then
    echo "Error: Please provide exactly one directory path."
    echo "Usage: $0 /path/to/directory"
    exit 1
fi

INPUT_DIR="$1"

if [ ! -e "$INPUT_DIR" ]; then
    echo "Error: The path '$INPUT_DIR' does not exist."
    exit 2
fi

if [ ! -d "$INPUT_DIR" ]; then
    echo "Error: '$INPUT_DIR' is not a directory."
    exit 3
fi

if [ ! -r "$INPUT_DIR" ]; then
    echo "Error: The directory '$INPUT_DIR' is not readable."
    exit 4
fi

ABS_DIR="$(realpath "$INPUT_DIR")"
TIMESTAMP="$(date '+%Y%m%d_%H%M%S')"
REPORT_FILE="$PWD/directory_report_$TIMESTAMP.txt"

FILE_COUNT="$(find "$ABS_DIR" -type f 2>/dev/null | wc -l)"
DIR_COUNT="$(find "$ABS_DIR" -mindepth 1 -type d 2>/dev/null | wc -l)"
TOTAL_SIZE="$(du -sh "$ABS_DIR" 2>/dev/null | cut -f1)"

LARGEST_FILE_INFO="$(
    find "$ABS_DIR" -type f -printf '%s\t%p\n' 2>/dev/null |
    sort -nr |
    head -n 1
)"

if [ -n "$LARGEST_FILE_INFO" ]; then
    LARGEST_FILE_SIZE_BYTES="$(printf '%s\n' "$LARGEST_FILE_INFO" | cut -f1)"
    LARGEST_FILE_PATH="$(printf '%s\n' "$LARGEST_FILE_INFO" | cut -f2-)"
    LARGEST_FILE_SIZE="$(numfmt --to=iec-i --suffix=B "$LARGEST_FILE_SIZE_BYTES")"
else
    LARGEST_FILE_PATH="No regular files found"
    LARGEST_FILE_SIZE="Not applicable"
fi

{
    echo "=========================================="
    echo "          DIRECTORY ANALYSIS REPORT"
    echo "=========================================="
    echo "Generated on       : $(date)"
    echo "Directory analysed : $ABS_DIR"
    echo "Number of files    : $FILE_COUNT"
    echo "Number of folders  : $DIR_COUNT"
    echo "Total size         : $TOTAL_SIZE"
    echo "Largest file       : $LARGEST_FILE_PATH"
    echo "Largest file size  : $LARGEST_FILE_SIZE"
    echo "Report file        : $REPORT_FILE"
    echo "=========================================="
} | tee "$REPORT_FILE"

echo
echo "Success: Report saved to '$REPORT_FILE'."
```

---

# EXPLANATION

The script accepts one directory path through the `$1` command-line argument. It checks whether the input exists, whether it is a directory, and whether it is readable.

The `find` command counts regular files and subdirectories recursively. The `du -sh` command calculates the total disk usage. File sizes are sorted in descending order to identify the largest file.

The `tee` command displays the results in the terminal and saves them to a timestamped report file.

---

# OUTPUT

Example output:

```text
==========================================
          DIRECTORY ANALYSIS REPORT
==========================================
Generated on       : Tue Aug 4 12:45:10 IST 2026
Directory analysed : /home/user/task8_test
Number of files    : 5
Number of folders  : 3
Total size         : 616K
Largest file       : /home/user/task8_test/backups/backup.dat
Largest file size  : 500KiB
Report file        : /home/user/directory_report_20260804_124510.txt
==========================================

Success: Report saved successfully.
```

---

# SCREENSHOTS

The following screenshots were captured:

1. Script content
2. Executable permissions
3. Test directory structure
4. Successful script execution
5. Generated report file
6. Missing input error
7. Non-existent directory error
8. File supplied instead of a directory
9. Empty directory result

Screenshots are stored in:

```text
Week2/Screenshots/Task8/
```

---

# ERRORS ENCOUNTERED

## ERROR 1: PERMISSION DENIED

```text
bash: /usr/local/bin/directory_report.sh: Permission denied
```

### RESOLUTION

The script was made executable:

```bash
sudo chmod +x /usr/local/bin/directory_report.sh
```

## ERROR 2: DIRECTORY DOES NOT EXIST

```text
Error: The path does not exist.
```

### RESOLUTION

The supplied directory path was checked and corrected.

## ERROR 3: INPUT WAS NOT A DIRECTORY

```text
Error: '/etc/hosts' is not a directory.
```

### RESOLUTION

A valid directory path was supplied instead of a regular file.

## ERROR 4: REPORT FILE NOT FOUND

The report filename contains a timestamp, so the exact name changes on each run.

### RESOLUTION

The newest report was displayed using:

```bash
cat "$(ls -t directory_report_*.txt | head -n 1)"
```

---

# LEARNING SUMMARY

Through this task, I learned how to:

- Accept command-line arguments in Bash.
- Validate whether input exists and is a directory.
- Check directory permissions.
- Count files and folders recursively using `find`.
- Calculate directory size using `du`.
- Sort file sizes and identify the largest file.
- Generate timestamped report files.
- Display and save output using `tee`.
- Use exit codes and meaningful error messages.
- Build reusable Bash automation scripts.
