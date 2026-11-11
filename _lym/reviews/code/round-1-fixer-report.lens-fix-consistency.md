```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the round-1 change set (siblings and twins of the changed text).

**Findings**

1. **LOW** — `internal/adoptcli/doc.go:208` (step pause section) and the `stepPause` doc comment in `internal/adoptcli/steps.go`.
   - The doc says "A slug the record lists as paused is skipped, so a slug an operator resumed stays resumed".
   - The changed loop now iterates `inputs.Paused`, so a listed slug is skipped for the pause write but reports `AdoptPaused`, `Run` and `PauseStanding` again.
   - That includes a slug whose entry the operator lifted, and its `PauseStanding` ("`lym tender resume X` lifts it") is then stale.
   - Neither the section doc nor the `stepPause` comment says the envelope restates the pause of listed slugs.
   - The loop's own inline comment is the only place it is explained.
   - Evidence: the step-pause doc line quoted above, and the new `for _, slug := range inputs.Paused` loop in `steps.go`.
   - Test `PausedSlugWhoseEntryTheOperatorLiftedIsNotPausedAgain` checks only `record.Live`, so nothing pins the envelope side.

2. **LOW** — The per-slug `WayForward` (`adopt.go:~441`) and the run-level stop from `stepDone` now name different commands.
   - The per-slug way forward in `takeOverSlug` still renders `lym adopt <project> --keep-local` for the project alone.
   - The record-level stop from `stepDone` now renders `lym adopt alpha beta --keep-local` for the whole adopt.
   - The test at `adopt_test.go:438` still asserts `lym adopt alpha --keep-local` in `alpha.WayForward`, while `wantErr` on the same row asserts `lym adopt alpha beta --keep-local`.
   - The per-slug command run alone would be refused by `judgeStanding` as other subjects of the standing record, which is the defect F2 fixed at the run level.
   - The same envelope therefore still carries a command the standing record refuses.
   - Whether to align it is the author's call, since the finding named only `stepDone`, and the two rows currently agree with the code.
   - The sibling stays inconsistent with the F2 premise.

3. **LOW** — `internal/adoptcli/doc.go:120–121` is unchanged, but it now sits beside a changed neighbour.
   - It says "Only the inputs of the steps from `deploy` on persist in the record".
   - The new sentence, "It keeps the steps listed, each replaced as it runs again", is consistent with it.
   - But the `reconcileWire` comment now states that the envelope's reconcile carries no mutations. A reader might expect the same for the kept steps' parts.
   - No contradiction; this note is only about reader clarity.

**Checked, no finding**

- The `adopt_test.go:433` `wantErr` update matches the new `stepDone` output.
- The single-slug rows at lines 447, 460, 966 and 848 remain valid.
- `adopt_integration_test.go:184` asserts only that both flags are named.
- `Request.Command` and `SameSubjects` agree with the new `stepDone` request.
- The F5 doc text agrees with the `pauserecord.Coverage` docs, which keep `OwnerHeld` on `OwnerCaller` only.

Tool calls:
1. `grep` over `internal` and `docs` for held-back re-run wording and the step-pause doc lines. I used it to find twins of the `stepDone` and `stepPause` changes, since my context lacked `doc.go` lines 195–210 and the other files that name the way forward.
