```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the change set (agreement of changed text with its surroundings).

1. LOW — `pushledger.Entry.HandOvers` (new field, F2) and `settleLevelEntry`'s widened keep condition: three descriptions of what a held entry holds were left as they were.
   - `internal/pushledger/doc.go`: "An entry holding only held records or dropped landings is held, not owed. Nothing can push such an entry before its pair goes or its commit leaves the parent." An entry holding only owed hand-overs is now also kept, and it ends when the hand-over succeeds.
   - `internal/runpusher/doc.go:43`: "A level target's entry is removed unless it holds landings, held records or dropped landings." `run.go` now also keeps one with `HandOvers`.
   - `internal/dobbyengine/doc.go:275`: the same two-member list.

2. LOW — `HandOvers` has a twin reader that was not updated: `internal/dobbyengine/pushsummary.go:81` lists an entry as held only for `entry.HeldRecords != "" || len(entry.Dropped) > 0`. An entry holding only an owed (failed) hand-over is listed neither as owed nor as held, although its `held_back` record stands and `parentTargets`, `hasParentWork` and `settleLevelEntry` all gained the `HandOvers` case.

3. LOW — F3 `restartHubPushers` now starts each unrefused beam pair's pusher, but `internal/beamcli/doc.go:230` still says the dobby "restarts the prime's pusher at the start of its `Tick`, the one hub-wide pusher restart."

4. LOW — F9: `internal/webstercli/doc.go` (the reset section, around line 32) describes the reset's records commit and its `Run-Transition` trailer but not that the verb now pushes it synchronously, reports `push_error` and leaves a failed-push record, which the verb's help, `PATTERN-pair-remote` and `PATTERN-run-fsm` now state. It also now refuses on a hand-over after the branch was reset, which the section does not mention.

Tool calls (5 of 6):
- grep for `Dropped`/`HeldRecords` readers plus `cat internal/pushledger/doc.go`: other readers of the ledger entry were not in context.
- grep for retired symbols and "detached" wording: to find twins of the F4/F8/F15/F17 changes outside the files read.
- grep for webster-reset and protected-branch descriptions: sibling docs of F9/F5 not in context.
- sed of `pushsummary.go`, `dobbyengine/doc.go`, `webstercli/doc.go`, `pairteardown/landingstands.go`: the twins the first greps surfaced.
- grep of `beamcli/doc.go` and battenshed/battencli docs: tick and teardown descriptions were not in context.
