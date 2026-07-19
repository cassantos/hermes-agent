Phase 4  Expand Artifactization to core tools

Objective
---------
Extend two-tier persistence to the core toolset (search_files, skill_view, terminal, and others identified as high-volume), enforce retention policies, and provide a production-grade rehydration API and access control.

Components affected
-------------------
- Tool adapters for search_files, skill_view, terminal
- Persistence Manager policies and configuration
- Artifact Store scale/retention/ACL features
- Rehydration service (API, caching)
- Monitoring and billing for storage usage

Acceptance criteria
-------------------
- Core tools persist artifacts and only save summary+pointer in Conversation Memory across canary/gradual rollout.
- Rehydration API reliably returns artifacts with acceptable latency and auth controls.
- Retention policies enforce storage lifecycle; storage growth under expected thresholds.
- Tokens_est and TPM errors reduced on representative workloads.

Tests
-----
- Per-tool functional tests: artifact creation, pointerization, rehydrate, ACL checks.
- Load tests for rehydration latency and artifact store throughput.
- Storage retention test (policy enforcement dry-run).

Rollback plan
-------------
- Revert tools to legacy inline persistence via feature flags.
- Provide migration plan to restore pointers to inline content if needed.

Estimate
--------
Effort: 4-8 engineer-weeks (integration, policy, scaling, security)
Risk: High (storage, cost, access control, data lifecycle)

Dependencies
------------
- Phase 0..3 completed and validated
- Storage capacity and security review
