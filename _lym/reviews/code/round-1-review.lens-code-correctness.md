```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each branch and error path the diff adds or changes in `newCommitStatusSeam`.

**Branches and the cases I ran**

- **`writeMarker` closure (`commitstatus.go:224-228`).** I ran a successful write and a failed `state.WriteJSON`. A failure warns and returns, as the inline closure did at the base. Correct.
- **`startCore` (`commitstatus.go:238-242`).**
  - Mid-merge (`MergeActive` true or erroring): skips the commit and the marker write, like `core`. Correct.
  - Commit error: goes through `failureDisposition` (re-probe, then the error is returned). Correct.
  - `committed` false: marker written, no push. Correct.
  - `committed` true: marker written, the no-op `Push` returns nil. Correct.
  - Admit, Owed and Sleep are nil. `pauserecord.Admit` is the network gate (`internal/pauserecord/hold.go:462`) and `startCore` makes no network call, so skipping the gate is correct.
  - An owed push from an earlier transition cannot be pending here, because `core` retries an owed push until it lands (`statuscommit.go:124-128`). Correct.
- **`isStartRow` (`commitstatus.go:245-247`).** Run-Shed returns true; Seed-Child through Open-IDE return true; Worktree-Create, Land and Worktree-Teardown return false. Correct.
- **Start condition (`commitstatus.go:264`).**
  - Marker (Open-IDE, running) then (Run-Shed, running): `startCore`. Correct.
  - Marker (Run-Shed, running) then (Land, running): `core`, because Run-Shed is not in `RowsBeforeRunShed`. Correct.
  - Marker (Open-IDE, paused) then (Open-IDE, running): `core`. Correct.
  - Marker (Seed-Child, running) then (Seed-Child, failed): `core`. Correct.
  - Missing or corrupt marker: `core`. Correct.
  - A stale marker from an earlier run of the slug, then (Worktree-Create, running): `core`, since Worktree-Create is excluded. Correct.
  - Goto from Run-Shed back to a start row: `core`. Correct.
- **Failure path: a failed `core` push at the Worktree-Create transition, then resume.** Not correct; see F1.
- **Test rows.** I hand-computed every new row's commits and pushes against the code. All match: full chain 8/3, skipped gated rows 3/1, resumed start row 3/2, halted start row 3/2.

**Findings**

F1
- severity: MEDIUM
- class: design
- location: `internal/battencli/commitstatus.go:264`
- summary: Once the Worktree-Create transition's push fails, a resume runs the whole start without trying to push again.
- detail:
  - When `core`'s push fails with an error that is not owed (a rejection without a takeover, an auth failure), or the process dies between that commit and its push, the run stops at Worktree-Create.
  - `persistMarked` writes the status before it calls `CommitStatus` (`internal/shedengine/run.go:889-939`). Run then returns the error (`run.go:540-542`), leaving the status file and the marker at (Worktree-Create, running), not in a halted state.
  - On resume, the next persist is (Seed-Child, running). The marker still holds a running pre-Run-Shed row, so every transition from Seed-Child to Run-Shed takes `startCore` and pushes nothing.
  - So the run starts its inner run while origin has none of its records, and the push is first tried again at Land.
  - At the base, each of those transitions tried the push again and stopped the run again. That is the "any other push failure halts the run at the transition" rule in the file header (`commitstatus.go:10`).
  - The doc's "a resumed or halted start row pushes" (`commitstatus.go:213`) does not cover this case, because shed never records the halt as a state.
- evidence: `internal/shedengine/run.go:889-939` (persist, then `CommitStatus`), `internal/shedengine/run.go:540-542` (Run returns the error), `internal/statuscommit/statuscommit.go:122-125`.
- fix: The seam needs a durable record that the last core push did not land, and takes `core` while that record stands. One way is to write the marker with a pushed flag, or after the push only.

**Tool calls**

1. Read `pauserecord.Admit`, `shedengine/run.go` lines 375-410, and the `LastCommitMarker` sites. My context lacked what the skipped gate admission does and how a step error is persisted.
2. Read `shedengine/run.go` lines 455-480 and 540-575. My context lacked how a `CommitStatus` error from a transition persist leaves the run.
