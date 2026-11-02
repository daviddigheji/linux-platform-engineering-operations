# RB-001 — Ubuntu Firewall Management Runbook

**Project:** Linux Platform Deployment and Recovery Lab  
**Server:** platform-dev-ubuntu-01  
**Environment:** VMware Fusion development lab  
**Operating System:** Ubuntu 24.04.5 LTS  
**Service:** UFW firewall  
**Required Access:** sudo privileges and VMware console recovery access

## 1. Purpose

Provide a repeatable procedure for checking, configuring, troubleshooting and recovering the Ubuntu firewall without unnecessarily disrupting SSH administration.

## 2. Prerequisites

- Authorised Linux administrator account.
- Working SSH connection to the server.
- VMware console access.
- Current network configuration and approved firewall changes.

## 3. Check Firewall Status

```bash
sudo ufw status verbose
```

**Explanation:**

- `sudo`: Run with elevated privileges.
- `ufw`: Uncomplicated Firewall.
- `status`: Display firewall status and rules.
- `verbose`: Include additional configuration information.

**Expected result:** UFW is active, incoming traffic is denied by default, outgoing traffic is allowed, and approved access rules are present.

## 4. Display Configured Rules

```bash
sudo ufw show added
```

Displays user-added firewall rules, including rules configured before UFW is enabled.

## 5. Verify SSH Listening Port

```bash
sudo ss -tlnp | grep ':22 '
```

Shows TCP listening sockets and their processes, filtering for port 22.

**Expected result:** SSH is listening on TCP port 22.

## 6. Verify Authorised SSH Access

From the authorised Mac administration environment:

```bash
ssh david@192.168.182.185
```

**Expected result:** A new SSH connection succeeds.

## 7. Add an Approved Firewall Rule

Example for this lab:

```bash
sudo ufw allow from 192.168.182.1 to any port 22 proto tcp
```

The source IP must be verified before use. Do not assume this address will remain valid if VMware networking changes.

## 8. Troubleshooting

If an SSH connection fails:

1. Keep any functioning SSH session open.
2. Check UFW status and configured rules.
3. Verify the server IP address and SSH service.
4. Verify the source address and network route.
5. Use VMware console access if remote administration is unavailable.

## 9. Emergency Recovery

From the VMware console, inspect and correct the firewall rule.

If necessary to restore access:

```bash
sudo ufw disable
```

**Warning:** This temporarily stops UFW enforcement. Use only when justified, and restore the intended security policy promptly.

Verify and restore approved firewall rules before re-enabling UFW:

```bash
sudo ufw enable
sudo ufw status verbose
```

## 10. Verification and Closure

Confirm that:

- UFW is active.
- Default policies match the approved configuration.
- Only required inbound access is permitted.
- A fresh SSH connection succeeds.
- The change record and evidence are updated.

## 11. Related Documentation

- `docs/changes/CHG-001-ufw-configuration.md`
- `docs/security-hardening.md`
- `docs/runbooks/ssh-access-recovery.md`