# CHG-001 — Ubuntu Firewall Configuration

**Project:** Linux Platform Deployment and Recovery Lab  
**Server:** platform-dev-ubuntu-01  
**Operating System:** Ubuntu 24.04.5 LTS  
**Environment:** Local VMware Fusion development lab  
**Change ID:** CHG-001  
**Change Type:** Security configuration  
**Change Status:** Implemented — firewall verified; fresh SSH acceptance test awaiting confirmation  
**Risk Level:** Medium  
**Change Owner:** Linux Platform Lab Administrator

---

## 1. Objective

Enable and configure the Ubuntu Uncomplicated Firewall (UFW) to restrict unauthorised incoming traffic while preserving SSH administrative access from the trusted VMware host.

This change follows the principle of least privilege by allowing only the required incoming network traffic.

## 2. Initial State

Before the change, the following configuration was identified:

| Component | Initial Configuration |
|---|---|
| Operating system | Ubuntu 24.04.5 LTS |
| Hostname | platform-dev-ubuntu-01 |
| Network interface | ens33 |
| Server IP address | 192.168.182.185 |
| Default gateway | 192.168.182.2 |
| IP assignment | DHCP |
| SSH service | Listening on TCP port 22 |
| SSH client address | 192.168.182.1 |
| UFW status | Inactive |

Initial verification:

```bash
sudo ufw status verbose
```

Observed output:

```text
Status: inactive
```

## 3. Pre-Change Verification

### 3.1 Verify SSH Listening Port

Command:

```bash
sudo ss -tlnp | grep ':22 '
```

**Explanation:**

- `sudo` — Run the command with administrative privileges.
- `ss` — Display socket statistics.
- `-t` — Display TCP sockets.
- `-l` — Display listening sockets.
- `-n` — Display numerical addresses and ports.
- `-p` — Display associated processes.
- `|` — Pass the output of the first command to the next command.
- `grep ':22 '` — Filter lines containing the port 22 pattern.

**Observed result:** SSH was listening on TCP port 22 using IPv4 and IPv6.

The listener was associated with `sshd` and `systemd`.

### 3.2 Verify SSH Connection Information

Command:

```bash
echo "$SSH_CONNECTION"
```

Observed result:

```text
192.168.182.1 60124 192.168.182.185 22
```

**Interpretation:**

| Value | Meaning |
|---|---|
| 192.168.182.1 | SSH client IP address |
| 60124 | Temporary client source port |
| 192.168.182.185 | Ubuntu server IP address |
| 22 | SSH destination port |

This information was used to identify the trusted SSH source address for the firewall rule.

## 4. Change Implementation

### 4.1 Configure Restricted SSH Access

Command:

```bash
sudo ufw allow from 192.168.182.1 to any port 22 proto tcp
```

**Purpose:** Allow incoming SSH connections from the trusted VMware host only.

Observed result:

```text
Rules updated
```

### 4.2 Verify the Configured Rule

Command:

```bash
sudo ufw show added
```

Observed result:

```text
Added user rules (see 'ufw status' for running firewall):
ufw allow from 192.168.182.1 to any port 22 proto tcp
```

The SSH access rule was configured before the firewall was activated.

### 4.3 Verify Default Firewall Policies

Command:

```bash
sudo grep -E '^(DEFAULT_INPUT_POLICY|DEFAULT_OUTPUT_POLICY|DEFAULT_FORWARD_POLICY)=' /etc/default/ufw
```

Observed result:

```text
DEFAULT_INPUT_POLICY="DROP"
DEFAULT_OUTPUT_POLICY="ACCEPT"
DEFAULT_FORWARD_POLICY="DROP"
```

**Explanation:**

- `DROP` for incoming connections: Discard incoming packets unless another rule permits them.
- `ACCEPT` for outgoing traffic: Allow outgoing traffic by default.
- `DROP` for forwarding: Prevent forwarding by default.

The default policies already matched the intended configuration, so no changes were necessary.

### 4.4 Activate the Firewall

Command:

```bash
sudo ufw enable
```

The system warned that enabling the firewall might disrupt SSH connections.

The previously configured SSH allow rule was checked before proceeding.

The operation was confirmed by entering `y`.

Observed result:

```text
Firewall is active and enabled on system startup
```

## 5. Post-Change Verification

### 5.1 Verify Firewall Status

Command:

```bash
sudo ufw status verbose
```

Observed output:

```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    192.168.182.1
```

### 5.2 Verify Administrator and Server Identity

Commands:

```bash
whoami
hostname
```

Observed results:

```text
david
platform-dev-ubuntu-01
```

### 5.3 Latest Verification Results

The following verification was repeated successfully:

```bash
whoami
hostname
sudo ufw status verbose
```

Observed output:

```text
david
platform-dev-ubuntu-01
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    192.168.182.1
```

The successful UFW command confirms that the firewall was still active during this verification.

The login session also showed:

```text
Last login: Sat Oct 31 18:23:30 2026 from 192.168.182.1
```

This identifies the source of the previous recorded SSH login. It does not, by itself, prove when the current SSH session was established.

### 5.4 Verification Summary

| Verification Check | Result | Status |
|---|---|---|
| UFW firewall | Active | Passed |
| Firewall enabled on startup | Confirmed | Passed |
| Incoming policy | Deny | Passed |
| Outgoing policy | Allow | Passed |
| Routed traffic | Disabled | Passed |
| Firewall logging | Enabled (low) | Passed |
| SSH destination | TCP 22 | Passed |
| SSH permitted source | 192.168.182.1 | Passed |
| Administrator account | david | Passed |
| Server hostname | platform-dev-ubuntu-01 | Passed |
| Fresh SSH connection after UFW activation | Awaiting confirmation | Pending |

## 6. Risk Assessment

**Primary Risk:** Misconfigured firewall rules could block legitimate SSH administrative connections.

**Potential Impact:** Loss of remote server administration, operational interruption and the need for console-based recovery.

**Mitigation Measures:**

1. Verify the current server and client IP addresses.
2. Identify the SSH listening port.
3. Add the restricted SSH rule before activating UFW.
4. Check default firewall policies.
5. Keep a working SSH session open during changes.
6. Maintain VMware Fusion console access for emergency recovery.
7. Validate the firewall configuration and establish a new SSH connection after activation.

## 7. Rollback and Recovery Procedure

If SSH becomes unavailable following a firewall change:

1. Open the Ubuntu VM console in VMware Fusion.
2. Log in using an authorised administrator account.
3. Inspect the firewall configuration.

```bash
sudo ufw status verbose
```

4. Identify the incorrect or missing firewall rule.

5. Correct the rule if possible.

6. If necessary for emergency recovery, temporarily disable UFW:

```bash
sudo ufw disable
```

**Security warning:** Disabling UFW removes its active enforcement. This should be an authorised, temporary recovery action rather than a routine troubleshooting step.

7. Restore the approved access rules and re-enable the firewall:

```bash
sudo ufw enable
sudo ufw status verbose
```

8. Verify fresh SSH access and record the recovery action.

**Rollback Test Status:** Procedure documented; intentional recovery testing has not yet been completed.

## 8. Final SSH Acceptance Test

### 8.1 Objective

Verify that a new administrative SSH connection can be established after enabling UFW without disabling or weakening the firewall.

### 8.2 Test Procedure

From a new Mac terminal:

```bash
ssh david@192.168.182.185
```

After successful authentication:

```bash
whoami
hostname
sudo ufw status verbose
```

### 8.3 Acceptance Criteria

The test passes when:

- A new SSH session is established after the firewall was activated.
- The authenticated user is `david`.
- The server hostname is `platform-dev-ubuntu-01`.
- The UFW firewall remains active.
- The expected SSH allow rule remains configured.

### 8.4 Current Test Status

**Pending confirmation of session timing.**

The administrator, hostname and UFW checks have produced the expected results.

However, the available evidence does not conclusively establish that these checks were performed in a newly opened SSH session after UFW activation.

Once this is confirmed, update:

**Fresh SSH acceptance test:** Passed

And update the change status to:

**Completed — Verified**

## 9. Evidence

Evidence directory:

`evidence/stage-03-security/`

Recommended evidence records:

| Evidence File | Description |
|---|---|
| `ufw-before-enable.txt` | Initial inactive status |
| `ssh-listener.txt` | SSH listening port and process |
| `ssh-connection-details.txt` | Client and server connection details |
| `ufw-added-rule.txt` | Restricted SSH firewall rule |
| `ufw-default-policies.txt` | Default firewall policy verification |
| `ufw-active-status.txt` | Active firewall output |
| `ssh-post-change-verification.txt` | Fresh SSH connection verification |

Only actual command output should be stored as test evidence.

## 10. Lessons Learned

This change demonstrated several important Linux Platform Engineering practices.

1. Verify the existing configuration before applying security changes.
2. Identify the authorised SSH source before restricting inbound traffic.
3. Apply least-privilege firewall rules.
4. Understand the difference between configured rules and active enforcement.
5. Validate changes using observed system output.
6. Keep a recovery method available when modifying remote administration controls.
7. Document risks, implementation steps, verification results and rollback procedures.
8. Never mark an operational test as passed without sufficient evidence.

## 11. Related Documentation

- `docs/runbooks/firewall-management.md`
- `docs/runbooks/ssh-access-recovery.md`
- `docs/security-hardening.md`
- `docs/architecture.md`

---

**Final Change Status:** Implemented — firewall verified; final SSH acceptance confirmation outstanding.