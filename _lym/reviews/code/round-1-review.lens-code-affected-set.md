```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. For each change, the set I found and whether the diff covers it:**

- **Green end no longer lifts holds or starts runs.** I searched Go, markdown and yaml for stale lift or green claims.
  - Covered: `docs/overview.md` (the adopt entry and the dropped "adopt check's green end starts a run's first pass" line), `internal/beamcli/cli.go`, `internal/battencli/owner.go`, `internal/landingshed/finalize_owner.go`, `internal/dobbyengine/passes.go` and `doc.go`, `internal/pauserecord/record.go` and `doc.go`, and `internal/adoptcli/doc.go` and `cli.go`.
  - No leftover `waiting_for_check`, `held_failed_check` or non-test `holds.json` reference exists.
  - Not covered: the one finding below, F1.
- **`AdoptCaller` and `Coverage.Held` widened to `AdoptCaller` or `HoldCaller`.**
  - Covered: the dobby's `RunView.Held` and its owner-probe guard (`passes.go:122`, `:193`), the tender report and the tests.
  - The dobby's owner lift (`beamcli/dobbydeps.go:501`) and the hibernate wake judge only `OwnerCaller`, and `pauserecord/doc.go:25-26` says so.
- **`PATTERN-dobby-runs`.**
  - Covered: the declared text in `PATTERN.md` matches the declared change verbatim.
  - Covered: the background has the `--no-orch` exception under "Dobby starts runs" and the rewritten "Adopt pause" rule, naming the legacy `adopt-hold` entry.
- **`PATTERN-daemon-wakeups`.**
  - Covered: the follow is listed in the background, and `cmd/lym/daemonwakeups_test.go` lists `internal/adoptcli/follow.go`.
- **`orchcli.BeamUpOptions.SkipOrch`.**
  - Covered: `beamUp` returns before `startOrch`, the field is documented, and `orchcli/doc.go` is edited.
- **`runproc.RoleAdopt`.**
  - Covered: `record.go` and `doc.go`.
  - Covered: the worker registers under `runproc.HubOwner` (`caller.go:223`).
- **`AdoptScratchDir` and `lymdirs.AdoptDirName` now also hold the record, lock and worker output.**
  - Covered: both doc comments are updated.
- **`lym beam status` adopt section.**
  - Covered: the `statusResult` field, the help text, and `adoptsection.go`.
- **Declared PATTERN change landed.**
  - Covered: the entry's add or change is in the diff as declared.
- **Commit messages.**
  - I saw no `Pattern-Departure:` trailer in the log subjects. The body lines showed nothing to judge.

**Findings**

- **F1, LOW, consistency.** `internal/pauserecord/doc.go:80` and `internal/pauserecord/hold.go:241,306` (consumed at `internal/tendercli/gate.go:84`) both key on `OwnerCaller` only.
  - `OwnerEntryExempt` lets a push child under its slug's parent lock pass an owner pause.
  - `innerrun_parentlock.go:36` releases a child's parent-lock record only for `OwnerHeld`.
  - D5 now converts an own-scope `OwnerCaller` entry into `AdoptCaller` in place, for moved runs and for adopted beam slugs.
  - Neither the exemption nor the lock release then applies to a converted slug. That is a changed behavior for runs converted while holding a parent lock.
  - Neither the diff nor `pauserecord/doc.go` states whether this is intended.
  - Evidence: `pauserecord/doc.go:25-26` names only the dobby's owner lift and the hibernate wake as judging `OwnerCaller`, and omits these two consumers.
  - Fix: either state in `pauserecord/doc.go` that `OwnerEntryExempt` and the parent-lock release deliberately ignore a converted entry, or extend both to `AdoptCaller` if a converted run must still release its parent lock.

I found no other member of the affected set that the diff leaves wrong.

**Tool calls (4)**
1. A grep for stale terms (`waiting_for_check`, `held_failed_check`, `holds.json`, and similar) across non-test files. The manifest had no repo-wide search.
2. A grep for lift and green wording in Go and markdown, plus the `docs/overview.md` diff. The manifest did not include that diff.
3. A grep for the stale terms in all files including tests, plus other `HoldCaller` references. I needed to see the tests and fixtures that assert them.
4. Reads of `battenshed/innerrun_parentlock.go`, `pauserecord/hold.go`, `beamcli/dobbydeps.go` and the `OwnerEntryExempt` users. These are the other consumers of `OwnerCaller`, which the manifest lacked.
