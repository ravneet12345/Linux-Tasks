# Task 6 – Systemd Service Management

 OBJECTIVE 
 
The objective of this task is to create and manage a custom Linux service using **systemd**. The service continuously appends the current date and time to a log file, automatically restarts if it stops unexpectedly, and starts automatically whenever the Ubuntu system boots.

---

# Commands Used

## Step 1: Create the shell script

```bash
sudo nano /usr/local/bin/datetime_logger.sh
```

Script content:

```bash
#!/bin/bash

LOG_FILE="/var/log/datetime_logger.log"

while true
do
    echo "Service running at: $(date '+%Y-%m-%d %H:%M:%S')" >> "$LOG_FILE"
    sleep 10
done
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/datetime_logger.sh
```

Verify permissions:

```bash
ls -l /usr/local/bin/datetime_logger.sh
```

---

## Step 2: Test the script

```bash
sudo timeout 15 /usr/local/bin/datetime_logger.sh
```

View the log file:

```bash
sudo cat /var/log/datetime_logger.log
```

---

## Step 3: Create the systemd service

```bash
sudo nano /etc/systemd/system/datetime-logger.service
```

Service file:

```ini
[Unit]
Description=Custom Date and Time Logger Service
After=local-fs.target

[Service]
Type=simple
ExecStart=/usr/local/bin/datetime_logger.sh
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

---

## Step 4: Reload systemd

```bash
sudo systemctl daemon-reload
```

---

## Step 5: Enable and start the service

```bash
sudo systemctl enable --now datetime-logger.service
```

---

## Step 6: Verify the service

```bash
sudo systemctl status datetime-logger.service
```

```bash
systemctl is-enabled datetime-logger.service
```

```bash
systemctl is-active datetime-logger.service
```

```bash
sudo tail -n 10 /var/log/datetime_logger.log
```

```bash
sudo journalctl -u datetime-logger.service
```

---

## Step 7: Test automatic restart

```bash
sudo systemctl kill --signal=SIGKILL datetime-logger.service
```

```bash
systemctl show datetime-logger.service --property=NRestarts
```

---

# Explanation

### Shell Script

The shell script continuously writes the current date and time to the file `/var/log/datetime_logger.log` every 10 seconds.

- `#!/bin/bash` executes the script using the Bash shell.
- `while true` creates an infinite loop.
- `date` retrieves the current date and time.
- `>>` appends the output to the log file.
- `sleep 10` waits for 10 seconds before writing the next entry.

### Service File Directives

#### [Unit]

- **Description** – Provides a description of the service.
- **After=local-fs.target** – Starts the service after local file systems are mounted.

#### [Service]

- **Type=simple** – Runs the script as a simple foreground process.
- **ExecStart** – Specifies the shell script to execute.
- **Restart=always** – Restarts the service automatically if it stops unexpectedly.
- **RestartSec=5** – Waits five seconds before restarting the service.
- **StandardOutput=journal** – Sends normal output to the systemd journal.
- **StandardError=journal** – Sends error messages to the systemd journal.

#### [Install]

- **WantedBy=multi-user.target** – Starts the service automatically during normal system boot.

---

# Screenshots

Include the following screenshots:

1. Shell script (`datetime_logger.sh`)
2. Executable permissions (`ls -l`)
3. Manual log output (`cat /var/log/datetime_logger.log`)
4. Service file (`datetime-logger.service`)
5. `systemctl status`
6. `systemctl is-enabled`
7. Log file updates (`tail`)
8. Restart verification (`NRestarts`)
9. Service after reboot

---

# Output

### Sample log output

```text
Service running at: 2026-08-03 21:20:20
Service running at: 2026-08-03 21:20:30
Service running at: 2026-08-03 21:20:40
```

### Service status

```text
Loaded: loaded
Active: active (running)
```

### Boot status

```text
enabled
```

### Restart count

```text
NRestarts=1
```

---

# Errors Encountered

### Error 1

```
Permission denied
```

**Reason**

The shell script was not executable.

**Resolution**

Made the script executable using:

```bash
sudo chmod +x /usr/local/bin/datetime_logger.sh
```

---

### Error 2

```
Unit datetime-logger.service not found
```

**Reason**

The service file had not been created or systemd had not reloaded its configuration.

**Resolution**

```bash
sudo systemctl daemon-reload
```

---

### Error 3

```
Failed to start service
```

**Reason**

Incorrect path specified in `ExecStart`.

**Resolution**

Verified the script path:

```bash
which datetime_logger.sh
```

Updated the correct path in the service file.

---

# Learning Summary

Through this task, I learned how to:

- Create shell scripts in Ubuntu.
- Make shell scripts executable.
- Create and configure custom systemd services.
- Manage Linux services using `systemctl`.
- Enable services to start automatically after reboot.
- Configure automatic service restart using `Restart=always`.
- Verify services using `systemctl`, `journalctl`, and log files.
- Troubleshoot common service configuration errors.

The task provided practical knowledge of Linux service management and demonstrated how systemd can be used to automate background processes efficiently.
