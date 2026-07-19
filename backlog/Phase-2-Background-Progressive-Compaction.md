Phase 2  Background progressive compaction

Objective
---------
Introduce background workers that progressively compute summaries for sessions (especially tool-heavy sessions) so that compaction work is amortized over time and on-demand compaction is cheaper and faster.

Components affected
-------------------
- Scheduler / cron / background worker infrastructure
- Compression orchestrator (worker integration)
- Telemetry (compaction events, tokens_before/after)

Acceptance criteria
-------------------
- Background worker can be started/stopped via feature flag on canary.
- Background compaction reduces tokens_est for sessions that have been processed by workers vs unprocessed sessions.
- Background compaction events emit telemetry including tokens_before/after, messages_affected, summary size.

Tests
-----
- Schedule background compaction on a batch of canary sessions and verify that summaries are written and tokens_est decreases.
- Validate that on-demand compress after background pass is lightweight and does not overflow provider limits.
- Monitor worker resource usage and ensure it stays within acceptable bounds.

Rollback plan
-------------
- Stop background workers and clear any in-progress job queue items.
- Revert any schema changes related to background compaction indexing.

Estimate
--------
Effort: 5-10 engineer-days (worker, scheduling, telemetry, safety gates)
Risk: Medium (resource usage and coordination complexity)

Dependencies
------------
- Phase 0 and Phase 1 completed
- Access to job queue infrastructure and permission to run background processes
