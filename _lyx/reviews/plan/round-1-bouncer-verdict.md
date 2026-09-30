---
verdict: APPROVED
rationale: "All four round-1 findings are fixed in the plan text and hold against the code: card 4 now deletes ReplaceStrand's new-strand script only when its post-add SaveState and reconcileApplyPersistLocked (whose last step is SaveState, spawn.go) both fail, tests the AddStrand and ReplaceStrand failed-save deletes, and rewords emptycmd_integration_test.go; card 3's ReplaceStrand test uses render.AnchorHidden; no rubric-scope defect remains."
---

The round-1 report returned `APPROVED` with four findings, and the fixer report says all four are fixed.
I checked each fix against the current card text and against the code at `85902381e`.

- **F1 (ReplaceStrand post-add save).**
  Card 4 now lists `Engine.ReplaceStrand` and deletes the new strand's script only when the post-add `SaveState` failed and `reconcileApplyPersistLocked` also returned an error.
  That condition is exact.
  In `replace.go` the tail runs even after a failed add, and `reconcileApplyPersistLocked` ends in `SaveState`, so a `nil` from it means the new GUID was persisted.
  An error from it, whether from `listPanes` or from the save itself, means the GUID was never persisted.
  The delete therefore fires only on the decision record's "record never persisted" outcome.
- **F2 (AddStrand save-failure test).**
  Present: the `Enter` send-keys hook replaces `reed.json` with a non-empty directory after the script write and before `AddStrand`'s `SaveState`.
- **F3 (emptycmd doc).**
  `emptycmd_integration_test.go` is on card 4's Edit list, and the Intent says how to reword its file doc.
- **F4 (card 3 ReplaceStrand test).**
  The replacement spec is `render.AnchorHidden`, so `needsLaunchOnAdd` skips the launch and the test isolates the removal step.

Other checks came back clean:

- **Decision record.**
  Every Decision and Constraint has a card: `Source`/`ScriptExt` (1), the atomic `0o644` write and its degrade (2), removal and `Down` cleanup (3), the chokepoint wiring, failed-add deletes and docs (4), and the live POSIX proof (5).
  No card adds scope the record does not license, and no card deletes a script on `Resume`, on `UpdateStrand`'s surface, or on the `--if-absent` relaunch, where the records stay persisted.
- **Verify coverage.**
  `go test ./...` covers `internal/shell` and `internal/reedengine`, and the integration line runs both packages under `-tags integration`, which picks up card 5 and the existing reed integration tests.
- **ImpactSummary.**
  Card 1's claim holds: outside `internal/shell` the interface's only non-test users are `claudeengine`, `loomcli` and `reedengine`, and none of them implements it.
  Cards 2–4 each state a real blast-radius conclusion.
- **Granularity.**
  Cards 2–4 each form one testable unit.
  The deletion-before-write order and the same-commit docs rule justify the bundling in card 4.
- **Test helpers.**
  Every helper the cards name exists with a compatible build tag: `withInjectedExecutablePath`, `captureLogOutput` (which captures Info and above, so it sees `Warn`), `newTestEngine`, `newColdScratchEngine`, `waitUntil`, `sendKeysLiteralArg`, and the two existing tests the cards reference or retarget.
- **Enforcement.**
  `panebin_enforcement_test.go` keeps passing, because card 4 still calls `composePaneLaunchLine` inside `launchStrandLocked`.

The report has no `## Focus departures` section, so there are no departures to ratify or reject.
