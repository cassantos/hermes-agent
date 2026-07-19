Phase 3  Pilot Artifactize-on-write (read_file)

Objective
---------
Introduce a minimal Persistence Manager + Artifact Store and pilot two-tier persistence for a single representative tool (read_file): store full file outputs in the Artifact Store and persist only summary+pointer in Conversation Memory.

Components affected
-------------------
- read_file tool adapter
- Persistence Manager (minimal API)
- Artifact Store (local filesystem or object store, minimal schema)
- Conversation Memory write path
- Rehydration API (simple fetch)

Acceptance criteria
-------------------
- read_file calls in canary produce an artifact_id and a concise summary/pointer saved in Conversation Memory.
- Fetching (rehydration) the artifact by id returns the original content.
- Tokens_est for sessions using read_file are reduced materially vs legacy.
- No data loss: raw outputs are durably stored and checksums verified.

Tests
-----
- Invoke read_file in canary and verify artifact_id and pointer recorded.
- Rehydrate artifact and compare checksums with original content.
- Measure tokens_est before/after for a set of sessions.

Rollback plan
-------------
- Revert read_file adapter to legacy behavior (persist inline) via feature flag.
- Clean up any pilot artifacts if required (or mark for archival).

Estimate
--------
Effort: 2-4 engineer-weeks (design + artifact store minimal + read_file adapter + tests)
Risk: Medium-High (storage, data migration handling, rehydrate UX)

Dependencies
------------
- Phase 0, Phase 1, Phase 2 completed
- Storage for artifact store and access control
