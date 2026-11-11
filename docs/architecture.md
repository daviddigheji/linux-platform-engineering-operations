# Linux Platform Engineering & Operations

## 1. Purpose

This project demonstrates the deployment,
security configuration, documentation,
and recovery of a Linux server in a
virtualised environment.

The lab develops practical Platform
Engineering skills, including Linux
administration, networking, access
management, firewall configuration,
troubleshooting, and documentation.

## 2. Infrastructure Overview

| Component | Configuration |
|---|---|
| Host | Intel Mac with VMware Fusion |
| VM | platform-dev-ubuntu-01 |
| OS | Ubuntu Server 24.04 LTS |
| Network interface | ens33 |
| Guest IP | 192.168.182.185 (DHCP) |
| Gateway | 192.168.182.2 |
| Remote access | OpenSSH, TCP 22 |
| Firewall | UFW |

Note: IP addresses are previously observed
lab values and should be verified.

## 3. Network Architecture

The Ubuntu Server virtual machine operates
on a VMware NAT network.

### Network Configuration

| Setting | Verified Value |
|---|---|
| Network interface | ens33 |
| IPv4 address | 192.168.182.185/24 |
| Default gateway | 192.168.182.2 |
| Network subnet | 192.168.182.0/24 |
| IP configuration | DHCP |
| Route metric | 100 |

### Connectivity Validation

The network interface was confirmed UP
using `ip -br address`.

The routing table was verified using
`ip route`.

Gateway connectivity was tested using:

```bash
ping -c 4 192.168.182.2
```

### Internet and DNS Verification

The Ubuntu VM completed these tests:

| Test | Result |
|---|---|
| Gateway ping | Passed: 0% packet loss |
| External IP ping | Passed: 0% packet loss |
| DNS resolution | Passed |
| HTTPS connection | HTTP/2 200 |

Commands used:
- `ping -c 4 192.168.182.2`
- `ping -c 4 1.1.1.1`
- `getent ahostsv4 google.com`
- `curl -I --max-time 10 https://www.google.com`

These tests confirm connectivity at
the time of testing.

### Network Security

UFW provides host firewall protection.
SSH on TCP port 22 was previously
restricted to the Mac host address
192.168.182.1.

Current firewall rules require verification.
