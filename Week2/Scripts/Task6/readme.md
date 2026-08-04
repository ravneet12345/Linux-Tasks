# EXPLANATION OF EVERY DIRECTIVE USED IN THE SERVICE FILE

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

## 1. [UNIT]

### DESCRIPTION=Custom Date and Time Logger Service

**PURPOSE**

Provides a human-readable description of the service.

**EXPLANATION**

- Displays a meaningful name for the service.
- Helps identify the service when using `systemctl status`.
- Used only for documentation and identification.
- Does not affect the execution of the service.

---

### AFTER=local-fs.target

**PURPOSE**

Starts the service only after the local file systems have been mounted.

**EXPLANATION**

- Ensures `/var/log` is available before the service starts.
- Controls the startup order of the service.
- Does not create a dependency; it only specifies the order.

---

## 2. [SERVICE]

### TYPE=simple

**PURPOSE**

Specifies that the service runs as a foreground process.

**EXPLANATION**

- Suitable for shell scripts that run continuously.
- Systemd considers the service started immediately after the script begins execution.

---

### EXECSTART=/usr/local/bin/datetime_logger.sh

**PURPOSE**

Specifies the command that systemd executes to start the service.

**EXPLANATION**

- Executes the shell script.
- Uses the full path to ensure systemd can locate the script.
- The script must have executable permission.

---

### RESTART=always

**PURPOSE**

Automatically restarts the service whenever it stops unexpectedly.

**EXPLANATION**

- Restarts the service if it crashes.
- Restarts the service if the process is killed.
- Improves service reliability and availability.

---

### RESTARTSEC=5

**PURPOSE**

Specifies the delay before restarting the service.

**EXPLANATION**

- Waits 5 seconds before restarting.
- Prevents continuous restart loops.
- Gives the system time to recover before restarting.

---

### STANDARDOUTPUT=journal

**PURPOSE**

Sends the normal output of the service to the systemd journal.

**EXPLANATION**

- Allows administrators to monitor service output.
- Output can be viewed using:

```bash
sudo journalctl -u datetime-logger.service
```

---

### STANDARDERROR=journal

**PURPOSE**

Sends error messages to the systemd journal.

**EXPLANATION**

- Records runtime errors.
- Simplifies troubleshooting.
- Error messages can be viewed using:

```bash
sudo journalctl -u datetime-logger.service
```

---

## 3. [INSTALL]

### WANTEDBY=multi-user.target

**PURPOSE**

Starts the service automatically during the normal system boot process.

**EXPLANATION**

- Enables automatic startup after reboot.
- Links the service to the `multi-user.target`.
- Activated when the following command is executed:

```bash
sudo systemctl enable datetime-logger.service
```

---

# SUMMARY

| DIRECTIVE | PURPOSE |
|-----------|---------|
| **DESCRIPTION** | PROVIDES A HUMAN-READABLE DESCRIPTION OF THE SERVICE. |
| **AFTER** | STARTS THE SERVICE AFTER THE LOCAL FILE SYSTEMS ARE MOUNTED. |
| **TYPE** | DEFINES THE SERVICE TYPE AS A FOREGROUND PROCESS. |
| **EXECSTART** | SPECIFIES THE SCRIPT TO EXECUTE. |
| **RESTART** | RESTARTS THE SERVICE AUTOMATICALLY IF IT STOPS. |
| **RESTARTSEC** | DEFINES THE DELAY BEFORE RESTARTING. |
| **STANDARDOUTPUT** | SENDS NORMAL OUTPUT TO THE SYSTEMD JOURNAL. |
| **STANDARDERROR** | SENDS ERROR OUTPUT TO THE SYSTEMD JOURNAL. |
| **WANTEDBY** | ENABLES THE SERVICE TO START AUTOMATICALLY AFTER REBOOT. |
