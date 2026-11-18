---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: >-
      The fix key reads the trace id from the loop's delivered envelope or an interrupted stop's record the re-step already reads, but the re-step reads neither:
      the pid file is gone after an envelope and the loop id is unknown at the re-step.
      The loop instead reads the newest kind-interrupted record in the steps directory, when the status reads failed with the text interruptedError writes,
      and calls the same OnStepOrphaned hook before its first child (internal/shedverbs/loop.go, reconcileInterruptedStep).
  - id: F8
    disposition: fixed-differently
    reason: >-
      The fix key asks for a removal assertion on the cleared-pane-binding row.
      A cleared binding is tracked, not live and not an own-window present pane, so strandRemovable is false and no stop removes it (internal/shuttleengine/stop.go strandRemovable);
      the row asserts the resume and that no removal happens.
      The dead own-window pane row asserts the removal.
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed-differently
    reason: >-
      The fix key offers either renaming prompt.md to a kept name or documenting the replacement; the documented form was chosen, since a rename adds a file operation with its own failure paths.
      relaunchForReprompt's doc comment, doc.go's gate paragraph and the landing summary state that a resume replaces prompt.md with the re-prompt.
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed-differently
    reason: >-
      stopStepRuns is a method on loomCLI, so a test cannot substitute it.
      A stopRuns seam field on loomCLI (nil means the runner stops them, like the receiver's other test seams) lets the new specFor subtest assert the hook's steps directory and trace id.
  - id: F15
    disposition: fixed
---

# Fixer-report, round 1

## What changed

Commits, grouped by file because several findings share a file (hunks of one file could not be committed per finding without interactive staging):

- `shuttleengine: fix F1, F2, F3, F4, F6, F8, F10, F11` and a follow-up `fix F2, F4`:
  - F1: `Run.startupEventsOffset`, set by `relaunchForReprompt` to the events file's size before `Prepare`; `checkLivenessTick`'s events proof and `sessionStartRecorded` read only past it; the `checkLivenessTick` doc states the new rule.
  - F2: `Spec.SkipSkillLoadTurn`; both engines' `Prepare` keep the pointer on the launch line and return no `PromptLine` while `Skills` still shape scope, settings and passages; `relaunchForReprompt` no longer clears `Skills`.
  - F3: `copyManifest` after the record save.
  - F4: `Prepare`, `AddStrand`, record save, then `removeStrandIfPresent` on the old strand with a Warn on failure; `removeGoneStrand` is gone.
  - F6: a non-empty `checkResume` warning refuses the resume before `Prepare`.
  - F8: `TestGate_RepromptToGoneStrand` rows for a dead own-window pane, a cleared binding, a never-ready resumed provider (events file already holding a session start and turn start),
    and failures of the holder lookup, `CheckResume` (error and warning), `Prepare`, `AddStrand` and the record save; the last two keep a removable old strand to pin F4's order.
  - F10: `run.state` is copied before the assignments and restored when the save fails; the save-failure row asserts `Result.StrandGUID` stays the old strand.
  - F11: documented, see above.
  - Engine-level pins: `TestPrepare_SkipSkillLoadTurn` in `claudeengine` and `codexengine`.
- `shuttleengine: fix F5, F12`: `StepStopReport.Held` and `Finished`; `StopStepRuns` leaves a tied run whose strand is untracked or has a cleared pane binding running when its engine cannot rule out a live holder; a run still reading running after its stop is listed as finished.
  Doc comments and `doc.go` follow; `TestRunner_StopStepRuns_StopsOnlyTheStepsRuns` gains cleared, held and finished rows.
- `shedverbs: fix F7, F9, F15`: `reconcileInterruptedStep`, a loop test for it, a panic-with-child-in-flight subtest and an empty-note subtest.
- `loomcli: fix F14, F5, F12`: `stepRunsNote` renders `Finished` and `Held`; `stopRuns` seam and the `specFor` wiring subtest.
- F13: `_lym/landing/summary.md` reworded, with the other changes folded in; it is outside the git tree and not committed.

A gate repair followed: the comment lines this round added were rewrapped at sentence boundaries, one sentence per line, with each over-long line split or rewritten as shorter sentences.
It touched comments only, in the files the round changed, and changed no behaviour.

`lym gate test` over the five touched packages passes.

## Disputed

No review finding is disputed.
Two fork findings are disputed on ground `premise false`, listed in the self-check below: the manifest skill-load items on a resume, raised by fix-proposal and fix-affected-set.

## Deferred

None.

## Rulings

None reached this round.

## Self-check

- fix-code-correctness, MEDIUM, F4 test did not pin the removal order: fixed (rows with a removable old strand and a failing add or save).
- fix-code-correctness, MEDIUM, F2 engine guards untested: fixed (`TestPrepare_SkipSkillLoadTurn` for both engines).
- fix-code-correctness, LOW, F7 selection not tied to an interrupted stop: fixed (status must be failed with the interrupted text; a producer-error row added).
- fix-consistency, LOW, `LoopSpec.OnStepOrphaned` doc stale: fixed.
- fix-proposal, MEDIUM, manifest skill-load candidates on a resume: disputed, premise false. Both manifests keep a candidate only when its proof is in the launch line,
  so a resumed launch's manifest drops the skill-load items and claims no delivery (`internal/shuttleengine/claudeengine/manifest.go:100`, `internal/shuttleengine/codexengine/manifest.go:101`).
  No `omitted` item is produced.
- fix-affected-set, MEDIUM, the same manifest point for both engines: disputed on the same ground.
- fix-proposal, LOW, F6 refuses every resume on one corrupt registry entry: no change; the reviewer chose the strictness and it ends the loop with a clear naming, the safe outcome.
- fix-proposal, residual, F4's `AddStrand` beside a still-tracked old strand: an explicit `NameOverride` already held by the old strand is refused unless reed reclaims it as dead (`internal/reedengine/strand.go` `strandNameLocked`, `reclaimDeadNameLocked`).
  The refusal ends the loop with both failures named and leaves the record on the old strand, the safe state; no change.
