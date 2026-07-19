Phase 0  Preparation and Safety (pre-migration)

Objective
---------
Prepare production and canary environments for safe incremental migration: backups, telemetry, feature-flagging mechanism, canary profile, runbooks and rollback playbooks.

Components affected
-------------------
- Telemetry & metrics pipeline
- state.db backups
- Deployment/config management (feature flags, canary profile)
- Runbooks and incident channels

Acceptance criteria
-------------------
- Verified backups of ~/.hermes/state.db and config saved off-host with checksums.
- Telemetry tokens_before/after, compression events, TPM errors and CQ proxies are emitted and visible in dashboards for canary profile.
- Canary profile created and testable.
- Rollback playbook documented and validated (dry-run) and available in repo.

Tests
-----
- Restore test from backup to a temp location.
- Simulate a config feature-flag flip in canary and validate that the old path is restored on rollback.
- Validate telemetry emits expected metrics for a short synthetic load.

Rollback plan
-------------
- Revert any config changes and restore config.yaml from backup.
- Disable canary profile if created.

Estimate
--------
Effort: 1-2 engineer-days
Risk: Low

Dependencies
------------
- Access to backup storage and permissions.
- Observability stack (logs/metrics) available.
