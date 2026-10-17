---
verdict: APPROVED
---

# Plan review, round 2

No findings.
The plan carries every Decision (D1–D5) and every Constraint of the decision record, and neither focus question turned up a defect.

## Focus question 1: `log --id` assertions

Every existing assertion on `log --id` output was checked against a scheduler that journals a `reminded` event for a carried or unread message.
Only a reminder whose `Unread` holds the filtered id changes a list, and only mail ids appear in `Unread`.

- `internal/creelengine/verbs_test.go` (`LogFilter{ID: ...}` rows): `newEnv` starts no scheduler, so no `reminded` event is journaled.
- `internal/creelengine/deliver_test.go`: the `LogFilter{ID: ...}` rows filter an inject id (lines 453, 474, 550, 782, 1145), which never appears in `Unread`.
  The one mail row (line 1065) forwards mail to `orch2` and then to `alpha`, which `without(e, alpha)` removes before `e.start`, so no live idle strand holds it.
  `deliverableReceiver` also never reminds an orch.
- `internal/creelengine/server_test.go`: the day-change row at line 969 counts only `carried` mail entries, and the row at line 1009 filters an inject id.
- `internal/creelcli/cli_test.go`: `log --id prints a carried entry's kind` is the one affected assertion, and card 1 names `internal/creelcli#TestCreelCLI` in its Edit list and fixes it.
  `beta` is registered live and idle, so the daemon's scheduler would remind it of the carried #1.
  Registering `beta` as `creelfake.DormantStrand` keeps the reminder out, because `worker.locate` returns not-live and `remind` is never reached.
- No other test, doc or suite file asserts on `log --id`: `tools/sandbox/SANDBOX-CREEL-SUITE.md:116` is the only other expectation, and card 1 edits it.

Card 1's new `LogByIDListsTheRemindedEventsThatHoldTheID` row follows the sequences of `PostToALiveIdleStrandIsRemindedOnce` and `UnansweredReminderBacksOffToTheCeilingAndANewMessageResets`.
Its expected lists agree with `Store.Log`'s other clauses: a `reminded` record carries `To` and no `From`.

## Focus question 2: verify coverage

The packages the cards target are `internal/creelengine`, `internal/creelcli` and `tools/sandbox`.
`tools/sandbox` embeds `SANDBOX-CREEL-SUITE.md` through `suite.go`, and it is the only package that embeds it.
The verify section runs all three untagged and runs `internal/creelengine` and `internal/creelcli` under `--tags integration`.
`tools/sandbox` has no build-tagged test file.
The `tmux` and `llm` files in `internal/creelcli` are not hermetic tiers a plan's verify runs; `PATTERN-test-speed` runs them at publish and by hand.

## Rejected

- Card 1's edit of `logCmd`'s help strings sits outside the decision record's In list.
  It follows from D4 changing what `--id` lists, and CLAUDE.md's docs-in-the-same-commit rule covers it, so it is not unlicensed scope.
- Card 1 says `server_test.go` runs no scheduler, while its rig does start a daemon.
  The conclusion holds regardless, since the daemon-backed row counts only `carried` entries for a mail whose receiver is an orch.
- Card 3 sets `done` for `release` without checking `resp.Released`, so a release that ended no hold says `the release took effect`.
  D2 of the decision record names `release` unconditionally, and only `interrupt` is conditional, so the plan follows the answer key.
