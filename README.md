# Linux Platform Engineering & Operations

**An evidence-driven Ubuntu operations and incident-response environment**

**Focus:** Linux systems administration · Service reliability · Network security · Troubleshooting · Operational documentation

**Environment:** Local VMware Fusion lab on an Intel Mac

**Project status:** In progress — baseline Ubuntu/NGINX functionality verified; CHG-002 reverse-proxy expansion proposed; independent review and deployment controls not yet enforced

**Repository:** `linux-platform-engineering-operations`

## Executive summary

**The problem.** Small IT teams and growing organisations can lose access to critical services when a host firewall, remote-administration path or application service is misconfigured. Restoring service without established diagnostics, controlled changes or recovery instructions can increase disruption and operational risk.

**The approach.** I built an isolated Ubuntu Server environment to practise the operational lifecycle of a Linux workload: deployment, network validation, least-privilege access, web-service verification, change recording, evidence capture and recovery planning. The lab uses VMware Fusion, Ubuntu, OpenSSH, UFW, systemd and NGINX. Each completed control is supported by observed commands or documented configuration rather than an assumed result.

**Current outcome.** The VM is reachable over SSH from its authorised VMware host address; UFW is active with default-deny inbound policy; DNS, external IP reachability and HTTPS were checked; and NGINX was installed, started and validated with an HTTP `200 OK` response and a successful configuration test. A two-service expansion has been proposed under CHG-002, while automated health checks, independent approvals, measured recovery and end-to-end backup restore remain to be demonstrated.

**Scope:** This is a single-VM, non-production engineering lab. It does **not** claim production traffic, business SLA achievement, high availability or savings measured at a customer organisation.

### At a glance

| Dimension | Demonstrated | Remaining / limitation |
|---|---|---|
| Linux platform | Ubuntu Server 24.04 LTS VM, hostname and network interface verified | Automated provisioning not yet implemented in this repository |
| Secure administration | OpenSSH on TCP/22; source-restricted UFW rule | Dedicated service identities and additional hardening to evaluate |
| Connectivity | Gateway, public IP, DNS resolution and HTTPS checks passed | Routine automated checks and alerting pending |
| Web workload | NGINX active/enabled; loopback HTTP 200; config syntax passed; TCP/80 listener observed | Remote HTTP policy, workload-specific config and monitoring pending |
| Change control | CHG-001 UFW record; CHG-002 backend integration proposed | PR rules, independent review and deployment approval not yet enforced |
| Recovery | Recovery instructions documented | Integration fault diagnosis (if naturally encountered) and measured restoration or explicitly labelled resilience test pending |

### Verification evidence

- [UFW firewall status and rules](evidence/stage-03-security/ufw-verbose-status.txt)
- [NGINX configuration test](evidence/stage-05-service-deployment/nginx-config-test.txt)
- [NGINX service status](evidence/stage-05-service-deployment/nginx-service-status.txt)
- [SSH access recovery runbook](docs/runbooks/ssh-access-recovery.md)

For the operational context, see [Operations, recovery and evidence](#6-operations-recovery-and-evidence).

## 1. Business objectives and stakeholder value

### Hiring managers and technical reviewers

This project demonstrates how I approach an operational system: confirm the baseline, make a controlled change, inspect the resulting state, preserve command evidence and document rollback options. It provides concrete material for discussing Linux administration, incident diagnosis, network security, operational ownership and service reliability.

### Finance and accounting stakeholders

The lab runs on an existing local host rather than an always-on cloud VM, avoiding an additional metered compute deployment during development. This **does not** make the lab free: host hardware, electricity, staff time, storage and opportunity costs still exist. No monetary saving has been measured.

Production investment decisions would compare platform costs, support effort, recovery objectives and the financial impact of downtime. A future cost model will explicitly distinguish **measured costs** from **assumed scenarios**.

### Directors and senior management

The project demonstrates a repeatable approach to protecting administration access, verifying service health and preserving recovery knowledge. The value is reduced operational uncertainty and improved preparedness—not a claim of proven business uptime. Future reporting will include service health, tested recovery times, incident trends, risk ownership and prioritised recommendations.

## 2. Architecture and trust boundaries

```text

Intel Mac (VMware Fusion)

    |

    | VMware NAT virtual network (192.168.182.0/24)

    |

    +-- Host-side SSH client (observed: 192.168.182.1)

    |

    +-- Ubuntu Server: platform-dev-ubuntu-01

          Interface: ens33

          Observed IPv4: 192.168.182.185/24 (DHCP)

          Default gateway: 192.168.182.2

          |

          +-- OpenSSH / TCP 22

          |     UFW: allow source 192.168.182.1 only

          |

          +-- NGINX / TCP 80

                Service active and enabled

                Local loopback HTTP 200 confirmed

                Remote HTTP access not yet authorised/tested

```

**Network caveat:** The displayed addresses were observed in the lab and may change because DHCP is used. NGINX listening on an interface does not mean external access is permitted; UFW still controls inbound traffic.

**System of record:** [Architecture](docs/architecture.md) · [Security hardening](docs/security-hardening.md) · [UFW change record](docs/changes/CHG-001-ufw-configuration.md)

## 3. Technology choices and engineering trade-offs

| Choice | Why I used it for this lab | Trade-off / production consideration |
|---|---|---|
| **Ubuntu Server LTS** | Familiar enterprise Linux platform, APT packaging and systemd management; directly relevant to Linux support | Requires planned patching, vulnerability management, lifecycle policy and hardened baseline |
| **VMware Fusion** | Provides isolated, repeatable practice on existing hardware and emergency console access if SSH fails | Single-host dependency, no workload redundancy and host resource limits |
| **VMware NAT + DHCP** | Supports local host-to-guest access and outbound testing with minimal network changes | Guest address can change; production needs explicit addressing/discovery and controlled routing |
| **OpenSSH** | Standard remote administration and a realistic failure/recovery surface | Prefer identity-governed access, robust key management, audited access and restricted sources |
| **UFW** | Readable host-based default-deny policy and clear evidence for learning least privilege | Should complement network segmentation, upstream controls, monitoring and change automation |
| **NGINX** | Lightweight, widely deployed service for practising HTTP, systemd, configuration validation and outages | A single process/VM is not highly available; TLS, observability, tested deployment and capacity planning needed |
| **Markdown + Git** | Auditable text documentation and change history for runbooks, architecture and decisions | Production use benefits from reviews, protected branches, automated checks and versioned release processes |

**Decision pattern:** start with the simplest solution that meets the lab's learning and verification requirements; make limitations explicit; harden and automate only after a reliable baseline is proven.

## 4. Implementation and verified results

### 4.1 Linux and networking

- Installed and configured the Ubuntu Server VM under VMware Fusion.
- Verified hostname `platform-dev-ubuntu-01`, network interface `ens33`, IPv4 `192.168.182.185/24` and default route via `192.168.182.2`.
- Confirmed local gateway reachability with **4/4 replies and 0% packet loss**.
- Confirmed public-IP reachability to `1.1.1.1` with **4/4 replies and 0% packet loss** (observed mean RTT **53.078 ms** for that sample).
- Confirmed DNS returned an IPv4 address for `google.com`, then observed successful HTTPS response headers from the website.

These are point-in-time health checks, **not** availability statistics.

### 4.2 Security and administrative access

- Configured UFW as **active**, with **deny incoming / allow outgoing / routed disabled**, logging `on (low)`.
- Allowed SSH TCP/22 from the designated VMware host address `192.168.182.1`, rather than permitting all sources.
- Verified an SSH session identified source `192.168.182.1` and destination `192.168.182.185:22` using `SSH_CONNECTION`.
- Recorded and verified the firewall configuration in CHG-001, with raw text and screenshot evidence.

Evidence: [UFW verbose output](evidence/stage-03-security/ufw-verbose-status.txt) · [UFW screenshot](evidence/stage-03-security/ufw-verbose-status.png) · [Change record](docs/changes/CHG-001-ufw-configuration.md)

### 4.3 Web service deployment

- Refreshed APT package indexes and installed the `nginx` package.
- Verified NGINX service is **active (running)** and **enabled** in systemd.
- Observed TCP port **80** listeners for IPv4 and IPv6 using `ss`.
- Received **HTTP/1.1 200 OK** from NGINX on `127.0.0.1`.
- Ran `sudo nginx -t`: configuration syntax **OK**, test **successful**.
- Copied genuine NGINX configuration-test output from Ubuntu to the repository.

Evidence: [NGINX configuration test](evidence/stage-05-service-deployment/nginx-config-test.txt)

Evidence: [NGINX service status](evidence/stage-05-service-deployment/nginx-service-status.txt)

## 5. What went wrong: troubleshooting and engineering reasoning

These are genuine **development and workflow issues**, not fabricated production incidents.

| Observation / failure | Diagnosis and action | Lesson |
|---|---|---|
| Remote `sudo -n ufw status verbose` returned **a password is required** | `-n` forbids an interactive password prompt. Ran the authorised command inside Ubuntu and copied its output to the Mac over SCP | Separate authentication context from command execution; preserve least privilege rather than weakening sudo |
| Ubuntu password was initially rejected in one attempt | Retried successfully; after a previous VM reboot password entry also worked | Hidden password entry, keyboard/paste behaviour and interactive environment should be checked before attempting account recovery |
| Git rejected staged UFW evidence due to trailing whitespace and a blank line | Corrected non-semantic formatting, re-staged the file and re-ran `git diff --cached --check` | Validate the **staged** snapshot, not only the working tree; preserve the meaning of raw evidence |
| SSH recovery runbook contained an older copy of CHG-001 | Compared documents, created and byte-verified a backup, then replaced the wrong content with a dedicated SSH recovery runbook | Inspect existing state, create a rollback point and verify before replacing documentation |
| Initially checked a nonexistent `nginx` installation | `dpkg -l nginx` returned no matching package; refreshed APT metadata, installed NGINX and verified service state | Verify prerequisites and actual system state instead of assuming software is installed |
| One command used `system` instead of `systemctl` | Corrected the command and successfully inspected the NGINX service | Distinguish shell command errors from an application/service failure |

### Reasoning pattern used

1\. **Observe:** capture the exact error and scope (local shell, SSH, sudo, systemd, HTTP or Git).

2\. **Hypothesise:** identify the smallest plausible component that could explain it.

3\. **Test:** use low-risk read-only checks before changing configuration.

4\. **Correct:** make one controlled, reversible change.

5\. **Verify:** test the user-facing behaviour and the underlying control independently.

6\. **Record:** preserve the result, limitation and next preventive action.

**Not yet claimed:** No application-expansion incident, DNS failure, SSH lockout or recovery-time result has been completed in this project. A future exercise will evolve the service architecture and investigate **any genuine regression resulting from the change**; a failure will never be invented to complete a checklist.

### Proposed expansion: change-induced failures, not a staged outage

**Change record:** [CHG-002 — NGINX backend integration](docs/changes/CHG-002-nginx-backend-integration.md) is proposed. Independent review and separate deployment approval are pending. The NGINX root endpoint returned HTTP `200 OK` on 7 November 2026 at 17:18 UTC, providing a pre-change baseline. The proposed target is NGINX `/app/` reverse-proxying a loopback-only Python service on port 8001.

The next engineering iteration is to add a small application behind NGINX (reverse proxy -> locaally bound application service). The purpose is to practise a realistic migration from a simple web server to a multi-component service: process ownership, systemd unit configuration, loopback binding, upstream ports, proxy routing, log correlation, and rollback. **A regression is possible, not guaranteed.**

Before the change: write a change record, verify the current `/` HTTP 200 baseline, preserve a restorable configuration copy, and confirm VMware console access. During the change: modify one integration boundary at a time, validate NGINX syntax, start the application using a least-privileged service identity, and test the new route independently. If anything genuinely fails: record the timestamp, user-visible symptom, error code, relevant NGINX and application logs, falsifiable hypotheses, fix, rollback decision, and post-fix evidence. Never weaken UFW or expose the upstream listener publicly merely to make a test pass.

**Potential failure modes to assess, not claims of incidents:** HTTP 502 from an upstream socket/port mismatch, HTTP 404 from path handling, systemd `203/EXEC` caused by an invalid executable path, permission errors when the service identity lacks access, and configuration errors caught by `nginx -t`. Select the investigation based on the **actual observed symptoms**.

A genuine regression gets an incident record and lessons learned. A clean deployment gets a successful change report and a follow-up resilience test clearly marked as planned. The project will not misrepresent planned fault injection as an accidental production incident.

## 6. Operations, recovery and evidence

| Operational document | Purpose |
|---|---|
| [Deployment guide](docs/deployment-guide.md) | Repeatable server setup and operational context |
| [Architecture](docs/architecture.md) | Components, network boundaries and design intent |
| [Security hardening](docs/security-hardening.md) | Baseline security controls and verification |
| [UFW change record](docs/changes/CHG-001-ufw-configuration.md) | Change intent, checks, risk, acceptance and rollback |
| [Firewall management](docs/runbooks/firewall-management.md) | Routine firewall checks and safe handling |
| [SSH access recovery](docs/runbooks/ssh-access-recovery.md) | Diagnostics, emergency-console recovery and post-recovery validation |
| [Ubuntu provisioning SOP](docs/sops/ubuntu-server-provisioning.md) | Standard operating procedure |
| [Linux command reference](docs/command-reference/linux-commands.md) | Command explanations and troubleshooting reference |

**Evidence policy:** link only files that exist and were produced by actual tests; redact secrets, unnecessary personal data and sensitive access details. Raw output is not a substitute for interpretation.

## 7. Production-readiness assessment

This lab intentionally prioritises operational learning over production scale. To make a comparable service suitable for real customers, I would assess and implement the following, subject to business requirements and threat modelling:

| Area | Current lab | Production improvement |
|---|---|---|
| Availability | Single VM and host | Define SLOs and failure domains; use redundancy and tested failover if justified |
| Deployment | Manual installation and documented procedures | Version-controlled configuration/IaC, CI validation, approvals and rollback |
| Access | Restricted SSH source + UFW | Centralised identity, key/credential lifecycle, MFA where supported, privileged-access audit and separation of duties |
| Network | Local VMware NAT | Segmented networking, managed DNS, controlled ingress/egress and load balancing where justified |
| Web security | Default HTTP service | TLS termination, certificate renewal, secure headers and explicit exposure policy |
| Observability | Point-in-time `systemctl`, `ss`, `curl` and logs | Metrics, central logs, alert ownership, dashboards and actionable thresholds |
| Recovery | Documented runbook | Tested backups, restore procedures, defined RTO/RPO and game-day exercises |
| Vulnerability management | APT update and configuration checks | Patch cadence, package inventory, vulnerability scanning and change windows |
| Cost | Existing local host, no cloud billing for this VM | Capacity forecasts, measured spend, tagging, budgets and periodic rightsizing |
| Governance | Git and change record | Peer reviews, protected branches, change approvals, evidence retention and audit trails |

**AWS translation, not implemented in this lab:** for an AWS-hosted equivalent, evaluate private EC2 instances, IAM roles, Systems Manager Session Manager, Systems Manager Patch Manager, security groups, CloudWatch, AWS Backup, managed certificates and infrastructure as code. Choose components according to operational needs rather than copying a reference architecture blindly. Nothing in this repository should imply that this Ubuntu VM currently runs on AWS.

## 8. Lessons learned

- **Evidence is stronger than a status claim.** A running process, passing config test and successful HTTP response each verify a different layer; none alone proves end-to-end reliability.
- **Least privilege is observable.** Default-deny UFW and source-specific SSH access can be validated, but must also be rechecked after network changes.
- **Change safety depends on recovery access.** VMware console access protects against remote-administration lockouts during controlled experiments.
- **Troubleshooting is hypothesis-driven.** Authentication, permissions, service state, routing and HTTP failures require different tests.
- **Documentation can itself fail.** The duplicated runbook showed the importance of file inventory, backups, content checks and review.
- **Operations cost money even in a lab.** A local VM reduces cloud-metered usage but still consumes hardware capacity, time and energy.
- **Production readiness must be stated, not implied.** Tested lab behaviours are valuable; untested availability and recovery claims would be misleading.

## CI/CD review and deployment approval — required design

For future changes, use feature branches and pull requests, automated validation, an **independent PR review**, a protected `main` branch, and a **separate explicit approval before deployment**. Only the approved revision should deploy, followed by smoke tests, monitoring, and rollback if checks fail. The change record must show reviewer, approver, artifact/commit, test outcomes and final result.

**Current status:** Proposed, not yet configured or enforced. A second email or account controlled by the author does not constitute independent approval. GitHub required-reviewer deployment gates have plan/repository visibility limitations, including private repositories on Free/Pro/Team plans; until a supported independent approval mechanism exists, keep real automated deployment disabled and label any rehearsal as simulation. The existing NGINX deployment predates this policy; do not describe it as pipeline-approved. See [CI/CD change and deployment approval policy](docs/operations/CI-CD-CHANGE-APPROVAL-POLICY.md) and [.github/workflows/ci-validation-and-deployment-gate.yml](.github/workflows/ci-validation-and-deployment-gate.yml).

CI/CD validation status: The GitHub Actions workflow passed local static validation using actionlint 1.7.12 on the Intel Mac. The same version was installed and verified on the Ubuntu VM. GitHub-hosted execution, branch protection, independent pull-request approval and mandatory deployment approval have not yet been configured or verified. Automated infrastructure deployment remains disabled.

## 9. Remaining work and acceptance criteria

- [x] Copy and inspect NGINX `systemctl status` evidence on Mac.
- [ ] Stage, review and commit NGINX service-status evidence.
- [ ] Capture an HTTP response transcript and verify listener/service status evidence.
- [x] Create CHG-002 backend-integration change-record file locally (content approval/review pending).
- [ ] Review and authorise CHG-002 as a documented solo-lab change, explicitly separate from independent approval.
- [ ] Document NGINX deployment/configuration and safe network exposure decision.
- [ ] Implement PR checks, independent code review, a separate deployment approval, and verified GitHub enforcement where eligible; keep real automated deploy disabled until then.
- [ ] Add a Bash health check with meaningful exit codes and failure handling.
- [ ] Expand NGINX from a static page into a reverse proxy for a small locally bound upstream application, with a documented change and rollback plan.
- [ ] When a **real integration regression** occurs during expansion, capture symptoms and logs, evaluate hypotheses, correct the cause, verify end-to-end service and write a blameless incident review. If the deployment succeeds without fault, report success honestly and do not manufacture an outage.
- [ ] Add a change-induced regression test and demonstrate recovery through a separate **explicitly labelled planned resilience exercise**, only if needed and approved in the lab.
- [ ] Test a file/configuration backup and restore; record observed recovery time.
- [ ] Create at least one realistic support ticket, incident report and knowledge-base article.
- [ ] Add measured/simulated SLA and finance reports with clearly defined assumptions.
- [ ] Prepare technical, finance and executive presentations from the same verified evidence.
- [ ] Review the repository for secrets, personal information, stale files and unverified claims before publication.

**Completion rule:** a task is marked complete only when its acceptance criteria and supporting evidence have been checked.

## 10. Reproducing and validating the lab

This repository is currently a **documented hands-on lab**, not a one-command provisioner. On an authorised Ubuntu Server 24.04 LTS VM with network access, first inspect the environment and adapt addresses/settings to your own setup.

Read-only checks:

```bash

hostname

ip -br address

ip route

sudo ufw status verbose

systemctl status nginx --no-pager

sudo nginx -t

curl -I http\\://127.0.0.1

```

These commands assume NGINX and UFW have already been installed/configured. Do not blindly copy firewall rules or expose services on another machine; use the runbooks and change record to understand intended access first.

## 11. Interview walkthrough: decision, verification, improvement

**Problem:** How can an engineer securely operate and diagnose an Ubuntu-hosted service without assuming that a successful installation is enough?

**Action:** Built a VMware-hosted Ubuntu environment, restricted SSH by source with UFW, deployed NGINX, verified service/process/network/HTTP layers, and recorded change and recovery documentation. When evidence collection and documentation steps failed, investigated and corrected them rather than bypassing controls.

**Result:** Successful SSH and firewall validation, local NGINX HTTP `200 OK`, passing configuration test, and locally saved documentation and verification evidence for the completed stages, with outstanding changes awaiting review and commit. No business uptime or service-recovery metrics are claimed yet.

**Next iteration:** Implement CHG-002 with baseline, safe rollout and rollback; diagnose any naturally occurring integration regression; add monitoring, separately labelled resilience testing, recovery objectives and enforceable deployment controls.
