# TASK 9 – PACKAGE MANAGEMENT

## OBJECTIVE

The objective of this task is to understand Linux package management in Ubuntu. This task demonstrates how to install, verify, and completely remove software packages using the Advanced Package Tool (APT). It also explains the differences between **apt**, **dpkg**, and **snap**, and identifies the appropriate use cases for each package manager.

---

# COMMANDS USED

## Step 1 – Update the Package Repository

```bash
sudo apt update
```

---

## Step 2 – Install Nginx

```bash
sudo apt install nginx -y
```

Verify installation:

```bash
nginx -v
```

Check the service status:

```bash
sudo systemctl status nginx
```

---

## Step 3 – Install Git

```bash
sudo apt install git -y
```

Verify installation:

```bash
git --version
```

---

## Step 4 – Install Curl

```bash
sudo apt install curl -y
```

Verify installation:

```bash
curl --version
```

---

## Step 5 – Verify Installed Packages

```bash
dpkg -l | grep -E "nginx|git|curl"
```

---

## Step 6 – Remove Installed Packages

Remove packages:

```bash
sudo apt remove nginx git curl -y
```

Remove configuration files:

```bash
sudo apt purge nginx git curl -y
```

Remove unused dependencies:

```bash
sudo apt autoremove -y
```

Clean downloaded package files:

```bash
sudo apt clean
```

---

## Step 7 – Verify Cleanup

```bash
dpkg -l | grep -E "nginx|git|curl"
```

or

```bash
which nginx
which git
which curl
```

---

# EXPLANATION

## Installing Packages

The `apt install` command downloads the package and its dependencies from the Ubuntu repositories and installs them on the system.

Example:

```bash
sudo apt install nginx -y
```

The `-y` option automatically answers **Yes** to installation prompts.

---

## Verifying Installation

Package installation was verified using:

```bash
nginx -v
git --version
curl --version
```

The installed packages were also verified using:

```bash
dpkg -l | grep -E "nginx|git|curl"
```

This displays all installed packages that match the given names.

---

## Removing Packages

The packages were removed using:

```bash
sudo apt remove
```

Configuration files were deleted using:

```bash
sudo apt purge
```

Unused dependencies were removed using:

```bash
sudo apt autoremove
```

Downloaded package cache was cleaned using:

```bash
sudo apt clean
```

---

# DIFFERENCE BETWEEN apt, dpkg, AND snap

## apt

APT (Advanced Package Tool) is the default package manager for Ubuntu.

### Features

- Installs software from Ubuntu repositories.
- Automatically installs required dependencies.
- Updates packages.
- Removes packages.
- Easy to use.

### Example

```bash
sudo apt install git
```

### When to Use

Use **apt** for:

- Installing software from Ubuntu repositories.
- Updating installed software.
- Removing software.
- Daily package management.

---

## dpkg

DPKG is the low-level package manager used by Debian and Ubuntu.

### Features

- Installs local `.deb` packages.
- Removes packages.
- Lists installed packages.
- Does not automatically install dependencies.

### Example

```bash
sudo dpkg -i package.deb
```

### When to Use

Use **dpkg** when:

- Installing downloaded `.deb` files.
- Viewing installed packages.
- Checking package information.

---

## snap

Snap is a universal package management system developed by Canonical.

### Features

- Installs sandboxed applications.
- Supports automatic updates.
- Works across many Linux distributions.
- Includes all required dependencies.

### Example

```bash
sudo snap install postman
```

### When to Use

Use **snap** when:

- Installing the latest version of applications.
- Installing software not available in APT repositories.
- Using sandboxed applications.

---

# COMPARISON TABLE

| Feature | apt | dpkg | snap |
|----------|-----|------|------|
| Package Source | Ubuntu Repository | Local .deb File | Snap Store |
| Dependency Management | Automatic | Manual | Automatic |
| Updates | Manual using apt | Manual | Automatic |
| Internet Required | Yes | No (for local packages) | Yes |
| Package Format | .deb | .deb | .snap |
| Best Used For | Daily package management | Installing downloaded packages | Universal applications |

---

# OUTPUT

### Nginx Version

```text
nginx version: nginx/1.18.0
```

---

### Git Version

```text
git version 2.34.1
```

---

### Curl Version

```text
curl 7.81.0
```

---

### Installed Packages

```text
ii nginx
ii git
ii curl
```

---

### Cleanup Verification

```text
(No output)
```

This confirms that all packages have been removed successfully.

---

# SCREENSHOTS

Capture the following screenshots:

1. `sudo apt update`
2. Installing Nginx
3. `nginx -v`
4. `sudo systemctl status nginx`
5. `git --version`
6. `curl --version`
7. `dpkg -l | grep -E "nginx|git|curl"`
8. Package removal using `apt remove`
9. Cleanup verification

Store the screenshots in:

```text
Week2/
└── Screenshots/
    └── Task9/
```

Example file names:

```text
apt-update.png
nginx-install.png
nginx-status.png
git-version.png
curl-version.png
installed-packages.png
package-removal.png
cleanup.png
```

---

# ERRORS ENCOUNTERED

## Error 1

```text
E: Unable to locate package
```

### Reason

The package list was outdated.

### Resolution

```bash
sudo apt update
```

---

## Error 2

```text
Could not get lock /var/lib/dpkg/lock
```

### Reason

Another package management process was already running.

### Resolution

Wait for the process to finish or restart the system before trying again.

---

## Error 3

```text
dpkg was interrupted
```

### Resolution

Run:

```bash
sudo dpkg --configure -a
```

to repair the package database.

---

# LEARNING SUMMARY

Through this task, I learned how to:

- Update the Ubuntu package repository.
- Install software packages using **APT**.
- Verify installed software versions.
- Check installed packages using **dpkg**.
- Remove packages completely using **apt remove** and **apt purge**.
- Remove unused dependencies using **apt autoremove**.
- Clean the package cache using **apt clean**.
- Understand the differences between **apt**, **dpkg**, and **snap**.
- Identify the appropriate package manager for different software installation scenarios.

Ubuntu primarily uses **APT** for day-to-day package management, **DPKG** for managing local `.deb` packages, and **Snap** for installing universal sandboxed applications that receive automatic updates.
