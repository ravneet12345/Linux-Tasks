# TASK 15 – MINI LINUX ADMINISTRATION PROJECT

## OBJECTIVE

The objective of this project is to apply the Linux administration concepts learned during Weeks 1 and 2 by provisioning and configuring an Ubuntu Linux server.

The project includes:

- Two Linux users
- Two Linux groups
- File and directory permissions
- Nginx web server
- Simple hosted webpage
- Custom systemd service
- Automated Cron job
- Backup automation
- SSH key authentication
- Persistence after system reboot

---

# ARCHITECTURE DIAGRAM

```text
                    +----------------------+
                    |   Administrator      |
                    |   SSH Client         |
                    +----------+-----------+
                               |
                         SSH Public Key
                               |
                               v
+------------------------------------------------------+
|                  Ubuntu Linux Server                 |
|                                                      |
|  +----------------+       +----------------------+   |
|  | alice          |       | bob                  |   |
|  | webadmins      |       | backupops            |   |
|  +-------+--------+       +----------+-----------+   |
|          |                           |               |
|          v                           v               |
|  +-------------------+      +--------------------+   |
|  | /var/www/task15   |      | Backup Management  |   |
|  +---------+---------+      +----------+---------+   |
|            |                           |             |
|            v                           v             |
|  +-------------------+      +--------------------+   |
|  | Nginx             |      | task15_backup.sh   |   |
|  | Port 80           |      +----------+---------+   |
|  +---------+---------+                 |             |
|            |                            v             |
|       Web Browser              /var/backups/task15   |
|                                                      |
|  +------------------------+                          |
|  | Custom systemd Service | ---> /var/log           |
|  +------------------------+                          |
|                                                      |
|  +------------------------+                          |
|  | Cron Scheduler         | ---> Backup Script      |
|  +------------------------+                          |
+------------------------------------------------------+
```

---

# 1. USER AND GROUP MANAGEMENT

## Create Groups

```bash
sudo groupadd webadmins
sudo groupadd backupops
```

## Create Users

```bash
sudo adduser alice
sudo adduser bob
```

## Assign Groups

```bash
sudo usermod -aG webadmins alice
sudo usermod -aG backupops bob
```

## Verification

```bash
id alice
id bob
getent group webadmins
getent group backupops
```

---

# 2. PERMISSIONS CONFIGURATION

The web directory was created:

```bash
sudo mkdir -p /var/www/task15
```

The Nginx user was added to the web administration group:

```bash
sudo usermod -aG webadmins www-data
```

Ownership was configured:

```bash
sudo chown -R root:webadmins /var/www/task15
```

Permissions were configured:

```bash
sudo chmod 2770 /var/www/task15
```

## Verification

```bash
ls -ld /var/www/task15
```

The SGID bit ensures that new files inherit the `webadmins` group.

---

# 3. NGINX INSTALLATION

Nginx was installed using:

```bash
sudo apt update
sudo apt install nginx -y
```

It was enabled to start automatically:

```bash
sudo systemctl enable --now nginx
```

## Verification

```bash
sudo systemctl status nginx
```

Expected status:

```text
Active: active (running)
```

---

# 4. WEB PAGE

The website was created at:

```text
/var/www/task15/index.html
```

Content:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Task 15 Linux Administration</title>
</head>
<body>
    <h1>Linux Administration Project</h1>
    <p>Nginx is running successfully.</p>
    <p>Task 15 implementation completed.</p>
</body>
</html>
```

File permissions were configured:

```bash
sudo chown root:webadmins /var/www/task15/index.html
sudo chmod 660 /var/www/task15/index.html
```

---

# 5. NGINX CONFIGURATION

Configuration file:

```text
/etc/nginx/sites-available/task15
```

Content:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name _;

    root /var/www/task15;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

The site was enabled:

```bash
sudo ln -s /etc/nginx/sites-available/task15 \
/etc/nginx/sites-enabled/task15
```

The default site was disabled:

```bash
sudo rm -f /etc/nginx/sites-enabled/default
```

Configuration was tested:

```bash
sudo nginx -t
```

Nginx was restarted:

```bash
sudo systemctl restart nginx
```

## Verification

```bash
curl http://localhost
```

The webpage was also verified using a browser.

---

# 6. CUSTOM SYSTEMD SERVICE

The following script was created:

```text
/usr/local/bin/task15_logger.sh
```

```bash
#!/bin/bash

LOG_FILE="/var/log/task15-service.log"

while true
do
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Task15 service is running" >> "$LOG_FILE"
    sleep 60
done
```

The script was made executable:

```bash
sudo chmod +x /usr/local/bin/task15_logger.sh
```

Service file:

```text
/etc/systemd/system/task15-logger.service
```

Content:

```ini
[Unit]
Description=Task15 Custom Logging Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/task15_logger.sh
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

The service was loaded and enabled:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now task15-logger.service
```

## Verification

```bash
sudo systemctl status task15-logger.service
sudo tail /var/log/task15-service.log
```

---

# 7. BACKUP SCRIPT

Backup directory:

```text
/var/backups/task15
```

The backup script was created at:

```text
/usr/local/bin/task15_backup.sh
```

Content:

```bash
#!/bin/bash

set -euo pipefail

SOURCE="/var/www/task15"
BACKUP_DIR="/var/backups/task15"
LOG_FILE="/var/log/task15-backup.log"

TIMESTAMP="$(date '+%Y%m%d_%H%M%S')"
BACKUP_FILE="$BACKUP_DIR/task15_$TIMESTAMP.tar.gz"

mkdir -p "$BACKUP_DIR"

echo "$(date '+%Y-%m-%d %H:%M:%S') - Backup started" >> "$LOG_FILE"

if tar -czf "$BACKUP_FILE" \
    -C "$(dirname "$SOURCE")" \
    "$(basename "$SOURCE")"
then
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Backup created: $BACKUP_FILE" \
        >> "$LOG_FILE"
else
    echo "$(date '+%Y-%m-%d %H:%M:%S') - ERROR: Backup failed" \
        >> "$LOG_FILE"
    exit 1
fi

find "$BACKUP_DIR" \
    -type f \
    -name 'task15_*.tar.gz' \
    -mtime +7 \
    -delete

echo "$(date '+%Y-%m-%d %H:%M:%S') - Cleanup completed" \
    >> "$LOG_FILE"
```

The script was made executable:

```bash
sudo chmod +x /usr/local/bin/task15_backup.sh
```

## Manual Verification

```bash
sudo /usr/local/bin/task15_backup.sh
sudo ls -lh /var/backups/task15
sudo cat /var/log/task15-backup.log
```

---

# 8. CRON CONFIGURATION

The Cron configuration was created at:

```text
/etc/cron.d/task15-backup
```

Content:

```cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

0 2 * * * root /usr/local/bin/task15_backup.sh
```

This runs the backup every day at 2:00 AM.

Permissions were configured:

```bash
sudo chmod 644 /etc/cron.d/task15-backup
```

Cron was enabled:

```bash
sudo systemctl enable --now cron
```

## Verification

```bash
sudo cat /etc/cron.d/task15-backup
sudo systemctl status cron
```

---

# 9. SSH KEY AUTHENTICATION

OpenSSH Server was installed:

```bash
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

An Ed25519 key pair was generated:

```bash
ssh-keygen \
-t ed25519 \
-f ~/.ssh/task15_alice_ed25519 \
-C "task15-alice"
```

Alice's SSH directory was created:

```bash
sudo install -d \
-m 700 \
-o alice \
-g alice \
/home/alice/.ssh
```

The public key was installed:

```bash
sudo install \
-m 600 \
-o alice \
-g alice \
~/.ssh/task15_alice_ed25519.pub \
/home/alice/.ssh/authorized_keys
```

## Verification

```bash
ssh -i ~/.ssh/task15_alice_ed25519 alice@localhost
```

After login:

```bash
whoami
```

Expected:

```text
alice
```

The private SSH key was never shared or committed to GitHub.

---

# 10. REBOOT PERSISTENCE

The server was rebooted:

```bash
sudo reboot
```

After reboot, the following checks were performed.

## Nginx

```bash
systemctl is-active nginx
systemctl is-enabled nginx
```

Expected:

```text
active
enabled
```

## Custom Service

```bash
systemctl is-active task15-logger.service
systemctl is-enabled task15-logger.service
```

Expected:

```text
active
enabled
```

## Cron

```bash
systemctl is-active cron
```

Expected:

```text
active
```

## Website

```bash
curl http://localhost
```

The web page was displayed successfully.

## SSH

```bash
ssh -i ~/.ssh/task15_alice_ed25519 alice@localhost
```

SSH key authentication continued to work after reboot.

---

# COMMANDS USED

```bash
sudo apt update
sudo groupadd webadmins
sudo groupadd backupops
sudo adduser alice
sudo adduser bob
sudo usermod -aG webadmins alice
sudo usermod -aG backupops bob
id alice
id bob

sudo mkdir -p /var/www/task15
sudo usermod -aG webadmins www-data
sudo chown -R root:webadmins /var/www/task15
sudo chmod 2770 /var/www/task15

sudo apt install nginx -y
sudo systemctl enable --now nginx
sudo nginx -t
sudo systemctl restart nginx
curl http://localhost

sudo systemctl daemon-reload
sudo systemctl enable --now task15-logger.service

sudo chmod +x /usr/local/bin/task15_backup.sh
sudo /usr/local/bin/task15_backup.sh

sudo systemctl enable --now cron

sudo apt install openssh-server -y
sudo systemctl enable --now ssh
ssh-keygen -t ed25519
ssh -i ~/.ssh/task15_alice_ed25519 alice@localhost

sudo reboot
```

---

# CONFIGURATION FILES

The important configuration files created or modified were:

```text
/etc/nginx/sites-available/task15
/etc/nginx/sites-enabled/task15
/etc/systemd/system/task15-logger.service
/etc/cron.d/task15-backup
/usr/local/bin/task15_logger.sh
/usr/local/bin/task15_backup.sh
/var/www/task15/index.html
/home/alice/.ssh/authorized_keys
```

---

# VERIFICATION SUMMARY

| Component | Verification | Expected result |
|---|---|---|
| Users | `id alice`, `id bob` | Users and groups displayed |
| Permissions | `ls -ld /var/www/task15` | Correct group and SGID permissions |
| Nginx | `systemctl is-active nginx` | `active` |
| Website | `curl http://localhost` | HTML displayed |
| systemd service | `systemctl is-active task15-logger.service` | `active` |
| Service log | `tail /var/log/task15-service.log` | New timestamps |
| Backup | `ls -lh /var/backups/task15` | `.tar.gz` file |
| Cron | `systemctl is-active cron` | `active` |
| SSH | `ssh -i KEY alice@localhost` | Successful login |
| Reboot | Repeat all checks | Everything remains operational |

---

# TROUBLESHOOTING NOTES

## Nginx Returns 403 Forbidden

Check:

```bash
ls -ld /var/www/task15
id www-data
```

Ensure `www-data` belongs to `webadmins`:

```bash
sudo usermod -aG webadmins www-data
sudo systemctl restart nginx
```

---

## Nginx Configuration Error

Run:

```bash
sudo nginx -t
```

Correct the configuration before restarting Nginx.

---

## Custom Service Does Not Start

Check:

```bash
sudo systemctl status task15-logger.service
sudo journalctl -u task15-logger.service
```

Check executable permission:

```bash
ls -l /usr/local/bin/task15_logger.sh
```

---

## Backup Fails

Check:

```bash
df -h
ls -ld /var/backups/task15
sudo cat /var/log/task15-backup.log
```

---

## Cron Does Not Execute

Check:

```bash
sudo systemctl status cron
sudo cat /etc/cron.d/task15-backup
sudo journalctl -u cron
```

Ensure:

```bash
sudo chmod 644 /etc/cron.d/task15-backup
```

---

## SSH Key Login Fails

Check:

```bash
sudo ls -ld /home/alice/.ssh
sudo ls -l /home/alice/.ssh/authorized_keys
```

Expected permissions:

```text
.ssh             700
authorized_keys  600
```

Correct using:

```bash
sudo chown -R alice:alice /home/alice/.ssh
sudo chmod 700 /home/alice/.ssh
sudo chmod 600 /home/alice/.ssh/authorized_keys
```

---

# LESSONS LEARNED

Through this project, I learned how to:

- Create and manage Linux users and groups
- Configure secure filesystem permissions
- Use SGID permissions for shared group directories
- Install and configure Nginx
- Host a web page on Linux
- Create and manage custom systemd services
- Configure automatic service startup
- Automate tasks using Cron
- Develop automated backup scripts
- Implement backup retention
- Configure SSH public-key authentication
- Verify Linux services after reboot
- Troubleshoot web, service, Cron, backup, permission, and SSH issues
- Integrate multiple Linux administration concepts into one working server

---

# SCREENSHOTS

Recommended screenshots:

```text
01-users-groups.png
02-permissions.png
03-nginx-status.png
04-nginx-config.png
05-webpage-browser.png
06-systemd-service.png
07-systemd-status.png
08-service-log.png
09-backup-script.png
10-backup-output.png
11-cron-config.png
12-cron-status.png
13-ssh-key.png
14-ssh-login.png
15-after-reboot-services.png
16-after-reboot-webpage.png
```

---

# FINAL RESULT

The Ubuntu server was successfully provisioned with:

```text
✓ Two users
✓ Two groups
✓ Controlled filesystem permissions
✓ Nginx web server
✓ Hosted webpage
✓ Custom systemd service
✓ Automated Cron job
✓ Automated backup solution
✓ SSH key authentication
✓ Automatic startup after reboot
```

The complete environment remained functional after a system reboot.
