```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: tracing each guard, branch and test row the plan adds, to check that a real entry point can reach it.

**MEDIUM — Card 10 step 5 / Card 7 `Runner.Query` sharing: the query-sharing shortcut can hand a stale snapshot to the child that holds the lock**
- Card 7's `Query` returns any `kwinquery.json` report for the same output that ended after `after`. That holds for every caller, not only a window-wait child.
- Card 10 step 5 runs "a fresh `Query`" under `kwinplace.lock`, but never says what `after` it passes.
- This sequence reaches the shortcut under the lock:
  1. Child A queries under the lock.
  2. Child C's window-wait query runs. It holds only the script lock, so it can run between A's query and A's apply.
  3. A applies and releases the lock.
  4. B takes the lock. Its "fresh" query passes B's last query end as `after`, and C's report ended after that.
  5. B plans from a snapshot taken before A's apply and can fill the same zone. That is the hazard D10's lock exists to prevent.
- D10 limits the sharing to a window-wait child: "A window-wait child that takes the script lock and finds there a report…".
- Fix: the query under the lock passes the lock-acquisition time as `after`, or skips the shared report. Add a `TestFlow_Place` or `TestRunner_ScriptRuns` row with a report that ended before the lock was taken.

**MEDIUM — Card 8 `ReadPlaces` / `RecordOpen` / `Settle` / `RecordReport`: a corrupt places file is tolerated only on read, and every write fails on it**
- Card 8 says an undecodable file "reads as empty, … logged at Warn", which is D9's "An absent or unreadable file reads as empty".
- Every write goes through `state.UpdateJSON`. Per `internal/state/state.go`, "A decode failure on an existing file aborts before mutate runs, and no write happens".
- So once the file is corrupt, `RecordOpen`, `Settle` and `RecordReport` fail every time, and no entry ever settles.
- Every child whose parent's entry lives in that file then waits out the whole 180-second parent wait and lands on the current desktop, homeless, for good.
- `TestPlacesFile` has a row for an absent file only, none for an undecodable one.
- Fix: each write's mutate treats a decode failure as empty, or the file is moved aside before the update. Add the row.

**MEDIUM — Card 10 step 4 "Unless the window is the prime's" / Card 11 parent resolution: `parent == ""` means both "the prime's own window" and "parent is the prime"**
- `Request` carries "the parent worktree (`""` when the window is the prime's)".
- Card 11 resolves the parent through `hubgeom.ParentOf`, "whose `""` names the prime". `internal/hubgeom/parentof.go:34-35` returns `""` for a beam, and for a task under the prime.
- Card 11 never says it maps that `""` to the prime's worktree name. As written, every beam and every task under the prime takes the prime-window branch: no parent wait, anchor on the current desktop.
- D8 says the anchor is "the prime for a beam or a task under the prime".
- No row reaches the case: `TestFlow_Place` has no beam-under-prime row, and Card 11's `tier1` test uses a task worktree.
- Fix: Card 11 maps `ParentOf`'s `""` to the prime's worktree name from `fabricengine.PrimeName`, and passes a separate prime flag. Add a flow row in which a beam's child waits for the prime's entry.

**LOW — Card 12 `windowCmd` hand-off failure: the window command is dropped on a path no card guards**
- When the pre-launch checks pass and `handPlacement` (`kwinplace.Spawn`) fails, the verb only warns and runs no `placeWindow`. No child exists, so `desktop.window_command` never runs for that window.
- D12 runs the command either from the child, once it is handed the window, or from the verb when the checks fail. A failed hand-off is neither, and the card leaves it unhandled.
- The `HandOffFailureOnlyWarns` row checks the envelope only, not whether the window command runs.
- Card 13 has the same gap. It is consistent with "machine start runs none" today, but under placement the command was owed.
- Fix: run `placeWindow` (with its wait) when the hand-off fails, and assert it in that row.

**LOW — Card 10 row "a waiting child makes no lock attempt until the lock frees": the assertion cannot fail**
- Card 7's `acquireWithin` wraps the blocking `lock.AcquireWriteLock` in a goroutine. A blocked flock makes no attempts the test can count, so the row passes whatever the code does.
- A failing row would assert that the flow's sleeper records no wake while the lock is held and that the lock is taken once it is released, or would drop the row as covered by the acquire's design.

Further tool calls:
- One grep, of `internal/fabricengine/worktreelist.go` and `internal/proc/procgroup.go`. My context lacked two facts: whether `fabricengine.List` runs git, which bears on Card 11's `PATH`-stripped test reaching its failure branch (it reads through `dotgit`, so the test likely reaches it), and whether `proc.ConfigureGroupTerminate` exists with the signature Card 3 relies on (it does).
