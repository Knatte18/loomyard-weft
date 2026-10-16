# Fixer report — discussion, round 1

## Changes

- F1 (BLOCKING), fixed as proposed, taking the review's second option. D4 now states what dropping the snapshot lets through: a batten tackler's writes in the prime still have their verdict acted on, and a `resume` may re-enter Land or Worktree-Teardown over them. It also states that the stop-key check covers only a moved run, and that the only bound is the stencil's write rule, which is prompt text. The D4 Bound line carries that bound. A new acceptance criterion requires the rendered batten prompt to carry the write rule. The narrowed snapshot was not chosen: nothing read this round shows that the prime's other writers leave no tracked dirty paths outside the records side (support-log, round 1).
- F2 (BLOCKING), fixed as proposed, taking option (a).
  - D5 states that `Migrate` runs on every reconcile and on `run_settings` (`internal/configreg/runsettings.go:60`).
  - The rewrite fires only when all four keys hold the old defaults together, each with the old template comment. It writes the new comments, so it fires at most once per file.
  - `run_settings` documents are marshalled from JSON (`runsettings.go:56`), carry no comments and never match.
  - D5 states the bound for a hand-edited file.
  - The loom template's four keys, `darn.yaml`'s four and `batten.yaml`'s three ship with the inherit value.
  - The Migrate criterion now covers once-only firing, files that do not match and a `run_settings` pin. A template criterion is added.
- F3 (MEDIUM), fixed as proposed. D5 says each run's tackler keys come from its own recipe's override: `lyx loom drive`'s wiring picks by the seed's recipe, and batten's CLI wiring passes `batten.yaml`. The darn criterion is added.
- F4 (MEDIUM), fixed as proposed. D2 gains the resume goto targets seam, and D2's rationale names `tackler.go`'s goto check. D4's resume sentence points at the seam, D7's goto rule reads from it, and the batten criterion gains the refused routed-from goto.
- F5 (LOW), fixed as proposed. D2's Bound line is rewritten. D2's opening sentence, which made the same overclaim, now says an unwired seam removes a row, an agent or a check.
- F6 (LOW), fixed differently. The proposed D3 sentence says "loom and darn runs are all seeded llm" and "no run has driver records today", and both are false.
  - Live loom seeds record `"driver": "go"`, this task's own included (`twb-btf-kt/_lyx/shed/twb-btf-kt/seed.json`), and `.lyx/loom/driver.log` files exist in live pairs.
  - The driver lock, park marker, resume report, drive reports and tackler directory already sit at `shedrun` per-run-id paths at HEAD (`internal/shedrun/paths.go:164-267`).
  - D3 now says that only the settle records and the driver log move, without a migration, and what a run in flight across the deploy loses: its spent re-steps and verify retry, counted from the settle records.
  - D3's records sentence is recast so that it no longer reads as a move list.
  - D4 gains the old-build batten sentence. The proposed adoption wording was replaced after the self-check; see that section.
- F7 (LOW), fixed as proposed.
  - New criteria: a batten blocked stop after a build change goes to the tackler, not a hand-over, and a batten stop with a Publish failure record goes to the tackler, not a verify goto.
  - The prompt criterion requires that no arbiter is named and that the stop-kind list and the reading order are present.
  - The build-change criterion says "blocked", since only a stop the table sends to the tackler reaches it once the hand-over row is gone.
- F8 (LOW), fixed as proposed. A set `tackler_timeout_min: 0` inherits in a module and takes 10 in `shed.yaml`, so only `tackler_max_spawns` needs an unset-versus-0 representation. The overrides criterion covers both cases.
- F9 (NIT), fixed differently. The proposed fix replaces "status envelope" with `lyx batten run`'s envelope, but both envelopes exist. `battenStatusExtras` (`internal/battencli/arm.go:672`) carries `abandonedSession` and the done-child flag, and `battenPostRun` (`arm.go:647`) carries `abandonedSession`. The sentence now names both envelopes and the flags they carry, once.
- `support-log.md`: a round-1 entry records the picks for F1, F2, F6 and F8.

## Self-check

- claims, MEDIUM: D4's old-build sentence said the run lock serializes the two processes' steps and that a detached step is adopted. Fixed as proposed: at HEAD the old build holds the run lock for its whole in-process run (`internal/shedverbs/run.go`, `shedengine` `ErrShedBusy`). D4 now says the new build meets a busy step while the old process lives, and takes the driver lock once it has exited. The adoption claim is dropped.
- way-forward, MEDIUM: the same sentence, since the new driver exits on busy (`driverules.go` busy row). Merged with the claims finding and fixed as proposed: D4 says it exits through the busy row, leaving the run to the old process.
- consistency, MEDIUM: D8 and its criterion still said the tackler marker and driver lock move. Fixed as proposed: D8 says those reads already use `shedrun` and only the settle-records read moves. The rationale and the criterion follow.
- consistency, LOW: D3's records sentence read as a move list. Fixed as proposed: it now says every driver record is declared in `shedrun`.
- consistency, LOW: the in-flight effect on batten's publish-retry read was unstated. Fixed as proposed: D8 states the one-time extra Publish retry across the deploy.

## Disputed

None.

## Deferred

None.
