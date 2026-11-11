# CHG-002 — NGINX Backend Integration

**Status:** Proposed — Awaiting documented authorisation\
**Environment:** Local Ubuntu 24.04 VM\
**Change type:** Service expansion\
**Risk level:** Low for an isolated lab; subject to verification

## Business justification

Expand the existing single-service web server into a two-component platform consisting of NGINX and a Python backend application.

The expansion will provide practical experience in service integration, dependency management, monitoring, change control and troubleshooting.

## Proposed architecture

HTTP Client → NGINX Reverse Proxy → Python Backend (127.0.0.1:8001)

The backend will remain accessible only through the local loopback interface.

## Implementation plan

1. Verify existing NGINX functionality.
2. Install required Python runtime components.
3. Build a minimal application with a health endpoint.
4. Create a dedicated systemd service.
5. Verify backend functionality independently.
6. Configure NGINX reverse proxy routing.
7. Test the integrated service.
8. Preserve deployment evidence.

## Risks

- Backend service fails to start.
- Dependency or application configuration errors.
- Incorrect upstream address or port.
- NGINX configuration errors.
- HTTP 502 responses if the upstream service becomes unavailable.

## Rollback

Preserve the original NGINX configuration before modifying it. Restore the verified configuration, validate it with `nginx -t`, reload NGINX and confirm the original HTTP endpoint responds successfully.

## Acceptance criteria

- Backend service starts successfully.
- Local backend health endpoint returns HTTP 200.
- NGINX configuration validation succeeds.
- Reverse proxy routes requests correctly.
- Existing SSH access remains available.
- No unnecessary firewall ports are opened.
- Implementation and test evidence are saved.

## Approval

**Current status:** Pending solo-lab authorisation.

Independent review and deployment approval will be enforced when the GitHub controls and an authorised reviewer are available.

## Lessons learned

To be completed from actual implementation observations and troubleshooting evidence.