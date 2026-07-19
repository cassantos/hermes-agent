Phase 1  Pre-prune heuristic (mitigation)

Objective
---------
Implement a deterministic pre-prune step in the request-building pipeline that detects and reduces/removes obviously large blobs (tool outputs, base64 images, codex blobs) by replacing them with short placeholders or pointers before any call to the compressor or external LLMs. This reduces the risk of compressor-triggered requests exceeding provider TPM or context limits.

Components affected
-------------------
- Request building / turn-prologue pipeline
- Tool output handler adapters
- Logging/telemetry for compression attempts

Acceptance criteria
-------------------
- A feature-flagged pre-prune pass exists and can be toggled per profile (canary).
- For representative canary workload, manual compress (/compress) no longer builds LLM requests that exceed the TPM limit.
- Tokens_est before vs after pre-prune for the same session show a measurable reduction (target: >= 25% reduction for problematic sessions).
- No loss of auditability: original blobs are preserved in a safe store or saved as artifacts (local temp) until further artifactization decisions are made.
- No user-visible regressions in basic flows (read_file, search_files) in canary.

Tests
-----
- Run the representative workflow in canary (skill_view, search_files, read_file, terminal) and record tokens_est prior to pre-prune.
- Toggle pre-prune on and repeat the same workflow; measure tokens_est after.
- Execute manual compress in both states to ensure pre-prune prevents giant compress requests.
- Run a small human spot-check (10 sessions) to ensure no regressions in content accessibility.

Rollback plan
-------------
- Disable feature flag for pre-prune in canary; restore previous request builder.
- If any persisted placeholder logic accidentally removed raw blobs, restore original from the temporary store/backup.

Estimate
--------
Effort: 3-5 engineer-days (development + tests + canary validation)
Risk: Low-Medium (possibility of removing useful inline detail if heuristics too aggressive)

Dependencies
------------
- Phase 0 completed (telemetry + backups + canary ready)
- Access to request-build pipeline code paths and tool adapters
