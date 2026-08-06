# Task 13 – SSH Configuration and Security

## Objective

The objective of this task is to secure the OpenSSH server by:

- Disabling direct root login
- Configuring key-based authentication
- Disabling password authentication
- Changing the default SSH port
- Verifying successful login using the new configuration

---

## Environment

- Operating system: Ubuntu 22.04
- SSH server: OpenSSH
- Normal user: `ravneeth`
- New SSH port: `2222`
- Authentication method: Ed25519 public key

---

# Implementation

## 1. Install OpenSSH Server

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

The SSH service was verified as active before changing its configuration.

---

## 2. Generate an SSH Key

```bash
ssh-keygen -t ed25519 -C "ravneeth-task13"
```

The key files were created at:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

The private key was not shared or uploaded.

---

## 3. Install the Public Key

For a local virtual-machine test:

```bash
ssh-copy-id ravneeth@localhost
```

The key was stored in:

```text
~/.ssh/authorized_keys
```

Key permissions were secured:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

## 4. Test Key Authentication

Key-based login was tested before disabling password authentication:

```bash
ssh ravneeth@localhost
```

After successful login:

```bash
whoami
hostname
exit
```

---

## 5. Back Up the SSH Configuration

```bash
sudo cp /etc/ssh/sshd_config \
/etc/ssh/sshd_config.task13.backup
```

This backup can be restored if the new configuration causes a problem.

---

## 6. Allow the New Port

Port `2222` was checked:

```bash
sudo ss -tuln | grep ':2222'
```

If UFW was active, the new port was allowed:

```bash
sudo ufw allow 2222/tcp
sudo ufw status
```

---

## 7. Configure SSH Security

A dedicated SSH configuration file was created:

```bash
sudo nano /etc/ssh/sshd_config.d/99-task13.conf
```

The following directives were added:

```text
Port 2222
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
```

---

# Explanation of Directives

## `Port 2222`

Changes SSH from the default port `22` to port `2222`.

Changing the port reduces basic automated scanning and unwanted connection attempts. However, it does not replace key authentication, firewall rules, or other security controls.

## `PermitRootLogin no`

Prevents direct remote login using the root account.

Administrators must log in through their individual account and use `sudo`, which improves accountability and reduces attacks against the root username.

## `PubkeyAuthentication yes`

Enables SSH authentication using a public and private key pair.

The public key is stored on the server, while the private key remains on the client.

## `PasswordAuthentication no`

Disables SSH account-password login.

This prevents password guessing, credential stuffing, and attacks involving reused passwords.

## `KbdInteractiveAuthentication no`

Disables keyboard-interactive authentication so another password-like method cannot be used after password authentication is disabled.

---

## 8. Validate the Configuration

```bash
sudo sshd -t && echo "SSH configuration is valid"
```

The effective configuration was checked using:

```bash
sudo sshd -T | grep -E \
'^(port|permitrootlogin|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication) '
```

Expected values:

```text
port 2222
permitrootlogin no
pubkeyauthentication yes
passwordauthentication no
kbdinteractiveauthentication no
```

---

## 9. Apply the Configuration

```bash
sudo systemctl reload ssh
sudo systemctl status ssh --no-pager
```

The new port was verified:

```bash
sudo ss -tlnp | grep ':2222'
```

---

## 10. Verify Successful Login

A second terminal was opened while the original terminal remained open.

```bash
ssh -p 2222 ravneeth@localhost
```

After login:

```bash
whoami
hostname
```

The login succeeded using the normal user account, public-key authentication, and port `2222`.

---

## 11. Verify Password Authentication Is Disabled

```bash
ssh \
-o PubkeyAuthentication=no \
-o PreferredAuthentications=password \
-p 2222 \
ravneeth@localhost
```

Expected result:

```text
Permission denied (publickey).
```

---

## 12. Verify Root Login Is Disabled

```bash
ssh -p 2222 root@localhost
```

Expected result:

```text
Permission denied (publickey).
```

---

# Why These Changes Improve Security

| Change | Security benefit |
|---|---|
| Disable root login | Prevents direct attacks against the most privileged account. |
| Enable key authentication | Uses cryptographic credentials instead of reusable passwords. |
| Disable passwords | Prevents password guessing and credential-stuffing attacks. |
| Change SSH port | Reduces basic automated scanning and connection noise targeting port 22. |
| Validate configuration | Helps prevent applying syntax errors. |
| Keep the original session open | Provides a recovery path if the new connection fails. |

---

# Possible Risks of Incorrect Configuration

| Risk | Description |
|---|---|
| Remote lockout | Disabling password authentication before testing the key may prevent login. |
| Firewall lockout | The new SSH port may be blocked by UFW or an external firewall. |
| Syntax error | An invalid directive may prevent SSH from loading correctly. |
| Incorrect permissions | SSH may reject keys if `.ssh` or `authorized_keys` permissions are unsafe. |
| Lost private key | With password login disabled, losing the private key may prevent access. |
| Private-key exposure | A stolen private key could allow unauthorized access. |
| Port conflict | SSH cannot use port `2222` if another service already uses it. |
| Wrong file edited | Editing the client configuration instead of `sshd_config` will not secure incoming SSH access. |

---

# Commands Used

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
sudo systemctl status ssh
ssh-keygen -t ed25519 -C "ravneeth-task13"
ssh-copy-id ravneeth@localhost
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
ssh ravneeth@localhost
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.task13.backup
sudo ufw allow 2222/tcp
sudo nano /etc/ssh/sshd_config.d/99-task13.conf
sudo sshd -t
sudo sshd -T
sudo systemctl reload ssh
sudo ss -tlnp | grep ':2222'
ssh -p 2222 ravneeth@localhost
```

---

# Final Output

```text
SSH service                 : Active
SSH port                    : 2222
Root SSH login              : Disabled
Public-key authentication   : Enabled
Password authentication     : Disabled
Keyboard authentication     : Disabled
Login on new port           : Successful
```

---

# Screenshots

Store screenshots in:

```text
Week3/Screenshots/Task13/
```

Suggested screenshots:

1. `01-ssh-service-status.png`
2. `02-key-generation.png`
3. `03-public-key-installed.png`
4. `04-key-login-test.png`
5. `05-config-backup.png`
6. `06-hardening-config.png`
7. `07-config-validation.png`
8. `08-effective-settings.png`
9. `09-new-port-listening.png`
10. `10-successful-login.png`
11. `11-password-blocked.png`
12. `12-root-login-blocked.png`

---

# Learning Summary

Through this task, I learned how to:

- Install and manage the OpenSSH server
- Generate and install an Ed25519 SSH key
- Configure key-based authentication
- Disable root and password-based SSH access
- Change the SSH listening port
- Validate SSH configuration before applying changes
- Verify listening ports and authentication methods
- Avoid remote lockout by testing from a second terminal
- Restore SSH configuration from a backup
