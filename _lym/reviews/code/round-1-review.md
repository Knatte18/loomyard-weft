---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/preflight/preflight.go:17-20
    summary: The comment, the doc.go paragraph and the summary say the run's next status or records commit carries the exempt files, but the status commit stages neither the settle log nor the stop markers.
    fix: "Make the claim true or reword it. In internal/shedrun/paths.go add SettleRecordsRel(l, runID), the anchor-relative form of SettleRecords, beside SeedRel and StatusRel. In internal/loomcli/wiring.go statusCommitPathspec, append shedrun.SettleRecordsRel(location, runID) when os.Stat(shedrun.SettleRecords(location, runID)) succeeds, as battenRunCommitPaths does for the seed, and do the same in internal/battencli/commitstatus.go battenRunCommitPaths. Then reword the runStateExemptRels comment and the internal/preflight/doc.go paragraph to say the exempt files are lym's in-flight bookkeeping, committed by the status commit where it stages them and otherwise by the pair teardown's commit of uncommitted record changes. If the settle log is deliberately left out of the status commit, drop the false claim and name what does commit it."
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shedrun/paths.go:337-349
    summary: The SettleRecords doc says a dirty-path check counts the file as run state, and the RunsRootRel doc names fabric's Add as its only purpose; Preflight's worktree-clean check now skips the root, so both read as stale.
    fix: In internal/shedrun/paths.go, edit the SettleRecords doc comment to drop the sentence "a dirty-path check counts it as run state" or restrict it to the hand-over's dirty-path check, and edit the RunsRootRel doc comment so it says the root is the path both fabric's Add drops from a fresh pair and Preflight's worktree-clean check skips.
---

### [MEDIUM] Exempt files are not carried by the commit the comments name

**Location:** `internal/preflight/preflight.go:17-20`, `internal/preflight/doc.go` (new paragraph), `_lym/landing/summary.md`.
**Issue:** `runStateExemptRels` and the doc paragraph say the run's next status or records commit carries the exempt files.
`statusCommitPathspec` (`internal/loomcli/wiring.go:99-117`) stages the status file, the reviews and loom durable directories, and the drive-report, arbiter and tackler directories.
It does not stage `go-settle.jsonl`, `stop-marker` or `condition-stop-marker`, and `commitRecordsVerb`, which the driver's `CommitRecords` calls, reuses that pathspec.
`battenRunCommitPaths` (`internal/battencli/commitstatus.go:68-77`) adds only the seed and the notice marker.
So the settle log stays untracked after the exemption, until `fabricengine.Remove`'s commit of uncommitted record changes or a manual `lym fabric commit`.
Before the change, Preflight's refusal forced that commit; now nothing does, and the settle log that arbiter and tender hold-readings read (`internal/arbiterstop/holdreading.go:90`, `internal/tendercli/deps.go:307`) can sit uncommitted on one machine while a resume starts on another.
The Goal asks that each file be committed when written or excluded from the check, and the exclusion is met, but the stated commit path is wrong.

### [LOW] Settle-record and run-root docs are stale after the exemption

**Location:** `internal/shedrun/paths.go:337-349`.
**Issue:** The `SettleRecords` doc still says a dirty-path check counts the file as run state, which Preflight's check no longer does.
The `RunsRootRel` doc names only fabric's `Add` as the reason the function exists, and Preflight now uses it too.

## Rejected

- Lens code-tests, LOW: `Clean`'s `weftExempt` parameter is covered only through `preflight.CheckResolved`. The public surface test pins the behaviour, and a fabricengine-level row would duplicate it.
- Lens code-tests, NIT: the test's file list is hand-written. The whole-root exemption makes a file added later safe, so the list cannot drift into a failure.
- Lens code-affected-set, LOW: the `Clean` doc comment does not say weft side only. The sentence names "weft-side check" and "anchor-relative weft paths", which states the scope.
- Lens code-correctness raised no finding; its pathspec cases at anchors `.`, a subpath and empty all hold.
- Halt note under `_lym/loom/friction` not exempt: `seedAndCommitBootstrap` commits the friction directory before Preflight (`internal/loomcli/sharedbootstrap.go:169-238`), and the summary records the choice, so it is no scope gap.
