Epic: Hermes Memory Architecture v2

Goal
----
Evolve Hermes memory architecture to prevent unbounded context growth, restore long-conversation fluency, and introduce preventive persistence and lazy rehydration while preserving compatibility with existing sessions and tools.

Scope
-----
This epic contains the incremental migration phases (Strangler Fig) approved in the investigation. Implementation will be done incrementally via the listed phases. No implementation will occur from this document alone; it is a backlog tracker.

Backlog (phases)
-----------------
- Phase 0 — Preparation and Safety (pre-migration)
- Phase 1 — Pre-prune heurístic (mitigation)
- Phase 2 — Background progressive compaction
- Phase 3 — Pilot Artifactize-on-write (one tool)
- Phase 4 — Expand Artifactization to core tools

Owner: TBD
Priority: High

Notes
-----
Each phase is independent and contains acceptance criteria, tests, rollback steps and an estimate. Phases will be executed in order but each requires explicit authorisation before starting.
