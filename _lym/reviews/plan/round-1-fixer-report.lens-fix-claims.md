```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

**[BLOCKING] Card 15 does not exempt loom's own friction halt notes, so F1's Stuck-on-every-resume premise is still open**
- Changed text: `_lym/plan/15-drift-judge-seat.md`, the `Changes` bullet. It says the exempt paths are "`judgement.yaml` and everything under `RunRecordsDir`" and that "loom's own commits are never read as the judge's edits".
- Problem: loom writes more on the weft than the run's shed directory.
  - A `blocked` halt writes a Go-authored friction note. Per `internal/loomcli/halt.go:1-3` and `:46`, `writeHaltNote` records it as `loom-halt` in the friction directory.
  - `docs/overview.md`, in the `commit-records` paragraph near line 608, says friction notes live under `_lym/loom/friction/`, are committed with the run, and that every `lym loom start` or `step` commits the friction directory.
- Result: card 16's judge-change Stuck and snapshot-lost Stuck are blocked halts. Each leaves a halt note. On `lym loom resume` the note shows up in the weft's `Untracked` list, or in `WeftChangedSince` once committed. `Changes` names it, and the row stops again. This is the loop F1 was meant to break, for the "resumed after a judge-change Stuck" case the review names.
- Fix direction: add the friction directory (`_lym/loom/friction/`, in the same root-relative form) to the exempt set, or exempt every path loom's record commits write. Card 16's new test row should also cover a resume after a judge-change Stuck that left a halt note.

**Claims checked and found to hold:**
- Card 4: `templateConfig` is at `config_test.go:224`, `newVerifyFixture` at `verify_test.go:175`, `TestDriftVerify` at `verify_test.go:869`. The field names match.
- Card 2: `Repo.ChangedFilesSince` (`gitrepo.go:284`) reads committed history only and returns repo-relative paths, which matches "relative to its root".
- Card 15: `RunDirRel` (`shedrun/paths.go:417`) is anchor-relative. `l.AnchorRel` exists (`commitweftpaths.go:101`), so joining the two gives the root-relative form. `status.json` sits under `RunDir`.
- Card 9: `wait.go` polls from one second, doubling to thirty. `PATTERN-daemon-wakeups.md` has that "What counts" list.
- Card 25: the last callers of `CountedCapture` are `verify.go:170` and `verbs.go:258`. `doc.go:28` holds its line, and `TestCountedCapture` is in `records_test.go`.
- Card 16: `minimalEntries` builds DriftVerify with a `verify` gate (`conformance_test.go:91-96`).
- Card 10: `awaited_at` is set only by a passed Drift-Await or by a deferral (`records.go:77`).
- Card 19: `CaptureCurrent` holds the capture lock until it returns (card 9), so a check does contend for the lock `Defer` tests.

Tool calls:
- One Bash call (awk and grep): found which functions hold `verify_test.go:175` and `:869` and `config_test.go:224` and `:265`, and looked for loom's friction halt-note writer. My context had only the line numbers and no function names, and nothing on the halt-note path.
