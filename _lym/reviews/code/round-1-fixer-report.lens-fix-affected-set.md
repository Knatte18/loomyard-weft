No findings.

The sets I checked:

- **F1, seeding sites:** The only status seeding is `seedAbsentStatus` (`internal/battencli/arm.go:823`, Status literal at `:836`). Its only two callers, `battenPreRun` (`:793`) and `battenPreStepUnmarked` (`:1006`), both go through `seedFreshStatus`.
- **F1, CommitStatus wiring:** `battencli` sets `CommitStatus` in one place, `wire.go:1721`, and it now reuses the `c.commitStatus` seam instance. The other `CommitStatus:` hits are loom and landing wiring and tests, which are out of scope.
- **F2, marker:** `commitStatusMarker` is read and written only in `commitstatus.go`, at `:231` and `:268`. `shedrun.LastCommitMarker` has no other consumer, so no other reader sees the new `push_pending` key.
- **F5, wording:** Every live "before the run acts" site carries the start-row exception: `PATTERN.md:24` and `:33`, `pattern/PATTERN-pair-remote.md:3` and `:20`, `pattern/PATTERN-run-fsm.md:4` and `:40`. The remaining hits are `docs/research/...` records and `PATTERN-run-fsm.md:18` and `:20`, a heading and a rule line that its opening already qualifies. All are untouched by the fix.
- **F6, `statuscommit.New` callers:** There are three, `loomcli/wiring.go:345` and the two cores in `commitstatus.go:242` and `:251`. The reworded `Deps.Push` doc is consistent with all three.

Unfixed by design and outside this lens: `_lym/landing/summary.md` still carries the "untouched" and "pushes once" claims that the review asked the fixer to correct. It sits under `_lym`, which the fixer's rules put off limits.

Tool calls made, three Bash searches:
1. One grep for `seedAbsentStatus`, `seedFreshStatus`, `CommitStatus:` and `commitStatus` across the Go files. My context showed `arm.go`, `wire.go` and `commitstatus.go` edits but not the other `CommitStatus` wiring sites in the repo.
2. One grep for `commitStatusMarker` and `LastCommitMarker`. My context did not show every marker consumer in the repo.
3. One grep for "before the run acts" and the start-row wording across the `.md` and `.go` files, to find sibling docs the fix left as they were.
4. One grep for the `NameWorktreeCreate` Status literal and the `statuscommit.New` callers, to find any other seeding site or core caller.
