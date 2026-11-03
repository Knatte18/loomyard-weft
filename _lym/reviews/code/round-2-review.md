---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/preflight/preflight.go:19
    summary: runStateExemptRels and the preflight doc.go paragraph name a pair-teardown commit as the carrier of the exempt files, but pairteardown commits nothing, and no status pathspec stages the stop markers
    fix: In internal/preflight/preflight.go (runStateExemptRels comment, lines 19-20) and internal/preflight/doc.go (the new paragraph, lines 25-28), drop the claim that the pair teardown's commit carries the rest. State what the code does, namely that the status commits stage the settle records (statusCommitPathspec and battenRunCommitPaths) while the other run-record files, the stop and condition-stop markers among them, stay uncommitted until a later records commit or the run's removal, and that the check ignores them because they are lym's own in-flight bookkeeping. Keep the "decided in one place" sentence.
  - id: F2
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md
    summary: change description omits the settle-record staging the diff adds and names a landing records commit that does not carry a task pair's files
    fix: In _lym/landing/summary.md, add that both status commits (statusCommitPathspec in internal/loomcli/wiring.go, battenRunCommitPaths in internal/battencli/commitstatus.go) now stage the settle records through shedrun.SettleRecordsRel, and reword the design decision that calls committing the settle log an option not taken. Replace "the landing's records commit" with the real carrier of the other run-record files, or state that nothing commits the stop markers and the exemption covers them.
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shedrun/paths.go:349
    summary: RunsRootRel doc enumerates its consumers and already misses parentRecordsCommitter, and the SettleRecords doc says the run directory is staged though the status commits stage the file by its own path
    fix: In internal/shedrun/paths.go reword the RunsRootRel comment to say what the path is and that callers needing the whole run-records tree use it, without naming fabricengine's Add or Preflight. In the SettleRecords comment replace "so a run's records commit stages it with the run directory" with a statement that a status commit stages it through SettleRecordsRel.
---

### [MEDIUM] Exemption comments name a carrier that does not exist

**Location:** `internal/preflight/preflight.go:19`, `internal/preflight/doc.go:25`

**Issue:** Both comments say the status commit carries the files it stages and that "the pair teardown's commit of uncommitted record changes carries the rest".
`internal/pairteardown` has no commit call.
Its only handling of uncommitted paths is the force-case snapshot under a local tag (`teardown.go:444`), which is neither a commit onto the records branch nor run on every teardown.
The focus question asked whether the code stages `go-settle.jsonl`, `stop-marker` and `condition-stop-marker`.
It stages the first only: `statusCommitPathspec` (`internal/loomcli/wiring.go`) and `battenRunCommitPaths` (`internal/battencli/commitstatus.go`) both add `SettleRecordsRel` and neither adds `StopMarker` or `ConditionStopMarker`.
`parentRecordsCommitter` (`internal/loomcli/landingdeps.go:619`) commits the parent pair's run-records root, not the task pair's.
So the comment is accurate for the settle log and wrong for the stop markers: nothing commits them in a task pair, and the exemption alone keeps Preflight from refusing them.
That satisfies the amended entry's "committed or excluded", but the docs state a mechanism the code lacks.

### [LOW] Change description does not match the diff

**Location:** `_lym/landing/summary.md`

**Issue:** The "Design decisions" section presents committing the settle log when written as an option not taken, and says the next status commit or "the landing's records commit" carries the files.
The diff does stage the settle log in both status commits, with `SettleRecordsRel` and matching test rows, so the description omits a change the diff makes.
The landing records commit it names is `parentRecordsCommitter`, which carries the parent's root and so not a task pair's stop markers.

### [LOW] Path docs in shedrun are partly stale or enumerate consumers

**Location:** `internal/shedrun/paths.go:349`

**Issue:** The `RunsRootRel` comment now names fabricengine's Add and Preflight's worktree-clean check as its consumers.
That is an enumerated consumer list, which `prose` forbids, and it is already incomplete: `parentRecordsCommitter` (`internal/loomcli/landingdeps.go:632`) and `RunsRoot` also use the path.
The `SettleRecords` comment's remaining claim, "a run's records commit stages it with the run directory", is only partly true: the status commits stage the file by its own relative path.

## Rejected

- Settle-log readers (`internal/arbiterstop/holdreading.go:90`, `internal/tendercli/deps.go:307`) read the file from the task's working tree through `shedrun.SettleRecords`, not from a commit, so an uncommitted log does not break a resume. The status commits now also stage it.
- The whole-root exemption instead of a per-file list matches the entry's "decided once in one place".
- `fabricengine.Clean`'s variadic `weftExempt` keeps fabric free of any shed path, so no fabric-isolation finding.
