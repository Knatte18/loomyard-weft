# Fixer report — plan, round 1

Every finding's premise held against the plan and the code.
F1–F8 and F10–F12 are fixed as proposed, F3 and F9 fixed differently in part, as marked.
`lyx loom validate-plan` reports `ok: true` with informational findings only, before and after the self-check edits.

## Changes

- **F1** (BLOCKING), fixed as proposed.
  Card 5: `RunCapture` no longer holds `CaptureJobName` through `proc.HoldNamedJob`, releases only the slot and the lock, and says why; `internal/proc#HoldNamedJob` left its Uses.
  Card 7: the hidden `capture-child` holds `proc.HoldNamedJob(driftengine.CaptureJobName(worktreeRoot))` around `RunCapture` and releases it as its last act; `HoldNamedJob` and `CaptureJobName` joined its Uses.
  Card 9 and card 14's `driftDeps`: the foreground `RunCapture` runs in the step process with no named job, its command in the step's `StartTied` job tree.
  Card 13: only the detached child holds the named job.
  Overview: the Windows-job Shared Decision applies to card 7 too.
- **F2** (BLOCKING), fixed as proposed, with the probe gated on the agent-session mark of F9.
  Card 11: a new step 2 builds the verify table and calls `Deps.Seats.Probe` while `AgentSessionOpen` reads open; a live chair skips to the seat producer's resume, so neither the fresh-spawn preparation nor `ResetRuns` runs. Steps renumbered 1–10, with their internal references updated.
  Gating on the mark keeps the probe off a first entry, where no compare log exists for `VerifyTable`'s required input; the probe matches whatever compare log it is told, since `probeSeat` builds its spec with an empty prompt (`internal/seatengine/resume.go:67`).
  `TestDriftVerify` gains the live-chair row, plus rows for an open mark with no live chair and a mark closed after the agent's end.
- **F3** (MEDIUM), fixed differently in part.
  Card 11 step 7: the verify half runs only on the resume from this row's own dirty-tree or test-gate Stuck (history test over the last entry, reasons opening with package-constant texts) and only when `AttemptHead` is found and differs from HEAD; a first entry and a reroute run no verify gate.
  Not made: skipping the dirty-tree half on a first or reroute entry. The premise concerns the verify gate only, and without the dirty check a passing mechanical compare (step 8) would return Done over uncommitted code; the dirty check stays on every entry that reaches step 7.
  Test rows replaced as proposed: the test-gate resume row and the reroute row, plus a first-entry row.
- **F4** (MEDIUM), fixed as proposed.
  Card 14: `internal/loomrecipe#TestRouting_ProgressAt` and `internal/reportengine/doc.go` under Edit, with the Intent sentences for both; the ImpactSummary drops its test count.
- **F5** (MEDIUM), fixed as proposed.
  Card 5's dirty-tree refusal names restoring the paths to HEAD or stashing them, never removing or committing them, then `lyx loom resume` at `Drift-Baseline` and `lyx drift capture` then `lyx loom resume` at a `Drift-Await` halt, beside `lyx drift defer` then `lyx loom resume`; `TestCheckMainCapture` asserts the wording at both rows.
  The overview Shared Decision and card 9's sentence and test row carry the same wording.
- **F6** (MEDIUM), fixed as proposed.
  Card 9: `Deps.Commit` takes anchor-relative paths; without `WaitCapture` it commits only `scope.json` and `baseline.md`, and with it or on the fallback all of `DirRel()` after the capture ended; `TestDriftBaseline` asserts the paths per mode.
  Cards 10 and 11 commit all of `DirRel()`; card 14 builds `Commit` over the told paths. `DirRel` moved from card 14's Uses to cards 9, 10 and 11.
- **F7** (MEDIUM), fixed as proposed.
  Moved to Edit: card 3 `internal/gateslot/template.yaml` and `internal/gateslot/doc.go`; card 5 `internal/runproc/doc.go` and `internal/gateslot/doc.go`; card 6 `internal/gateslot/doc.go`; card 11 `pattern/PATTERN-gate-slot.md` and `internal/shedadapters/doc.go`; card 13 `internal/pairteardown/doc.go`, its Extends group removed.
- **F8** (MEDIUM), fixed as proposed.
  Card 7: `TestCapture_OutlivesCaller` declares a fake compare and gains the two `ErrRunsSpent` rows (agent wording with the session mark open, caller wording at `blocked`), each asserting `runs.json` unchanged.
- **F9** (LOW), fixed differently.
  Proposed change not made: driftcli asking the seat runner's `Probe` over the verify table. `VerifyTable` refuses an empty required input (card 8), its inputs (stencils, registry, slug, record path, the compare log) come from loom's wiring (card 14's `driftTables`), not from driftcli, and `Probe` validates the whole table (`internal/seatengine/resume.go:29`).
  Instead, card 2's `runs.json` carries an agent-session mark: `ResetRuns` opens it at the fresh spawn, `CloseAgentSession` closes it, `AgentSessionOpen` reads it. Card 11 closes it at an entry finding no live chair and at the agent's end. Card 7's `VerbStatus.AgentSession` reads it; a `running` status at `Drift-Verify` with the mark closed refuses a caller's accept, naming the wait for the halt and the parent's ruling. Rows were added to `TestTakeRun` and `TestAcceptWithdraw`.
  The overview's Shared Decision "run status decides the caller" states the mark and applies to card 2.
- **F10** (LOW), fixed as proposed, then reworded in the self-check.
  Card 5: both `StartCapture` errors end with a way forward; `TestStartCapture` gains the record-wait-bound row.
- **F11** (LOW), fixed as proposed.
  Card 3: `internal/gatecli/doc.go` under Extends, its example gaining `drift-verify: {tiers: [0, 1], scope: diff}`.
- **F12** (LOW), fixed as proposed.
  Card 7: `capture` carries `operator`, and its `Long` names `--side` as the `Drift-Verify` agent's form, which card 8's stencil names; the `Long`s of `accept` and `withdraw` name the parent's use at a halt.

## Self-check

- fix-consistency MEDIUM, and fix-proposal MEDIUM, merged: card 5's one `StartCapture` way forward names `lyx drift defer`, which refuses at the row's error stop. Fixed with own change: the wording is keyed on `FallbackDetach`. The `lyx drift capture` caller is told to wait for `lyx drift status` to show the lock free (which also covers fix-proposal's slow-child case), then run `lyx drift capture` again or defer; the row is told `lyx loom resume`. The test row covers both.
- fix-consistency MEDIUM, and fix-claims MEDIUM, merged: step 10's exhausted-entry Stucks were never said to open with step 7's halt texts, so their resume skipped the verify gate. Fixed as proposed: step 10 opens those reasons with the entry's halt text ahead of the seat producer's gate reason, step 7 says both share the constants, and a test row was added.
- fix-consistency LOW: step 7's "on every entry" contradicted step 2's skip. Fixed as proposed: "on every entry that reaches this step".
- fix-consistency LOW: the overview rationale said the mark closes at every entry. Fixed as proposed: it is kept by a reattaching entry and closed by one finding no live agent, and at the agent's end.
- fix-consistency LOW: card 7's audience lead-in was stale after F12. Fixed as proposed: the lead-in names `capture`'s `operator` audience.
- fix-claims LOW: the F2 test row must script a live chair for both probes. Fixed as proposed: the row says so.
- fix-affected-set LOW: `internal/loomshed/doc.go:40`'s per-row verify-key sentence was missing `drift-verify`. Fixed as proposed: card 11 puts it under Edit, and its Intent adds the key.
- fix-proposal LOW: the capture child could write to its log after `RunCapture` freed the lock. Fixed as proposed: card 7 says the child writes nothing to its log after that return.
- fix-proposal LOW: `TestRouting_ProgressAt`'s tail subtest name went stale. Fixed as proposed: card 14 says the name gains `Drift-Verify`.

## Disputed

None.

## Deferred

None.
