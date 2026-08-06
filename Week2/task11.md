# Task 11 – Linux Troubleshooting

## Objective

Investigate and troubleshoot common Linux issues by identifying symptoms, investigating the root cause, resolving the issue, and verifying the solution.

---

# Scenario 1 – A Service Fails to Start

## Symptoms

- The Apache2 service failed to start.
- The web server was unavailable.
- The service status showed **failed** or **inactive**.

---

## Investigation Steps

1. Checked the service status.
2. Examined the system logs.
3. Verified the Apache configuration.
4. Restarted the service after confirming the configuration was correct.

---

## Commands Used

```bash
sudo systemctl status apache2
sudo journalctl -u apache2
sudo apachectl configtest
sudo systemctl restart apache2
sudo systemctl status apache2
```

---

## Root Cause

The Apache2 service was stopped unexpectedly. Configuration was verified and found to be correct.

---

## Resolution

Restarted the Apache2 service after confirming the configuration.

```bash
sudo systemctl restart apache2
```

---

## Verification

```bash
sudo systemctl status apache2
```

Expected Output

```text
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service)
     Active: active (running)
```

---

## Screenshots

- service-status-before.png
- service-logs.png
- service-running.png

---

# Scenario 2 – Disk Space Almost Full

## Symptoms

- The filesystem utilization exceeded 90%.
- Low disk space warning was displayed.
- System performance became slower.

---

## Investigation Steps

1. Checked overall disk usage.
2. Identified large directories.
3. Cleaned package cache.
4. Removed unused packages.
5. Cleared old journal logs.
6. Verified available disk space.

---

## Commands Used

```bash
df -h
sudo du -sh /*
sudo apt clean
sudo apt autoremove
sudo journalctl --vacuum-time=7d
df -h
```

---

## Root Cause

Disk space was occupied by:

- Package cache
- Unused packages
- Old system log files

---

## Resolution

Removed unnecessary files by cleaning package cache, removing unused packages, and deleting old journal logs.

```bash
sudo apt clean
sudo apt autoremove
sudo journalctl --vacuum-time=7d
```

---

## Verification

```bash
df -h
```

Expected Output

```text
Filesystem      Size  Used Avail Use%
/dev/sda3        50G   35G   15G  70%
```

---

## Screenshots

- disk-before.png
- large-directories.png
- disk-after.png

---

# Scenario 3 – CPU Usage Above 90%

## Symptoms

- CPU utilization remained above 90%.
- System became slow and unresponsive.
- Applications responded slowly.

---

## Investigation Steps

1. Monitored CPU utilization.
2. Identified the process consuming maximum CPU.
3. Terminated the unnecessary process.
4. Verified CPU utilization after stopping the process.

---

## Commands Used

```bash
top
htop
ps -eo pid,user,%cpu,%mem,command --sort=-%cpu | head
kill PID
```

Replace **PID** with the process ID obtained from the `ps` or `top` command.

---

## Root Cause

A process continuously consumed excessive CPU resources.

---

## Resolution

Stopped the unnecessary process using the **kill** command.

```bash
kill PID
```

or

```bash
kill -9 PID
```

---

## Verification

```bash
top
```

Expected Output

```text
CPU Usage: 15%
System Load: Normal
```

---

## Screenshots

- cpu-before.png
- high-cpu-process.png
- cpu-after.png

---

# Learning Summary

- Learned how to troubleshoot Linux services using **systemctl** and **journalctl**.
- Learned how to analyze disk usage using **df** and **du**.
- Learned how to reclaim disk space using **apt clean**, **apt autoremove**, and **journalctl**.
- Learned how to monitor CPU utilization using **top**, **htop**, and **ps**.
- Learned how to terminate unnecessary processes using the **kill** command.
- Understood the complete troubleshooting workflow:
  - Identify the symptoms
  - Investigate the issue
  - Determine the root cause
  - Apply the resolution
  - Verify the solution




```
