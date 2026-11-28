---
round: 1
focus:
  - "Reported path: trace `lym board sync` from the prime through the hub pusher restart (`internal/beamcli/dobbydeps.go`, `internal/battencli/dobbyreads.go`, `internal/beamcli/ownerarchive.go`) for a removed pair, both with the path gone and with a leftover directory. Does any read before or beside `Start`, `TargetOf` or `TargetRefused` still run git on that path, or log a warning? Does the change cover what the board entry's `Expected:` sentence asks?"
  - "`worktreeGone` now decides on `<path>/.git`, and `Run`'s `WorktreeGone` uses it too. Is there a state where a live or still-owed pair lacks that entry (creation, adoption, hand-over, repair, a removal in progress or one that failed partway)? Settle what a running pusher, a ledger entry and pending commits get in that state. Also check how an error from the stat behaves, for example when the path is not a directory."
  - "`TargetOf` now returns a pair target with nil error for a gone worktree, whatever the path was. Check every caller of `TargetOf` (`internal/loomcli/pusher.go`, `internal/loomcli/wiring.go`, `internal/battencli/commitstatus.go`, including uses that do not go on to `Start`): does each stay correct with that target, or did one rely on the error?"
  - "`TargetRefused` now answers false for a gone pair. Check each consumer (`internal/beamcli/dobbydeps.go`, `internal/loomcli/landingdeps.go`, `Run`'s deps in `internal/runpusher/run.go`): what does each do next on `false`, and is that right for a pair that no longer stands, including when it is the parent of another run?"
  - "`PruneLedger` now drops the entry of a leftover directory without `.git`. Check that against the change description's `## Merge danger` door and blast radius, and against what the other packages that read the ledger or call into `runpusher` see."
  - "Check the claims in `_lym/landing/summary.md` against the code, in particular which starts reach the silent skip, how often the old warning was logged, and why `PairPresent` is left reading the directory alone."
  - "Do the comments and docs that describe a worktree or pair as gone, removed or no longer existing still match the new definition (for example the `WorktreeGone` field doc in `internal/runpusher/run.go`, and any repository doc or `pattern/` entry describing the pusher or its ledger pruning)?"
---

Seed focus for round 1, taken from the committed range, the board entry under `## Goal` and the change description.
No finding has been judged.
