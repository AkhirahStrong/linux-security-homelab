# UFW Firewall & Nginx Connectivity Lab

## Objective

The objective of this lab was to configure and test Ubuntu's UFW host-based firewall and demonstrate the relationship between:

- A running service
- A listening network port
- Firewall rules
- Client connectivity

Nginx was used as the test web service.

---

## Lab Environment

| Component             | Configuration             |
| --------------------- | ------------------------- |
| Server OS             | Ubuntu Server 22.04.5 LTS |
| Virtualization        | UTM / QEMU                |
| Architecture          | ARM64                     |
| Firewall              | UFW                       |
| Web Server            | Nginx                     |
| Remote Administration | OpenSSH                   |
| HTTP Port             | TCP 80                    |
| SSH Port              | TCP 22                    |

---

## 1. Review Listening Services

I inspected the server's listening TCP and UDP ports:

```bash
sudo ss -tulpn
```

The output showed that OpenSSH was listening on TCP port 22 and Nginx was listening on TCP port 80.

Example:

```text
0.0.0.0:22    OpenSSH
0.0.0.0:80    Nginx
```

This confirmed that Nginx was running and ready to accept HTTP connections.

---

## 2. Review Available UFW Application Profiles

I checked which application profiles were registered with UFW:

```bash
sudo ufw app list
```

Available profiles included:

```text
Nginx Full
Nginx HTTP
Nginx HTTPS
OpenSSH
```

The profiles correspond to different network services:

| Profile     | Port         | Purpose                   |
| ----------- | ------------ | ------------------------- |
| OpenSSH     | TCP 22       | SSH remote administration |
| Nginx HTTP  | TCP 80       | HTTP web traffic          |
| Nginx HTTPS | TCP 443      | HTTPS web traffic         |
| Nginx Full  | TCP 80 & 443 | HTTP and HTTPS            |

An available UFW application profile does not mean the application is currently allowed through the firewall.

---

## 3. Review Current Firewall Rules

I checked the active firewall configuration:

```bash
sudo ufw status
```

Initially, only OpenSSH was permitted:

```text
OpenSSH       ALLOW    Anywhere
OpenSSH (v6)  ALLOW    Anywhere (v6)
```

Nginx was running and listening on TCP port 80, but HTTP was not permitted through UFW.

---

## 4. Test HTTP Connectivity

From the macOS host, I attempted to connect to the Ubuntu server's web service:

```bash
curl --connect-timeout 5 http://<server-ip>
```

The connection failed:

```text
Failed to connect to <server-ip> port 80:
Timeout was reached
```

This demonstrated that a service can be running and listening on a network port while still being inaccessible because of firewall policy.

---

## 5. Allow HTTP Through UFW

I allowed the Nginx HTTP application profile:

```bash
sudo ufw allow 'Nginx HTTP'
```

I then verified the firewall configuration:

```bash
sudo ufw status
```

The firewall now permitted both SSH and HTTP:

```text
OpenSSH            ALLOW    Anywhere
Nginx HTTP          ALLOW    Anywhere
OpenSSH (v6)        ALLOW    Anywhere (v6)
Nginx HTTP (v6)     ALLOW    Anywhere (v6)
```

Only the HTTP profile was enabled rather than `Nginx Full` because HTTPS had not yet been configured.

This follows the principle of least privilege by allowing only the network access currently required.

---

## 6. Verify Connectivity

From the macOS host, I repeated the HTTP request:

```bash
curl http://<server-ip>
```

This time, Nginx returned its HTML welcome page.

The successful response confirmed that TCP port 80 was now permitted through UFW.

---

## Troubleshooting Process

The troubleshooting process followed this path:

```text
Nginx Installed
      |
      v
Is the service listening?
      |
      | sudo ss -tulpn
      v
YES - TCP 80
      |
      v
Is TCP 80 allowed through UFW?
      |
      | sudo ufw status
      v
NO
      |
      v
Test from client
      |
      v
TIMEOUT
      |
      v
Allow Nginx HTTP
      |
      | sudo ufw allow 'Nginx HTTP'
      v
Test from client again
      |
      v
SUCCESS
```

---

## Security Outcome

The server currently uses a default-deny approach for unsolicited incoming traffic while explicitly allowing required services.

Permitted services include:

```text
TCP 22  - SSH
TCP 80  - HTTP
```

Other unsolicited incoming connections remain denied by the default UFW policy.

---

## Skills Practiced

- Linux firewall administration
- UFW
- TCP/IP
- Network ports
- OpenSSH
- Nginx
- HTTP
- Network socket inspection
- Client/server connectivity testing
- `ss`
- `curl`
- Firewall troubleshooting
- Principle of least privilege
- Service verification

---

## Key Takeaway

A running application does not automatically mean that clients can reach it.

Successful network connectivity requires multiple components to work together:

```text
Application Running
        +
Port Listening
        +
Firewall Allows Traffic
        +
Network Connectivity
        =
Successful Connection
```

This lab demonstrated how checking each layer can isolate the cause of a connectivity problem instead of assuming the application itself has failed.
