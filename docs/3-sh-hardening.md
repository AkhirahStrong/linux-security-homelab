# SSH Hardening

## Objective

The objective of this lab was to secure remote administration of my Ubuntu Server by configuring SSH public-key authentication and disabling password-based SSH authentication.

Rather than relying on an Ubuntu account password for remote access, the server now requires possession of an authorized SSH private key.

---

## Lab Environment

| Component      | Configuration                     |
| -------------- | --------------------------------- |
| Host System    | macOS                             |
| Virtualization | UTM / QEMU                        |
| Guest OS       | Ubuntu Server 22.04.5 LTS         |
| Architecture   | ARM64                             |
| Remote Access  | OpenSSH                           |
| SSH Port       | TCP 22                            |
| Authentication | ED25519 public-key authentication |
| Host Firewall  | UFW                               |

> IP addresses and usernames in this documentation may be generalized for security and privacy.

---

## 1. Identify Listening Services

Before modifying SSH, I inspected the server's listening TCP and UDP services.

```bash
sudo ss -tulpn
```

The options used were:

- `-t` — Display TCP sockets
- `-u` — Display UDP sockets
- `-l` — Display listening sockets
- `-p` — Display the process using the socket
- `-n` — Display numerical addresses and port numbers

The results showed that OpenSSH was listening on TCP port 22.

Example:

```text
tcp LISTEN 0 128 0.0.0.0:22 0.0.0.0:* users:(("sshd",...))
```

This established that SSH was available for remote administration.

---

## 2. Configure the Host Firewall

Ubuntu's Uncomplicated Firewall (UFW) was configured to deny unsolicited incoming connections while permitting SSH.

Before making firewall changes, SSH was explicitly allowed:

```bash
sudo ufw allow OpenSSH
```

I then verified the firewall configuration:

```bash
sudo ufw status verbose
```

The resulting policy showed:

```text
Status: active

Logging: on (low)

Default: deny (incoming), allow (outgoing), disabled (routed)
```

SSH was explicitly permitted:

```text
22/tcp (OpenSSH)       ALLOW IN    Anywhere
22/tcp (OpenSSH (v6))  ALLOW IN    Anywhere (v6)
```

### Security Reasoning

The default-deny incoming policy reduces the server's network attack surface.

Instead of allowing unsolicited incoming connections by default, services must be explicitly permitted through the firewall.

SSH was allowed because it is required for remote administration of the server.

---

## 3. Generate an SSH Key Pair

On the macOS administration system, I generated an ED25519 SSH key pair.

```bash
ssh-keygen -t ed25519
```

This created:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

The files serve different purposes:

- `id_ed25519` — Private key
- `id_ed25519.pub` — Public key

The private key remains on the administration system and should never be distributed.

I also protected the private key with a passphrase.

### Why ED25519?

ED25519 provides modern public-key authentication with strong security and relatively small key sizes.

---

## 4. Copy the Public Key to the Ubuntu Server

The public key was installed on the Ubuntu server using:

```bash
ssh-copy-id <user>@<server-ip>
```

The server initially requested the Ubuntu account password because password authentication was still enabled.

After successful authentication, the public key was added to the server.

The public key is stored for the remote user in:

```text
~/.ssh/authorized_keys
```

The private key remained on the macOS administration system.

---

## 5. Verify Key-Based Authentication

Before disabling password authentication, I tested SSH access from a separate terminal session.

```bash
ssh <user>@<server-ip>
```

The SSH client requested the passphrase protecting my private SSH key rather than the Ubuntu user's login password.

Successful access confirmed that public-key authentication was working.

### Why Test First?

Disabling password authentication before verifying public-key authentication could result in losing remote access to the server.

For this reason, the existing SSH session remained open while the new authentication method was tested.

---

## 6. Inspect the Effective SSH Configuration

I checked whether the SSH daemon still permitted password authentication.

```bash
sudo sshd -T | grep passwordauthentication
```

Initially, the result was:

```text
passwordauthentication yes
```

This confirmed that users could still authenticate remotely using their Ubuntu account passwords.

---

## 7. Locate the Password Authentication Configuration

The SSH configuration can be defined in multiple files.

I searched the main SSH configuration and its configuration directory:

```bash
sudo grep -Rni "PasswordAuthentication" /etc/ssh/sshd_config /etc/ssh/sshd_config.d/
```

The search identified:

```text
/etc/ssh/sshd_config.d/50-cloud-init.conf:1:PasswordAuthentication yes
```

This demonstrated an important configuration-management concept: changing a setting in the main `sshd_config` file does not necessarily mean it will become the effective SSH configuration.

Additional configuration files can influence the final configuration used by `sshd`.

---

## 8. Create an SSH Hardening Configuration

Rather than relying solely on the cloud-init generated configuration, I created a dedicated SSH hardening configuration:

```text
/etc/ssh/sshd_config.d/00-hardening.conf
```

The configuration contains:

```text
PasswordAuthentication no
```

It was created using:

```bash
echo "PasswordAuthentication no" | sudo tee /etc/ssh/sshd_config.d/00-hardening.conf
```

This disables password-based SSH authentication.

---

## 9. Validate the SSH Configuration

Before applying the new configuration, I checked it for syntax errors:

```bash
sudo sshd -t
```

The command returned no errors.

A successful validation was important because restarting or reloading SSH with an invalid configuration could interfere with remote administration.

---

## 10. Apply the Configuration

After validation, the SSH service was reloaded:

```bash
sudo systemctl reload ssh
```

The effective SSH configuration was then checked again:

```bash
sudo sshd -T | grep passwordauthentication
```

The result was:

```text
passwordauthentication no
```

This confirmed that password authentication was disabled.

---

## 11. Test Password Authentication

Configuration settings should be verified through testing rather than assumed to be working.

I deliberately attempted an SSH connection without allowing public-key authentication:

```bash
ssh -o PubkeyAuthentication=no <user>@<server-ip>
```

The server rejected the connection:

```text
Permission denied (publickey).
```

This verified that password-based SSH authentication was no longer accepted.

A normal connection using the authorized SSH key continued to work:

```bash
ssh <user>@<server-ip>
```

---

## Final Security Configuration

The SSH configuration now follows this authentication model:

```text
                 SSH CONNECTION
                       |
                       v
              +------------------+
              |   UFW Firewall   |
              |    TCP 22        |
              +--------+---------+
                       |
                       v
              +------------------+
              | OpenSSH Server   |
              +--------+---------+
                       |
              +--------+---------+
              |                  |
              v                  v
        Public Key           Password
       Authentication      Authentication
              |                  |
           ALLOWED             DENIED
              |                  |
              v                  X
         SSH ACCESS
```

---

## Security Outcome

The Ubuntu server was hardened by:

- Identifying listening network services
- Configuring a default-deny incoming firewall policy
- Explicitly permitting SSH through UFW
- Generating an ED25519 SSH key pair
- Protecting the private key with a passphrase
- Installing the public key on the Ubuntu server
- Verifying public-key authentication before changing SSH settings
- Disabling password-based SSH authentication
- Validating the SSH configuration before applying changes
- Verifying the effective SSH daemon configuration
- Testing that password-only SSH access was rejected

The resulting configuration requires an authorized SSH key for remote administration while the Ubuntu account password remains available for local privilege escalation through `sudo`.

---

## Skills Practiced

This lab provided hands-on experience with:

- Linux server administration
- SSH
- OpenSSH configuration
- Public-key cryptography
- ED25519 authentication
- Linux file paths and configuration files
- UFW firewall configuration
- TCP/IP and port management
- Linux service management with `systemctl`
- Network socket inspection with `ss`
- Security hardening
- Configuration validation
- Troubleshooting configuration precedence
- Verification and security testing

---

## Key Takeaways

One of the most important lessons from this lab was that editing a configuration file does not automatically mean the running service is using that configuration.

When `PasswordAuthentication` continued to report `yes`, I investigated the SSH configuration files and identified a cloud-init configuration that was setting the value.

I then created a dedicated hardening configuration, validated the SSH configuration, reloaded the service, checked the effective configuration, and tested the authentication behavior.

This reinforced a security administration principle:

**Configure, validate, apply, and verify.**
