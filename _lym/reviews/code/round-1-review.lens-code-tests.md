```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour the diff adds or changes, the test that pins it (code-tests lens).

**Per behaviour → pinning test**

1. `timedNote`/`formatWaitNote` Steady form for the shuttle wait and the running verify (no-detail and detail variants): `TestShuttleWaiting` rows "live shuttle marker", "live held marker", "live verify marker ahead…" assert `note.Steady` (`reviewstatus_test.go:191`). `TestVerifyWaiting` "live marker…" rows assert it via `steadyOf(note.Text, "6m")` (`:319-322`). Pinned.
2. `verifyWaiting` Steady for the paused and waiting-for-slot branches (the holders clause stays in Steady): the `TestVerifyWaiting` rows "paused marker", "waiting marker names its site and the holders", and "failing holders read" run the Steady check. Pinned.
3. `gateWaiting` Steady (per-record clauses, holders clause, `; rest.Steady` join): `TestShuttleWaiting` row 1 (two records with different elapsed, holders, next) and row 2 (gate wait plus verify) cover it. Row 1 uses `6m` and `3m`, so both clauses are checked. Pinned.
4. `parentLockWaiting` Steady: `TestParentLockWaiting` (`:327`). Pinned.
5. Fall-through notes carry `Steady` equal to `Text`: `fixedNextStep` and the `next` stub return it. The dead-marker and absent rows check `Steady` through `steadyOf(...)` (no elapsed → identity). Pinned.
6. The watch dedupes on the key and not the line (an elapsed-only change is suppressed): `TestPrintStatusLinesOnChange_LineSequences` row "ElapsedOnlyChangeIsSuppressed". The same row shows a real change (an attempt clause) still prints. Pinned. Wait-kind and detail changes reach the same compare, so nothing more is needed.
7. `statusWatchLine` line versus key, note start and end, hook error, and `UnavailableLine` as its own key: `TestStatusWatchLine_AppendsWaitingNote` (`status_test.go` hunk at lines 582-618) checks the key on the with-note, no-note and hook-error paths. The unavailable branch has no key assertion in the visible diff. `TestUnavailableLine_DedupesAcrossPolls` exists but only checks that the line is byte-identical. The `return unavailable, unavailable` branch is therefore unpinned, which is LOW.
8. The one-shot status keeps `Text`, with `core["waiting"]` and the human view using `waiting.Text`: `TestStatusCmd_WaitingNote_RunningOnly` still passes through the hook, with `Text == Steady` in the stub. The stub makes Text and Steady equal, so using `Steady` by mistake in `statusCmd` would not fail it. Weakly pinned, and a LOW gap.
9. `checklistPoll` uses `note.Text`: no test. `checklist_test.go:1146` calls `renderChecklist` with a literal `waitNote` and never calls `checklistPoll`. Switching it to `Steady` would show `<elapsed>` in the pane and fail nothing. LOW gap.
10. The spec, overview and help prose: static prose, exempt.

**Findings**

- **F-A: LOW, design.** `internal/shedverbs/status_test.go` (the `TestStatusCmd_WaitingNote_RunningOnly` hook at about line 699, stub returning `WaitNote{Text: note, Steady: note}`). The one-shot status path is not pinned to `Text`, because the stub gives both fields the same value. Fix: give the stub distinct Text and Steady and assert the envelope's `waiting` key and the human view carry `Text`.
- **F-B: LOW, design.** `internal/loomcli/checklist.go:169`. No test covers `checklistPoll` taking `note.Text` rather than `Steady`. This is the Goal's "one-shot status keeps the elapsed time" requirement as it applies to the pane. Fix: add a row or assertion where the hook returns distinct Text and Steady and the frame's wait line shows the Text.
- **F-C: LOW, design.** `internal/shedverbs/status.go:122-125`, the unavailable branch of `statusWatchLine`. Nothing asserts that its key equals the line. Fix: add an assertion on a spec whose status file is missing, expecting `line == key == UnavailableLine(label)`.
- **F-D: NIT, consistency.** `steadyOf` in `reviewstatus_test.go:255-261`. In `TestVerifyWaiting` and `TestParentLockWaiting` the expected Steady is derived from the same Text. It does not pass trivially, since a Steady that kept `6m` or changed any other part would fail. It would pass, though, if `render(elapsedPlaceholder)` and `render(formatElapsed)` diverged in a way the replacement happened to mask, and a leftover elapsed other than `6m` would go unnoticed. The only elapsed strings in play are `6m` and `3m`, so this is acceptable. A literal expected Steady in one row would remove the doubt.

**Tool calls**
1. Read `reviewstatus_test.go` 90-320: the test rows were not in my context (only the diff hunks were).
2. `grep` for `checklistPoll|waitNote` in the loomcli tests: to confirm no test drives `checklistPoll`.
