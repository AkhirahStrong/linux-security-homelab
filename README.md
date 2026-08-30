# Linux Security Home Lab

A hands-on Linux administration and cybersecurity home lab built to develop practical experience with server deployment, networking, system hardening, access control, troubleshooting, and security monitoring.

The lab is being built incrementally, with each configuration documented along with the security reasoning and verification process.

## Lab Environment

| Component             | Configuration             |
| --------------------- | ------------------------- |
| Host                  | macOS                     |
| Virtualization        | UTM / QEMU                |
| Server OS             | Ubuntu Server 22.04.5 LTS |
| Architecture          | ARM64                     |
| Remote Administration | OpenSSH                   |
| Host Firewall         | UFW                       |
| Authentication        | ED25519 SSH Keys          |

## Architecture

```text
              macOS Host
                  |
                  | SSH
                  | TCP 22
                  v
        +---------------------+
        |      UTM/QEMU       |
        |                     |
        |   Ubuntu Server     |
        |      ARM64          |
        +----------+----------+
                   |
          +--------+--------+
          |                 |
          v                 v
     UFW Firewall       OpenSSH
     Default Deny       Key Authentication
```

## Current Progress

### Ubuntu Server Deployment

- Deployed Ubuntu Server 22.04.5 LTS as an ARM64 virtual machine
- Configured virtual networking
- Configured remote administration using OpenSSH
- Updated installed system packages
- Verified operating system, kernel, architecture, and virtualization environment

### Network & Service Inspection

Used Linux networking tools to identify active interfaces, IP addressing, listening ports, and running network services.

Commands practiced include:

```bash
ip addr
ss -tulpn
hostnamectl
```

### UFW Firewall

Configured Ubuntu's host-based firewall with a default-deny approach for incoming connections.

Current policy:

```text
Incoming: DENY
Outgoing: ALLOW
SSH:      ALLOW TCP/22
```

Firewall logging is also enabled.

### SSH Hardening

Hardened remote administration by replacing password-based SSH authentication with ED25519 public-key authentication.

Implemented and verified:

- ED25519 SSH key generation
- SSH public-key deployment
- Private-key passphrase protection
- Public-key authentication
- Disabled password-based SSH authentication
- SSH configuration validation
- Authentication testing
- Troubleshooting SSH configuration precedence

[View SSH Hardening Documentation](docs/03-ssh-hardening.md)

## Security Approach

This lab follows a simple process when implementing security changes:

```text
ASSESS
   |
   v
CONFIGURE
   |
   v
VALIDATE
   |
   v
APPLY
   |
   v
VERIFY
```

Changes are tested before existing access methods are removed whenever possible to reduce the risk of accidental lockout.

## Skills Demonstrated

- Linux Server Administration
- Linux Command Line
- SSH / OpenSSH
- Linux Networking
- TCP/IP
- Firewall Configuration
- UFW
- Public-Key Authentication
- ED25519
- Linux Service Management
- System Hardening
- Access Control
- Patch Management
- Configuration Management
- Troubleshooting
- Security Verification

## Documentation

Detailed documentation will be added as each lab component is completed.

| Lab                   | Documentation                                  | Status      |
| --------------------- | ---------------------------------------------- | ----------- |
| Ubuntu Server Setup   | `docs/01-ubuntu-server-setup.md`               | In Progress |
| Network Configuration | `docs/02-network-configuration.md`             | In Progress |
| SSH Hardening         | [View Documentation](docs/03-ssh-hardening.md) | Complete    |
| UFW Firewall          | `docs/04-ufw-firewall.md`                      | In Progress |
| System Monitoring     | `docs/05-system-monitoring.md`                 | Planned     |

## Project Goals

The goal of this project is to build practical experience administering and securing Linux infrastructure rather than relying solely on theoretical knowledge.

Future additions will be documented as they are implemented and tested.

---

> This repository represents an actively developing home lab. Features listed as complete have been configured and tested in the lab environment.
