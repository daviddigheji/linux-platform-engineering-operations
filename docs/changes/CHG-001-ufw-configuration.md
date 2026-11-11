# CHG-001 — Ubuntu Firewall Configuration

**Project:** Linux Platform Engineering & Operations\
**Server:** platform-dev-ubuntu-01  
**Operating System:** Ubuntu 24.04.5 LTS  
**Environment:** Local VMware Fusion development lab  
**Change ID:** CHG-001  
**Change Type:** Security configuration  
**Change Status:** Completed Verified
**Risk Level:** Medium  
**Change Owner:** Linux Platform Lab Administrator

---

## 1. Objective

Enable and configure the Ubuntu Uncomplicated Firewall (UFW) to restrict unauthorised incoming network traffic while preserving secure SSH administration from the trusted VMware host.

The implementation follows the principle of least privilege by permitting only explicitly authorised incoming connections.

## 2. Initial State

Before implementing the change, the following conditions were identified:

- Operating system: Ubuntu 24.04.5 LTS.
- Hostname: `platform-dev-ubuntu-01`.
- UFW installed but inactive.
- SSH service listening on TCP port 22.
- Server IP address: `192.168.182.185`.
- SSH client IP address: `192.168.182.1`.
- Network interface: `ens33`.
- Default gateway: `192.168.182.2`.
- IP address assigned using DHCP.

### Initial Firewall Status

```bash
sudo ufw status verbose
```

Observed result:

```text
Status: inactive
```

## 3. Pre-Change Verification

### 3.1 Verify SSH Listening Port

```bash
sudo ss -tlnp | grep ':22 '
```

**Purpose:** Identify the process listening on TCP port 22.

**Command explanation:**

- `sudo` — Run with elevated privileges.
- `ss` — Display socket statistics.
- `-t` — Display TCP sockets.
- `-l` — Display listening sockets.
- `-n` — Display numerical ports and addresses.
- `-p` — Display associated processes.
- `|` — Pass command output to another command.
- `grep ':22 '` — Filter matching lines containing port 22 followed by a space.

**Observed result:** SSH was listening on TCP port 22 for IPv4 and IPv6 connections.

### 3.2 Verify SSH Connection Information

```bash
echo "$SSH_CONNECTION"
```

Observed result:

```text
192.168.182.1 60124 192.168.182.185 22
```

**Interpretation:**

- `192.168.182.1` — SSH client IP address.
- `60124` — Temporary client source port.
- `192.168.182.185` — Ubuntu server IP address.
- `22` — SSH destination port.

The trusted client address was used to define the restricted SSH firewall rule.

## 4. Change Implementation

### 4.1 Configure Restricted SSH Access

```bash
sudo ufw allow from 192.168.182.1 to any port 22 proto tcp
```

**Purpose:** Permit incoming SSH connections only from the identified trusted client IP address.

**Result:**

```text
Rules updated
```

### 4.2 Verify Added Firewall Rule

```bash
sudo ufw show added
```

**Observed result:**

```text
Added user rules (see 'ufw status' for running firewall):
ufw allow from 192.168.182.1 to any port 22 proto tcp
```

### 4.3 Verify Default Firewall Policies

```bash
sudo grep -E '^(DEFAULT_INPUT_POLICY|DEFAULT_OUTPUT_POLICY|DEFAULT_FORWARD_POLICY)=' /etc/default/ufw
```

**Observed result:**

```text
DEFAULT_INPUT_POLICY="DROP"
DEFAULT_OUTPUT_POLICY="ACCEPT"
DEFAULT_FORWARD_POLICY="DROP"
```

**Explanation:**

- `DROP` for incoming traffic — Reject unsolicited incoming packets by silently discarding them unless allowed by a rule.
- `ACCEPT` for outgoing traffic — Permit outgoing connections by default.
- `DROP` for forwarded traffic — Prevent traffic from being forwarded through the server by default.

No changes to these defaults were required.

### 4.4 Activate UFW

```bash
sudo ufw enable
```

The system displayed:

```text
Command may disrupt existing ssh connections. Proceed with operation (y|n)?
```

The SSH allow rule had already been configured and verified.

The change was confirmed by entering `y`.

**Observed result:**

```text
Firewall is active and enabled on system startup
```

## 5. Post-Change Verification

### 5.1 Verify Firewall Configuration

```bash
sudo ufw status verbose
```

**Observed result:**

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    192.168.182.1
```

### 5.2 Verify Server Identity

```bash
whoami
hostname
```

**Observed result:**

```text
david
platform-dev-ubuntu-01
```

These results confirm the administrator account and target server identity.

### 5.3 Verification Summary

| Verification Check | Observed Result | Status |
|---|---|---|
| UFW enabled | Active | Passed |
| Firewall persistence | Enabled on startup | Passed |
| Incoming default policy | Deny | Passed |
| Outgoing default policy | Allow | Passed |
| Routed traffic | Disabled | Passed |
| Logging | On (low) | Passed |
| SSH destination port | TCP 22 | Passed |
| Permitted SSH source | 192.168.182.1 | Passed |
| Administrator account | david | Passed |
| Server hostname | platform-dev-ubuntu-01 | Passed |
| Fresh SSH connection after activation | Verified from 192.168.182.1 | Passed |

## 6. Risk Assessment

**Primary risk:** Enabling an incorrectly configured firewall could block remote administrative access.

**Impact:** Loss of SSH connectivity and interruption of remote server administration.

**Mitigation measures:**

1. Identify the trusted SSH client address.
2. Configure the required SSH allow rule before enabling UFW.
3. Confirm default firewall policies.
4. Keep the current SSH session open during activation.
5. Maintain access to the VMware Fusion console.
6. Verify SSH connectivity after activation.

## 7. Rollback and Recovery Procedure

If remote SSH access becomes unavailable:

1. Access the Ubuntu server through the VMware Fusion console.
2. Log in using an authorised administrator account.
3. Inspect firewall status and rules.

```bash
sudo ufw status verbose
```

4. Identify and correct the incorrect firewall rule.

5. If required to restore administrative access during an emergency, temporarily disable UFW:

```bash
sudo ufw disable
```

6. Restore the approved configuration, then re-enable UFW:

```bash
sudo ufw enable
sudo ufw status verbose
```

7. Establish a new SSH connection and verify that access has been restored.

Any emergency rollback should be documented in the change record.

**Note:** The rollback procedure is documented but has not yet been demonstrated through an intentional recovery test.

## 8. Final SSH Acceptance Test

From a new Mac terminal, establish a fresh SSH session:

```bash
ssh david@192.168.182.185
```

Once authenticated, execute:

```bash
whoami
hostname
sudo ufw status verbose
```

**Acceptance criteria:**

- A new SSH connection is established successfully after UFW activation.
- The correct Linux account is displayed.
- The correct hostname is displayed.
- UFW remains active.
- The authorised SSH access rule remains configured.

**Current status:** Passed — A fresh SSH session
from the Mac host was successfully verified
after UFW activation.

The `SSH_CONNECTION` variable confirmed
the client address `192.168.182.1`,
server address `192.168.182.185`,
and SSH destination port `22`.

The server hostname was confirmed as
`platform-dev-ubuntu-01`, and UFW remained
active with the expected restricted SSH rule.

## 9. Evidence

Evidence should be saved within:

`evidence/stage-03-security/`

Recommended evidence files:

- `ufw-before-enable.txt`
- `ssh-listener.txt`
- `ssh-connection-details.txt`
- `ufw-added-rule.txt`
- `ufw-default-policies.txt`
- `ufw-active-status.txt`
- `ufw-verbose-status.txt` — Verified UFW evidence.
- `ssh-post-change-verification.txt`

Evidence should contain actual observed results, not fabricated expected outputs.

## 10. Lessons Learned


This change demonstrates the importance of controlled security configuration on Linux servers.

Key lessons:

1. Always inspect the current system state before making changes.
2. Identify required network access before enabling restrictive firewall policies.
3. Apply the principle of least privilege to incoming network traffic.
4. Verify both the firewall configuration and application connectivity.
5. Maintain a recovery method when changing remote access controls.
6. Record implementation details, actual results and outstanding tests.
7. Keep runbooks and change records aligned with the deployed configuration.

## 11. Related Documentation

- `docs/runbooks/firewall-management.md`
- `docs/runbooks/ssh-access-recovery.md`
- `docs/security-hardening.md`
- `docs/architecture.md`

---

**Final Change Status:** Completed — Verified. The UFW firewall remains active, the restricted SSH rule is configured, and SSH connectivity from the authorised Mac host has been verified.
