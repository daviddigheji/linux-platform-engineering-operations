# SSH Access Recovery Runbook

**Project:** Linux Platform Deployment and Recovery Lab  
| Field | Details |
|---|---|
| Server | `platform-dev-ubuntu-01` |
| Operating System | Ubuntu Server 24.04 LTS |
| Environment | VMware Fusion |
| Document Type | Operational Recovery Runbook |
| Document Status | Draft — recovery testing pending |
| Owner | Linux Platform Lab Administrator |
---

## 1. Purpose

This runbook provides a safe, repeatable procedure for investigating and recovering Secure Shell (SSH) access to an Ubuntu Server.

It supports Linux system administration, troubleshooting, incident response, service restoration, and operational documentation.

The objective is to restore authorised administrative access while maintaining the security of the server.

## 2. Scope

This runbook applies to the Ubuntu Server virtual machine `platform-dev-ubuntu-01`, running in VMware Fusion.

It covers:

- SSH connection timeouts.
- SSH connection refused errors.
- Incorrect usernames or passwords.
- SSH service failures.
- Firewall rules blocking authorised SSH access.
- Network configuration problems.
- Recovery through the VMware Fusion console.
- Post-recovery validation and incident documentation.

This runbook does not authorise changing security configurations without appropriate verification and approval.

## 3. Infrastructure Overview

| Component | Configuration |
|---|---|
| Server hostname | `platform-dev-ubuntu-01` |
| Operating system | Ubuntu Server 24.04 LTS |
| Virtualisation | VMware Fusion |
| Network interface | `ens33` |
| Previously verified server IP | `192.168.182.185` |
| Network subnet | `192.168.182.0/24` |
| Default gateway | `192.168.182.2` |
| Authorised SSH client | `192.168.182.1` |
| SSH protocol | TCP |
| SSH port | `22` |
| Host firewall | UFW |
| Recovery access | VMware Fusion console |

**Important:** The server uses DHCP. Verify its current IP address before attempting recovery because the address may change.

## 4. Safety Precautions

Before troubleshooting:

1. Ensure VMware Fusion console access is available.
2. Keep an existing working SSH session open where possible.
3. Never disable the firewall without a justified recovery requirement.
4. Do not restart networking or SSH services without understanding the impact.
5. Avoid modifying SSH authentication settings during initial diagnosis.
6. Do not store passwords, authentication tokens or private keys in logs or evidence files.
7. Record the actual results of commands rather than assumed outcomes.
8. Perform disruptive recovery simulations only in the authorised lab environment.

## 5. Initial Troubleshooting

### 5.1 Verify Server Identity

Run these commands inside Ubuntu:

```bash
hostname
whoami
```

**Purpose:**

- `hostname` displays the server name.
- `whoami` identifies the currently authenticated user.

**Expected values:**

- Hostname: `platform-dev-ubuntu-01`
- Administrator account: `david`

These values must be verified against the actual environment.

### 5.2 Verify Network Interface and IP Address

```bash
ip -br address
```

**Purpose:** Display a brief summary of network interfaces and their addresses.

Verify that:

- `ens33` is enabled.
- The server has an expected IPv4 address.
- The network configuration matches the VMware lab environment.

### 5.3 Verify the Routing Table

```bash
ip route
```

**Purpose:** Display the routes used by the server to reach local and external networks.

Previously verified default gateway:

`192.168.182.2`

### 5.4 Test Gateway Connectivity

```bash
ping -c 4 192.168.182.2
```

**Purpose:** Check whether the server can communicate with its default gateway.

A successful test normally shows replies with minimal or zero packet loss.

**Note:** Some systems block ICMP traffic, so a failed ping alone does not prove the gateway is unavailable.

## 6. SSH Service Troubleshooting

### 6.1 Check SSH Service Status

```bash
sudo systemctl status ssh --no-pager
```

**Purpose:** Determine whether the OpenSSH service is running.

Check whether the service is:

- `active (running)`
- `inactive`
- `failed`

Do not restart the service automatically. Investigate the cause of a failure first.

### 6.2 Verify the SSH Listening Port

```bash
sudo ss -tlnp | grep ':22 '
```

**Purpose:** Check whether a process is listening on TCP port 22.

A healthy SSH service should normally be listening on the configured interface and port.

If no listener appears, investigate the SSH service status and configuration.

### 6.3 Inspect SSH Service Logs

```bash
sudo journalctl -u ssh -n 30 --no-pager
```

**Purpose:** Display the latest 30 journal entries associated with the SSH service.

Review the logs for:

- Authentication failures.
- Configuration errors.
- Service startup failures.
- Permission problems.
- Unexpected service termination.

Do not publish sensitive authentication information.

### 6.4 Validate SSH Configuration

```bash
sudo sshd -t
```

**Purpose:** Check the SSH daemon configuration for errors.

If the command completes successfully, it normally produces no output.

This validates configuration syntax and certain checks but does not guarantee that remote authentication or connectivity will succeed.

Do not change the SSH configuration until the cause of the problem is understood.

## 7. Firewall Troubleshooting

### 7.1 Inspect the Firewall

```bash
sudo ufw status verbose
```

**Purpose:** Display active firewall rules, default policies, and logging settings.

The verified lab configuration includes:

| Setting | Expected Configuration |
|---|---|
| UFW status | Active |
| Incoming traffic | Deny |
| Outgoing traffic | Allow |
| Routed traffic | Disabled |
| Logging | On (low) |
| Allowed SSH source | `192.168.182.1` |
| SSH destination | TCP port 22 |

### 7.2 Investigate Missing SSH Rules

If the authorised Mac host cannot connect, verify the Mac's current VMware network IP address and the configured UFW allow rule.

Do not add broad access rules such as unrestricted SSH access without an approved reason.

Any firewall correction must be documented in the change record.

### 7.3 Emergency Firewall Recovery

If a firewall configuration prevents legitimate administration, access the server through the VMware Fusion console.

Inspect the current rules:

```bash
sudo ufw status verbose
```

Identify and correct the problematic rule where possible.

If disabling UFW is necessary as a temporary emergency measure, obtain appropriate authorisation and document the security impact.

```bash
sudo ufw disable
```

After restoring approved rules:

```bash
sudo ufw enable
sudo ufw status verbose
```

**Warning:** Disabling UFW removes its active firewall enforcement. This is an emergency recovery option, not a routine troubleshooting step.

## 8. SSH Client Troubleshooting

From the Mac Terminal, test the SSH connection:

```bash
ssh -v david@192.168.182.185
```

**Purpose:** Display additional SSH connection diagnostics.

Possible issues include:

| Error | Possible Cause |
|---|---|
| Connection timed out | Network or firewall problem |
| Connection refused | SSH service not listening or port rejected |
| Permission denied | Authentication or account configuration issue |
| Host key verification failed | Stored host key does not match the server's presented key |
| No route to host | Network routing or reachability issue |

**Security note:** Never bypass a host key warning without verifying the server's identity and the reason the key changed.

## 9. Recovery Using VMware Fusion

If SSH cannot be accessed remotely:

1. Open VMware Fusion on the Mac.
2. Select `platform-dev-ubuntu-01`.
3. Open the virtual machine console.
4. Sign in using the authorised Ubuntu account.
5. Verify the hostname and network configuration.
6. Inspect the SSH service.
7. Review SSH service logs.
8. Verify the firewall configuration.
9. Identify the confirmed cause.
10. Apply an authorised corrective action.
11. Verify the affected service.
12. Test SSH access from a new Mac Terminal session.

Avoid making several unrelated changes simultaneously. This helps identify which action actually resolved the issue.

## 10. Post-Recovery Verification

After applying a corrective action, open a new SSH session from the Mac:

```bash
ssh david@192.168.182.185
```

Once connected, run:

```bash
whoami
hostname
echo "$SSH_CONNECTION"
sudo ufw status verbose
```

Verify that:

- The authenticated user is `david`.
- The hostname is `platform-dev-ubuntu-01`.
- The SSH client address matches the authorised Mac host.
- The server is listening and accepting SSH connections.
- UFW remains active.
- The restricted SSH rule remains configured.

A successful login is not sufficient by itself if the original security controls have not been restored.

## 11. Incident Documentation

Every controlled recovery exercise should produce an incident record.

Record the following:

| Field | Required Information |
|---|---|
| Incident ID | Unique reference |
| Reported problem | Description of symptoms |
| Detection time | When the issue was identified |
| Severity | Assessed business or operational impact |
| Affected service | SSH or related component |
| Diagnostic actions | Commands and checks performed |
| Root cause | Confirmed reason for failure |
| Recovery action | Steps used to restore access |
| Resolution time | When service was restored |
| Verification | Evidence that recovery succeeded |
| Preventive action | Recommended improvement |

For simulated incidents, label the record clearly as a laboratory exercise.

## 12. Evidence Management

Store actual command outputs and supporting evidence in the appropriate project evidence folders.

Suitable evidence includes:

- SSH service status.
- SSH listening port information.
- Firewall status.
- Network interface and route information.
- Sanitised diagnostic logs.
- Post-recovery verification results.

Never fabricate successful test results.

Never commit passwords, private keys, or other credentials to Git.

## 13. Current Verification Status

The following activities have already been verified during the project:

- Ubuntu Server installation.
- Network interface and routing.
- Gateway connectivity.
- External IPv4 connectivity.
- DNS resolution.
- HTTPS connectivity.
- Active UFW firewall.
- Restricted SSH access configuration.
- Successful SSH connection from the Mac host.

**Outstanding activity:** A deliberately simulated SSH failure followed by a complete, documented recovery exercise.

**Runbook Status:** Draft — operational instructions documented; controlled failure-and-recovery testing pending.

## 14. Related Documentation

- `docs/architecture.md`
- `docs/security-hardening.md`
- `docs/changes/CHG-001-ufw-configuration.md`
- `docs/runbooks/firewall-management.md`
- `docs/deployment-guide.md`

---

**Document Review Status:** Initial runbook created. Pending hands-on recovery validation and final technical review.
