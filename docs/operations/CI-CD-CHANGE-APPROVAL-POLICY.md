# CI/CD Change Review and Deployment Approval Policy

**Status:** Proposed design (7 November 2026). CHG-002 is proposed. Independent review and separate deployment approval are pending.
## Principles

1. No direct deployment from a developer workstation or an unreviewed commit to a shared/production-like target.
2. Changes begin on a feature branch and are proposed through a pull request (PR).
3. CI runs non-destructive checks: formatting, linting, tests, config/IaC validation, secret scanning, and a reviewed plan/diff where relevant.
4. A genuinely independent authorised person reviews the PR's rationale, change scope, security, cost impact, evidence, and rollback plan before merge. Author approval is not equivalent to independent review.
5. Merge to protected `main` occurs only after required CI status checks and required PR approval, with stale approvals dismissed for changed code where supported.
6. Promotion to the `production` environment requires a **second, explicit deployment approval** from an authorised person, after the exact release/version and deployment plan are reviewed. An approved PR does not automatically approve deployment.
7. The deployment job must be bound to the GitHub Actions `production` environment, with required reviewers, prevent self-review and no bypass where these features are supported. Restrict deployment to the approved branch.
8. After approval, deploy the reviewed artifact/plan; run smoke tests, health checks, audit logging and post-deployment verification. Roll back if acceptance criteria fail.
9. Record who approved, what changed, the commit SHA/artifact ID, test results, actual outcome and any incident or lesson learned.
10. Emergency changes use a documented, restricted break-glass procedure, independent review as soon as practical and a post-change review; never silently bypass controls.

## Solo-engineer portfolio rule

- A separate email address or a second identity controlled by the same person is **not** independent review. Do not represent it as four-eyes governance.
- A documented solo-lab release checklist and self-authorisation is useful practice, but must be labelled **self-review / simulation**.
- Until a real independent reviewer and enforceable GitHub environment protection are in place, the example workflow must have **no real deployment command, credentials or privileged cloud permissions**. It may show the approval environment only as a demo.
- Publishing the repository is conditional on a full current-file and **Git history** secret/privacy review; making a repo public does not retroactively sanitize commits.
- For this change, document the baseline (`HTTP/1.1 200 OK`, 7 Nov 2026 17:18 UTC), implementation tests, rollback, and actual outcome against `docs/changes/CHG-002-nginx-backend-integration.md`.

## Flow

`feature branch → PR → automated checks → independent PR approval → protected main → release candidate + change record → manual environment approval → deployment → smoke tests → observation → close change`

## GitHub implementation notes (verified November 2026)

- Branch protection/rulesets can require PR reviews, minimum approvals, CI checks, dismiss stale approvals and prevent direct pushes, subject to plan and repository eligibility.
- GitHub Actions environments can require reviewer approval *before* the deployment job begins, can prevent self-review and can disallow bypass, when supported.
- **Plan limitation:** GitHub documentation states that required-reviewer deployment protection rules are available on public repositories for GitHub Free, Pro, and Team plans. A **private repository on those plans cannot be assumed to enforce this environment-level approval**; confirm actual plan and repository controls before claiming enforcement. GitHub Enterprise plan/organisation features may differ.
- For a private solo-learning repository without supported environment reviewers or a second authorised reviewer, **do not automate actual deployment**. Keep the deployment job disabled/not connected, record a manual approval exercise as a simulation, or use a supported organisational plan with independent approvers. Never label a simulated approval as an enforced production gate.
- The example workflow `ci-validation-and-deployment-gate.yml` runs basic real CI checks (diff whitespace and required Markdown/documentation files) on PRs and uses `workflow_dispatch` for a manually initiated, environment-bound **approval demonstration only**; no real deployment credentials or production deployment steps are included. Reviewer and branch protection must be configured separately, not in YAML alone.

## Example review checklist

- [ ] Business reason, intended change and acceptance criteria recorded
- [ ] Security/IAM/firewall impact reviewed
- [ ] Cost/resource impact and alternatives assessed
- [ ] Tests and `git diff`/Terraform plan evaluated
- [ ] Backup and rollback steps practical and tested where possible
- [ ] Peer reviewer approval recorded, independent of author
- [ ] Deployment approval separately recorded, independent of initiator
- [ ] Change evidence and post-deployment service verification archived

## Lab status

The current NGINX installation was performed manually before this policy was introduced. Do not retroactively claim it went through an approved pipeline. We will use these steps for subsequent changes and document any controls that are simulated rather than technically enforced.

## Official documentation

- https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets
