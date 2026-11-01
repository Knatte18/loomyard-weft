```yaml
lens: guard-reach
outcome: reported
```

guard-reach lens: for each guard, refusal, branch or test row a card adds, can a real entry point reach it, and does the guarded site recur somewhere the card leaves unguarded?

- **MEDIUM — Card 5 (`Remove` / `Settle`), D8: the record drop that keeps the action from running can be undone by Settle's heal.**
  For a released front task that never started, `HasRun` is false, so `Remove` skips the cancel and the entry stays open (empty status) between step 2, the record drop, and step 3, the entry removal.
  In that window, `Settle`'s heal ("records … each open front entry of the project with no record", card 5) records the entry again from its `on_land`.
  The next tick then reads the entry as gone and runs `adopt-check`, which D5 and D6 mean the removal never to run.
  This is reached by card 12 step 5: the green end removes a released fix task right after its step 2 brought the subject's dobby up.
  Card 5's `TestRemove` and `TestSettle` rows do not pair the two calls.
  Evidence: `plan:internal/fronttask#Remove` steps 1–3, `plan:internal/fronttask#Settle` second bullet, `plan:internal/adoptcli#greenEnd` steps 2 and 5.
- **LOW — Card 12 step 3: the "`kind` is `shedverbs.KindBusy`" exemption cannot be reached.**
  `StartFirstPass` builds its envelope in `startFirstPass` from `output.Err(out, err.Error())`, which carries no `kind`.
  Its refusals there are only the covered, run-status and owner refusals.
  The driver-lock refusal happens inside the detached pass, after the envelope has already reported it started.
  A second start that races dobby's own pass at the lift therefore returns ok and is refused silently, never as a busy envelope.
  The `TestGreenEnd` row "a busy refusal returns no line" fakes a state production never yields.
  Evidence: `internal/battencli/cli.go:440-470` (`startFirstPass`), `internal/battencli/cli.go:633` (`StartFirstPass`).
- **LOW — Cards 4 and 5: the "nothing-left counts as the cancel step done" branch cannot be reached through `Remove`.**
  `Remove` runs `Cancel` only when `HasRun` reads true: a run status on the board, a status file, or a pair directory.
  Each of those makes `refuseNothingLeft`'s `otherLeft` true (`IsRunStatus(boardStatus)`, `recordsHere`, `parts.TaskWorktree`), so it returns nil before either nothing-left error.
  The `ErrNothingLeft` match and its `TestRemove` row are reached only through a fake.
  Evidence: `internal/battencli/cancel.go:556-575`, `plan:internal/fronttask#HubDeps` (`HasRun`).
- **LOW — Card 10: the input named for the owned-elsewhere refusal in `runCheck` cannot reach it.**
  The card says a dobby of the old owner's settle can spawn the check, but card 5's `Settle` drops records without running any action when `OwnedHere` reads false.
  The refusal is reached by a hand `lym adopt check <subject>`, or by a settle racing the ownership change; the card should name that input.
  Evidence: `plan:internal/fronttask#Settle` first bullet, `internal/adoptcli#runCheck` (card 10 Intent).
- **LOW — Card 14: `held_failed_check` is reachable in adopt's own envelope only after a failed check start.**
  Card 11 starts a check for every subject it recorded an owner for and every subject with a standing hold, so "this call started no check of the subject" holds only when that start failed.
  The card's `TestRunReports` row does not name that input.
  Evidence: `plan:internal/adoptcli#runReports` state list, card 11 "Checks:" paragraph.

Calls made:
- grep of `startFirstPass`'s refusals in battencli, and of hibernateengine's lift and caller handling: my context lacked the first-pass envelope's refusal shape and whether the hibernate wake lifts entries by caller.
- sed of `internal/hibernateengine/watch.go:60-100` and `internal/battencli/cli.go:440-470`: to confirm the hibernate wake lifts only the entry it owns, and that the first-pass envelope carries no `kind`.
