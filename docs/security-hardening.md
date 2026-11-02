# Linux Server Security Hardening

**Project:** Linux Platform Deployment and Recovery Lab  
**Server:** platform-dev-ubuntu-01  
**Operating System:** Ubuntu 24.04.5 LTS  
**Environment:** VMware Fusion Development Lab  
**Security Status:** Baseline configuration in progress

## 1. Objective

Establish a documented Linux security baseline that restricts unnecessary network access, supports authorised remote administration and provides verifiable security evidence.

## 2. Administrator Access

The server uses the administrator account `david`.

The account's membership of the `sudo` group and its administrative privileges were verified using:

```bash
whoami
id
sudo -l
```

The account also belongs to the `lxd` group, whose privileged access implications require further review.

## 3. SSH Configuration

SSH remote administration is available on TCP port 22.

The listening SSH service was verified using:

```bash
sudo ss -tlnp | grep ':22 '
```

The SSH connection was inspected using:

```bash
echo "$SSH_CONNECTION"
```

The server currently permits SSH traffic from the trusted VMware host address through a restricted UFW rule.

SSH key authentication, password authentication settings and other SSH hardening controls have not yet been fully audited.

## 4. Firewall Configuration

UFW has been enabled with the following configuration:

| Security Setting | Configuration |
|---|---|
| Firewall | Active |
| Incoming traffic | Deny by default |
| Outgoing traffic | Allow by default |
| Routed traffic | Disabled |
| Firewall logging | Enabled (low) |
| SSH destination port | TCP 22 |
| Allowed SSH source | 192.168.182.1 |

Verification command:

```bash
sudo ufw status verbose
```

Evidence:

`evidence/stage-03-security/ufw-active-status.txt`

## 5. Operating System and Updates

Operating system and kernel information were checked using:

```bash
cat /etc/os-release
uname -r
```

Package metadata was refreshed and available upgrades checked using:

```bash
sudo apt update
apt list --upgradable
```

The latest previously observed upgrade list contained no available package upgrades.

Package update status must be checked again during routine maintenance.

## 6. Verification Evidence

The current firewall configuration was captured on Ubuntu and securely transferred to the Mac using SCP.

The evidence was reviewed and confirmed to contain the active firewall status, default policies and restricted SSH rule.

## 7. Remaining Security Tasks

- Confirm fresh SSH connectivity after firewall activation.
- Review SSH authentication and hardening settings.
- Review privileged group memberships.
- Establish a routine patch-management procedure.
- Review security logging and auditing.
- Test documented recovery procedures.

## 8. Conclusion

The Ubuntu server now has an active UFW firewall with default-deny incoming traffic and a restricted SSH access rule.

This provides an initial security baseline for the development lab.

The system is not yet certified or claimed to be fully production-hardened. Further security testing and operational controls remain necessary.

## 9. Related Documents

- `docs/changes/CHG-001-ufw-configuration.md`
- `docs/runbooks/firewall-management.md`
- `docs/runbooks/ssh-access-recovery.md`