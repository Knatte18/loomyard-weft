```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked each bound, guarantee and claim about existing code in `decision-record.md`.

**Claims attacked**
- D1, `finishesAsSketch` keeps an arras run from making a Sketch: held (`internal/loomcli/wiring.go`, `finishesAsSketch` checks the `raddleFileEngine` row).
- D2, a re-armed perch gets the full budget again: held. `episodeStuckCount` ends the episode at the segment's last Done, and the prior cycle's approval is that Done (`internal/shedengine/run.go:630-680`).
- D2, Raddle-File's halt on its third failure bounds the cycles: held (`raddleMaxReturns = 2`, `sendBack`, `internal/loomshed/raddlefile.go:24-26, 556-565`).
- D2 and D4, with auto-accept off a spent budget never reaches Raddle-File, and the Bound lists every accept route: **broken** (F1).
- D4, Raddle-File still runs every proposal check and the board re-check in perch mode: held. `verify` runs Check and Recheck in every mode, and the hash match only in interactive mode (`internal/loomshed/raddlefile.go:191-217`).
- D4, a perch-mode accept files at most a round with open findings, and a BLOCKING one goes to the rule: held for the AutoAccept route (`blockingSendsToRule`, `internal/shedadapters/bouncer.go:1352`).
- D4 and D7, mode is decided by the rows: held. The mode is set once in state (`internal/loomshed/raddlewrite.go:89-94`), and a recipe change on a run-held entry is a scope change (`internal/boardengine/runlock.go:217`).
- D6, the admission lets through only the derived shape: held for shape. It is not bounded by producer or row (F3).
- D6, the status skip is keyed on a run-scoped mark: **broken** in the window between the board write and the mark (F2).
- D6, the raddle store is not a usable key: held. `Holds` matches the Sketch or a filed slug, and no run-clearing path removes the store (`internal/boardengine/raddlestore.go:45-52`).
- D6, a retry finds the converted entry as filed: held. `isRaddleFiledFrom` needs the discussed flag and the project body prefix, and the converted entry keeps both (`internal/boardengine/raddlefile.go:222-240`).
- D6, a failed write restores the entry's prior record: held as a decision. Today `FileRaddle` only removes the record files when `!saved`, which would delete the arras slug's own record (`internal/boardengine/raddlefile.go:125-129`), and that is what D6 changes.
- D6, `lym beam create` clears the done arras run: held as a decision. `beamcli` calls ClearMotifRun only for a done run with no pair (`internal/beamcli/create.go:195-201`).
- D7, the refusal from Raddle-File on, keyed by mode: held. Mode is always recorded before Raddle-File.
- D8, release only from the verb, never from `Board.SetStatus`: held. The other null-status writers (batten cancel's claim clear, fronttask Release, beamcli and pairteardown) all go through `Board.SetStatus` (`internal/battencli/cancelcmd.go:319`, `internal/fronttask/hub.go:46`).
- D8, a project's own status is empty: held. The draft rule applies only to kind task (`internal/boardengine/draft.go:42`), so a filed or converted project has no status.
- D10 rationale, a dependent left on the converted project never starts: held. Prune strips done members (`internal/boardengine/store.go:836-880`), and `metBelow` needs at least one member (`internal/boardengine/dependency.go:60`).
- D10 rationale, a dependent left on a done one-task arras entry starts early: held (`metBelow` counts a done entry as met).

**Findings**

**F1 — MEDIUM — design — D2, D4 Bound, acceptance criterion "D2: with the split perch's auto-accept key off, a spent budget does not reach Raddle-File"**
- Summary: the Bouncer's soft-cap rule auto-accepts a spent-budget round even with AutoAccept off, and D2's choice of no `carry_over` silently drops the open findings that accept would carry. So an unreviewed proposal with open findings files with no approval, and the record neither states nor bounds this route.
- Counterexample:
  - The perch spends its budget on a CONTINUE round whose ledger has only open findings not tagged design-wrong, such as a mechanical MEDIUM.
  - `settleUndecided` opens the window and calls `softCapRule` (`internal/shedadapters/bouncer.go:1358-1363`).
  - `softCapRule` finds no design-wrong keys, records an automatic accept and calls `acceptRound` (`internal/shedadapters/bouncer.go:1398-1404`). The Bouncer returns Done, so Raddle-File files.
  - The rule's own documented bound is the carry-over into `## Open risks` "which the recipe sets" (`internal/shedadapters/bouncer.go:1384-1387`). D2 sets no `carry_over`, so that bound is gone and the findings vanish.
- Effect: the acceptance criterion is false against existing code, and D4's Bound names only the AutoAccept route.

**F2 — MEDIUM — design — D6, the filing mark "which Raddle-File writes when its board write lands with a project"**
- Summary: a failure or crash between the board write and the mark lets a run status land on the converted project. That status is never cleared, and D8 then refuses to release the project.
- Counterexample:
  - `FileRaddle` saves `board.json`, then writes the store. A store write error returns an error after `saved = true` (`internal/boardengine/raddlefile.go:121-130`). Raddle-File cannot tell that the board write landed, so it writes no mark.
  - The row's failure transition makes loom's status hook call `SetRunStatus`, which writes any changed status onto a non-done entry (`internal/loomcli/wiring.go:220-225, 320-322`; `internal/boardengine/board.go` `SetRunStatus`). It writes a run status onto the converted project, and `RunHolds` now holds it (`internal/boardengine/runlock.go:55-61`).
  - On resume, the retry path (`AlreadyFiled`) writes only the store and the mark. The family outcome has no EndEntry, and the mark now suppresses every later status write, so the run status stays.
  - D8 then refuses the project's release, because the project carries a run status. A crash between the board save and the mark hits the same path.
- Fix direction: the record must place the mark before the board write, or key the skip on an independent fact, and state what the retry does with a status already written.

**F3 — LOW — design — D6, the run-lock admission and its Bound**
- Summary: the admission is judged by shape alone, at any row and from any writer, and the Bound does not say so.
- Counterexample: during Discussion-Write, a `lym board upsert` batch that writes the derived project shape, with members in the same write, passes `checkRunLocks`. Like `endsAsSketch`, the check reads only `before` and `after` (`internal/boardengine/runlock.go:163-166`). The write clears the run status and releases the run lock mid-run.
- Fix direction: the Bound should state that the admission can come before Raddle-File, and either limit it to a Raddle-File producer half or name this as accepted.

**Tool calls**
1. `budgetReached` in `internal/shedadapters/bouncer.go`: to see how the spent budget is judged, which bears on D2's re-arm claim.
2. Draft-status grep in `internal/boardengine/store.go`: to see whether a filed project takes draft status, for D8's "own status is empty".
3. Bounce-count wiring grep: to find the Bouncer's count source for the D2 re-arm.
4. `episodeStuckCount` in `internal/shedengine/run.go`: to confirm a perch Done resets the budget episode.
5. Prune grep in `internal/boardengine`: to test D10's rationale on how projects are closed and pruned.
6. `Store.Prune` and the `StatusDraft` grep (`internal/boardengine/store.go`, `internal/boardengine/draft.go`): to confirm prune strips dependencies and that the draft rule applies to tasks only.
