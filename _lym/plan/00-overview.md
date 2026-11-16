---
format: 5
approved: true
language: go
---

# Plan: Weaver, the cold-start implementer with model escalation

Add Weaver as the `weaver` value of tapestry's implementer row.
One Go producer forms each batch from the remaining cards, runs it as one cold gated strand per attempt and climbs `weaver.yaml`'s model ladder on a failed attempt.
It records each batch with the shared post-batch checks of `internal/runplan`, keeps a run journal, commits its run directory and the plan after each batch, and ends with the plan-level verify and its fixer.
Its stops go through shed's stop chain; `lym weaver reset` starts a run over.
Every loom reader of implementer state reads the active implementer's through one seam in `internal/weaverengine`, and the plan-conflict record moves to one implementer-neutral path.

## Card Index

1 — shared-plan-conflict-path — move the plan-conflict record to one implementer-neutral path both implementers write
2 — weaver-config — start weaverengine with the weaver.yaml module: ladder, fixer timeout and verify gate key
3 — weaver-geometry — Weaver's run and scratch directories, its told geometry and the hubgeom builder
4 — runplan-shared-readers — move the done-card and begun-card readers onto runplan.State and add the remaining-card partition
5 — weaver-state — Weaver's tolerant state file with attempts, fixers, batch ends and the stop record
6 — active-implementer-seam — the seam that picks the active implementer, with D1's refusals and repair advice
7 — journal-and-report-paths — per-attempt report paths, fixer reply paths and the idempotent journal append
8 — weaver-stencils — the batch prompt and verify-fixer body stencils and their render
9 — batch-attempt-strand — the gated cold strand of one attempt, its verdict and its stop
10 — batch-record — end every attempt with its journal entry and the per-batch commit, checking a recordable one through the shared post-batch checks
11 — plan-verify-and-fixer — the run deps, the plan-level verify with its rerun, the gatefixer fixer and summary.md
12 — run-loop — the producer's loop over batches, the ladder, the stops and Done
13 — run-reentry-and-resume — re-entry after a stop and resume after a crash by attach and probe
14 — weaver-archive-and-reset — the reset engine and the archive of Weaver's run directory
15 — weavercli-reset — the lym weaver reset verb mounted on the root
16 — weaver-shed-producer — the shed adapter over Weaver's run with the shared plan-conflict claim
17 — implementer-guard — the D1 guard on both implementer rows and Weaver's lazy producer
18 — weaver-implementer-row — the weaver entry in the implementer table, its Env fields and loom's wiring
19 — plan-gate-reads-active — the plan gate and the rubric note read the active implementer's state
20 — renumber-reads-active — the renumber verbs bound and lease on the active implementer
21 — decision-notice-weaver — decision notices reach Weaver's strands and record each delivery
22 — describe-and-rework-archive — Describe and the PR rework archive cover Weaver's run directory, and the failure tracer reads the active implementer's state
23 — merge-in-warning — merge-in warns of the active implementer's run in flight
24 — arbiter-names-run-dir — the arbiter's description names the active implementer's run directory
25 — batten-fixer-timeout — batten bounds a Weaver verify fixer by fixer_timeout_min
26 — implementer-spec — the implementer contract names Weaver, commits during the run and the stop chain

## Shared Decisions

### Decision: seam-in-weaverengine

- **Decision:** The active-implementer seam lives in `internal/weaverengine`, which imports `internal/websterengine` to read Webster's state; no new package is added.
  Loom readers take `weaverengine.Active` where a refusal must stop them and `ActiveOrSetting` where they only advise.
- **Rationale:** The decision record's scope adds no package for the seam, and `websterengine` must not import Weaver.
- **Applies to:** cards 6, 17, 19 to 25

### Decision: weaver-reuses-the-implementer-role-and-segment

- **Decision:** Weaver's batch strands run under `agentconfig.RoleWebster`'s skills and `segmentcolor.Webster`; their reed role is `weaver`, and the verify fixer runs under `gatefixer.RoleVerifyFixer`.
- **Rationale:** Both closed sets are guarded by tests and the implementer row is one segment; the reed role alone tells the strands apart for the decision-notice guard and the reset's live-strand check.
- **Applies to:** cards 6, 9, 11
