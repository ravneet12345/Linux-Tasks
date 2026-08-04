# TASK 7 – CRON JOB SCHEDULING

## OBJECTIVE

The objective of this task is to learn Linux task scheduling using **Cron**. Cron is a time-based job scheduler that automatically executes commands or scripts at specified intervals without user intervention.

The following cron jobs are created:

- Every minute
- Every 15 minutes
- Every Monday
- On the first day of every month
- At system startup

---

# COMMANDS USED

## STEP 1 – OPEN THE CRONTAB

```bash
crontab -e
```

If prompted, select the nano editor.

---

## STEP 2 – CREATE A TEST DIRECTORY

```bash
mkdir -p ~/cron_logs
```

---

## STEP 3 – ADD THE FOLLOWING CRON ENTRIES

### Every Minute

```cron
* * * * * echo "Executed every minute at $(date)" >> ~/cron_logs/every_minute.log
```

---

### Every 15 Minutes

```cron
*/15 * * * * echo "Executed every 15 minutes at $(date)" >> ~/cron_logs/every_15_minutes.log
```

---

### Every Monday

```cron
0 9 * * 1 echo "Executed every Monday at $(date)" >> ~/cron_logs/monday.log
```

---

### First Day of Every Month

```cron
0 8 1 * * echo "Executed on the first day of the month at $(date)" >> ~/cron_logs/monthly.log
```

---

### At System Startup

```cron
@reboot echo "System started at $(date)" >> ~/cron_logs/startup.log
```

---

## STEP 4 – SAVE THE CRONTAB

Press

```
Ctrl + O
Enter
Ctrl + X
```

---

## STEP 5 – VERIFY CRON JOBS

Display all cron jobs:

```bash
crontab -l
```

Check the log files:

```bash
cat ~/cron_logs/every_minute.log
```

```bash
cat ~/cron_logs/every_15_minutes.log
```

```bash
cat ~/cron_logs/monday.log
```

```bash
cat ~/cron_logs/monthly.log
```

```bash
cat ~/cron_logs/startup.log
```

---

# EXPLANATION

## WHAT IS CRON?

Cron is a Linux daemon used to schedule commands or scripts to run automatically at specified times.

Each cron entry consists of **five time fields** followed by the command.

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of Week (0–7)
│ │ │ └──── Month (1–12)
│ │ └────── Day of Month (1–31)
│ └──────── Hour (0–23)
└────────── Minute (0–59)
```

---

# EXPLANATION OF EACH CRON ENTRY

---

## EVERY MINUTE

```cron
* * * * * echo "Executed every minute at $(date)" >> ~/cron_logs/every_minute.log
```

### Explanation

| Field | Value | Meaning |
|-------|-------|---------|
| Minute | * | Every minute |
| Hour | * | Every hour |
| Day of Month | * | Every day |
| Month | * | Every month |
| Day of Week | * | Every day of the week |

This job runs once every minute.

---

## EVERY 15 MINUTES

```cron
*/15 * * * * echo "Executed every 15 minutes at $(date)" >> ~/cron_logs/every_15_minutes.log
```

### Explanation

| Field | Value | Meaning |
|-------|-------|---------|
| Minute | */15 | Every 15 minutes |
| Hour | * | Every hour |
| Day of Month | * | Every day |
| Month | * | Every month |
| Day of Week | * | Every day |

The value `*/15` means:

```
0
15
30
45
```

The job runs four times every hour.

---

## EVERY MONDAY

```cron
0 9 * * 1 echo "Executed every Monday at $(date)" >> ~/cron_logs/monday.log
```

### Explanation

| Field | Value | Meaning |
|-------|-------|---------|
| Minute | 0 | At minute 0 |
| Hour | 9 | 9:00 AM |
| Day of Month | * | Every day |
| Month | * | Every month |
| Day of Week | 1 | Monday |

This job runs every Monday at **9:00 AM**.

---

## FIRST DAY OF EVERY MONTH

```cron
0 8 1 * * echo "Executed on the first day of the month at $(date)" >> ~/cron_logs/monthly.log
```

### Explanation

| Field | Value | Meaning |
|-------|-------|---------|
| Minute | 0 | At minute 0 |
| Hour | 8 | 8:00 AM |
| Day of Month | 1 | First day |
| Month | * | Every month |
| Day of Week | * | Any day |

The job executes once every month on the **1st day at 8:00 AM**.

---

## AT SYSTEM STARTUP

```cron
@reboot echo "System started at $(date)" >> ~/cron_logs/startup.log
```

### Explanation

`@reboot` is a special cron keyword.

Whenever the Linux system boots, this command runs automatically.

---

# SCREENSHOTS

Capture screenshots of the following:

1. Opening crontab (`crontab -e`)
2. All cron entries
3. Output of `crontab -l`
4. Every minute log
5. Every 15 minute log
6. Monday log
7. Monthly log
8. Startup log
9. Terminal after reboot showing startup log

---

# OUTPUT

Example:

```
Executed every minute at Tue Aug 4 10:20:01 IST 2026
Executed every minute at Tue Aug 4 10:21:01 IST 2026
```

Example:

```
Executed every 15 minutes at Tue Aug 4 10:30:01 IST 2026
```

Example:

```
System started at Tue Aug 4 10:15:10 IST 2026
```

Display the cron table:

```bash
crontab -l
```

Example output:

```
* * * * * echo "Executed every minute at $(date)" >> ~/cron_logs/every_minute.log
*/15 * * * * echo "Executed every 15 minutes at $(date)" >> ~/cron_logs/every_15_minutes.log
0 9 * * 1 echo "Executed every Monday at $(date)" >> ~/cron_logs/monday.log
0 8 1 * * echo "Executed on the first day of the month at $(date)" >> ~/cron_logs/monthly.log
@reboot echo "System started at $(date)" >> ~/cron_logs/startup.log
```

---

# ERRORS ENCOUNTERED

## Error 1

```
crontab: command not found
```

### Reason

Cron package was not installed.

### Resolution

```bash
sudo apt update
sudo apt install cron
sudo systemctl enable cron
sudo systemctl start cron
```

---

## Error 2

Cron job did not execute.

### Reason

Cron service was not running.

### Resolution

```bash
sudo systemctl status cron
```

Start the service if necessary:

```bash
sudo systemctl start cron
```

---

## Error 3

Log file not created.

### Reason

Directory did not exist.

### Resolution

```bash
mkdir -p ~/cron_logs
```

---

# LEARNING SUMMARY

Through this task, I learned how to:

- Use Cron for task scheduling.
- Edit the user's crontab using `crontab -e`.
- View scheduled jobs using `crontab -l`.
- Schedule tasks every minute.
- Schedule tasks every 15 minutes.
- Schedule weekly tasks.
- Schedule monthly tasks.
- Execute commands automatically at system startup.
- Verify cron jobs using log files.
- Troubleshoot common cron scheduling issues.

Cron is an essential Linux utility that automates repetitive administrative tasks, improving efficiency and reducing manual effort.
