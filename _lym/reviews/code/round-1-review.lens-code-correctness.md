```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function and branch with a concrete case.

**Per function and branch (case run, verdict)**

1. `drivable` (`passes.go:62`):
   - Paused with `ReachedParent` false gives false, so no pass. Correct.
   - Paused with `ReachedParent` true gives true. Correct.
   - "blocked" with `ReachedParent` true gives false. Correct.
   - Empty `State` gives false. Correct.
2. `planPass` entry guard (`passes.go:109`):
   - Cancel marker, in-flight pass or held driver lock on a paused reached run gives none. Correct.
   - `Held` on a paused reached run gives none, because the check is after `drivable` and before any pass. Correct.
3. `planPass` owned-elsewhere branch (`passes.go:119`):
   - A paused reached run owned elsewhere gets a full pass on an uncovered probe tick or at a lift, and none otherwise. This is the same rule as for a running run. Correct.
4. `planPass` covered branch (`passes.go:129`):
   - Paused reached run covered, probe tick: `State != running` gives none. Correct, since a continue pass would be refused anyway by `refuseContinuePass` (`arm.go:637`).
   - Lift: full. Correct.
   - Running run covered, `Producer == Run-Shed`, no pass owed: still continue. Unchanged.
5. `planPass` uncovered branch (`passes.go:138`):
   - Paused reached run at Run-Shed with `FullPassOwed` false and `OriginOwed` false: the new clause forces full instead of continue. Correct.
   - With `OriginOwed` it gives `passFullNoReopen`. Correct.
6. `drivePasses` `liftPending` delete (`passes.go:186`):
   - Covered paused unreached run lifts: the lift is dropped, since no pass can answer it. Correct.
   - `ReachedParent` read flips to false for one tick (Warn path) after a lift: the lift is dropped. The next probe tick with `ReachedParent` true gives a full pass anyway, so no pass is lost. Correct.
7. `drivePasses` busy and probe (`passes.go:196`):
   - A paused reached run keeps `d.busy` true while uncovered. If a pass exits 0 but leaves the run paused (a `pause_after` condition), it gets another full pass each probe interval. This is bounded by the probe interval and documented in the summary's risks. Correct, but see finding F1.
8. `DobbyRuns` read closure (`dobbyreads.go:185`):
   - Running, done or blocked run: skipped. Correct.
   - Paused run, reader error (for example `taskWorktreeLocation` fails or git fails in `FinalizePastMerge`): Warn, then `ReachedParent` stays false, and the loop continues. Correct, and not a swallowed error given the stated intent.
   - Paused run at Worktree-Teardown with the child status absent: `pastMerge` stays false, and `childReachedParent` returns true via the Worktree-Teardown rule (`cancel.go:722`). Correct.
   - The second argument is always `true`. `run.status` was just read, so found is true. Correct.

**Findings**

- F1. LOW, `design`. `dobbyreads.go:191`: the Warn for an unreadable reached-parent judgment fires on every dobby tick for each affected paused run. `DobbyRuns` runs each tick and nothing dedups, so a persistent failure (a missing task worktree while the child status still exists) floods the log. Other noisy reads in this package use a `Fail` and `Recover` pair (`d.noticeFailures`, `runnotice.go:161`). Evidence: `dobbyreads.go:185-196`, `runnotice.go:159-164`.
- F2. LOW, `design`. `DobbyRuns` is also called by `DobbyViewReader.Read` (`dobbyreads.go:378`), whose doc says it "writes nothing". The new read adds git and settle-record reads for every paused run on each status view, and a Warn for each failure. This is not wrong, but the cost is paid by status readers that never act on the value. Evidence: `dobbyreads.go:363-378`, `cancelcmd.go:377-391`.
- F3. MEDIUM, `design`. `lym batten goto <slug> --to <producer>` also leaves the run paused. The orch role (rule 3) says "Both leave the batten run paused, so resume it with the hand pass". If the child's history already holds a Finalize Done, as in a re-run of a landed or merged run, `childReachedParent` (`cancel.go:728-731`) reads true, so dobby resumes the goto'd run at the next probe tick. That undoes the operator's hold, and no pause entry covers a goto. The added doc line names only `batten pause`, not goto. Evidence: `cancel.go:715-735`, `passes.go:138`, `orch-role.md` diff at rule 3.

No other defects found in the changed code.

**Tool calls:** none. The manifest context covered everything.
