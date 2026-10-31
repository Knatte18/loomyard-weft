```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function the diff adds or changes through one concrete input, from the context I already held plus three reads.

**Per function or branch: case run, result**

- `landNotices` closure, run listed ahead of Land, board entry gone: `onBoard` is false, so `memory.Requested` keeps its last value. Correct. A run first seen ahead and then pruned at Land still sends under `per-task`.
- `landNotices` closure, retried by `UpdateState`: `due = nil` resets on entry and the map is rebuilt from `s`. Correct.
- `landNotices` closure, run past Land and not remembered: skipped, no memory created. Correct. This is the "first seen past Land" case.
- `landNoticeRequests` with only past-Land runs listed: returns `(nil, false)` without reading the board. Correct.
- `landNoticeRequests` with `Tasks()` failing: `tasksRead` is false and the old `Requested` is kept. Correct.
- `DobbyTasks` feeds `landNoticeRequests`: it returns every board entry, run-held ones included, so `Requested` does refresh for a run. Correct.
- `settleLandNotice`, post fails on attempt 1 or 2: `Attempts` is persisted and the run stays listed, so it is due again next tick. Correct.
- `settleLandNotice`, third post fails: the memory is dropped after three attempts in all. Correct.
- `settleLandNotice`, post succeeds with `requested` true: the field is cleared, and a clear error is only logged. Correct.
- `settleLandNotice`, mode `off`: the memory is dropped and the field is not cleared. Matches D5.
- `LoadConfig` failing: `due` is not settled and the memory stays for the next tick. It clears once the config loads or the run is unlisted. Correct, no leak.
- `settleLandNotice`, post succeeds but the drop's `UpdateState` fails: the line is re-sent next tick. This is a duplicate notice, and there is a finding for it below.
- `landNoticeLine`, three cases: the commit slice is bounded by `min`, and the empty-commit and unlanded branches read right. Correct.
- `tick`: `landNotices` runs after `drivePasses`, `runNotices` and `startReady`'s predecessors, inside the `moving == ""` branch. `listedRuns` is cloned before the done filter. Correct against D5.
- `recordLanding`: fresh and merge-in-retry paths both record after `markTaskDone` and before the push. It records nothing on a merge failure, and a failed head read or record only logs. Correct.
- `ClearNotifyOnLand`: returns `noWrite` when the entry is gone or lacks the field. Correct.
- `normalizeTask`: a note with the field is refused with a way forward. `scopeChanged` ignores it. `replayFields` carries it. Correct.
- `PastLand`/`LandOutcomeOf` and hibernate's `triggerReading`: the rule is unchanged and `status.Producer` is the same field. Correct.
- `DobbyLines`, `BuildView`: the header and `notify_on_land` suffixes are right. A nil `LandNotices` map reads as zero. Correct.
- `ReadLandedCommit`/`WriteLandedCommit`: a missing file gives `""`, and the trailing newline is trimmed. Correct.

**Findings**

1. MEDIUM, `design`, `internal/battencli/dobbyreads.go:172-180` (the landed-commit read inside `DobbyRuns`).
   - An unreadable landed-commit file, for example a directory, bad permissions or an I/O error, returns `nil, fmt.Errorf("read the run %s: %w", ...)`.
   - That aborts the whole run listing, so `drivePasses` fails with "list the runs" and `tick` returns the error.
   - A done run stays listed and `ReadLandedCommit` runs on it every tick, so one bad file stalls every run's passes for good.
   - The focus line and D6 say a missing or unreadable file yields `landed, commit unknown`, and D6 says a failed record never changes a verdict. Only the missing-file case is handled that way.
   - Evidence: `dobbyreads.go:166-179` for the read closure and the error loop, `shedrun/landed.go` for `ReadLandedCommit` returning a non-`ErrNotExist` error.

2. LOW, `design`, `internal/dobbyengine/landnotice.go:112-146` and `settleLandNotice`.
   - After a successful `Notice` post, the memory drop goes through `updateLandNotice`, which only logs a failed `UpdateState`.
   - The memory then survives and the same line is posted again next tick, up to a repeated notice per tick while the state file fails to write.
   - The clear of `notify_on_land` can repeat the same way.
   - A fix would be to record "sent" before the post, or to count a failed drop against the attempts.

3. LOW, `design`, `internal/dobbyengine/landnotice.go:57-62`, the cleanup `for slug := range s.LandNotices { if !listed[slug] { delete ... } }`.
   - `DobbyRuns` skips a run whose status is unreadable (a `Warn` and `continue`). The memory of a transiently unlisted run is therefore deleted.
   - When the run reads again past Land, it counts as first seen past Land and gets no line.
   - This follows D4's "no longer listed" rule literally, but it loses a notice on a read hiccup. Whether that is acceptable is the handler's call.

Not checked: I did not confirm that nothing in Worktree-Teardown removes the run's `ScratchDir` before the dobby reads the landed-commit file. The `ScratchDir(` grep returned no run-scratch `RemoveAll`.

**Tool calls**
1. Read `internal/battencli/dobbyreads.go` lines 172-300 for the `DobbyRuns` read loop and the `DobbyTasks` filter; my context cut off there.
2. Read `internal/landingshed/finalize.go` lines 255-335 and grepped `markTaskDone`, `pushParent` to see the merge and retry paths and the PR-merge ordering.
3. Grepped `RemoveAll`/`ScratchDir(` over `internal` to see whether teardown deletes the run scratch holding the landed-commit file. It showed none.
