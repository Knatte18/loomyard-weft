```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2, F4, F5, F7, F8, F9, F10, F11, F12, and F6's state check), and report only cases where a proposal is wrong or breaks something.

Findings:

1. **MEDIUM, F6, the `waitForScope` state check (`internal/proc/processscope_linux.go`, `scopeEnded` and `scopeStateEnded`).** A healthy scope can be failed by mistake.
   - `scopeEnded` treats `inactive` as failure from the 10th poll on, about 200 ms in. `scopeStateEnded` accepts `failed` and `inactive`.
   - `StartTransientUnit` returns when the job is queued, so a unit whose start job is still waiting reads `inactive`, not `activating`.
   - The fixer's own probe shows a nonexistent unit reads `inactive` too, so the state cannot tell "job queued" from "job failed and collected".
   - On a loaded user manager, a scope that would have joined within the 5 s budget now returns `ErrMemoryCap` at ~200 ms, and the pane runs uncapped.
   - The old behaviour here was right, since it waited up to `scopeJoinTimeout`. The new check trades that for an earlier error on a failure that is rarer than a slow start.
   - Nothing tests the `waitForScope` branch itself. The new tests cover only the label escape and the string parse.

2. **LOW, F7, the retry-until-post-succeeds change (`internal/reedcli/oomnotice.go`, `reportPaneOOMKills`).** It can duplicate or loop.
   - `NotifyAsker` documents "the notice is not retried" and returns an error for a refusal or a failure. A refusal is permanent, so the same notice is now re-posted every cycle, journal read included, with a Warn each time, until the pane's cgroup goes away.
   - If creel accepts the mail but reports an error, the retry posts the notice a second time.
   - The cadence cap (60 s) bounds the rate but not the repeats.

3. **LOW, F3 (fixed differently), the told count used as a journal offset (`internal/reedcli/oomnotice.go`, `paneOOMNoticeLine`).** It assumes the Nth `memory.events` kill is the Nth journal entry for the cgroup, and that can break.
   - `ProcessOOMKills` reads the counter from `memory.events`, which can include kills in descendant cgroups. `parseOOMKills` matches `task_memcg` exactly to the pane's cgroup.
   - If a process in the pane makes a child cgroup, such as a nested `systemd-run --scope`, a kill there raises the counter with no matching journal entry. The offset then drifts, and later notices name the wrong entry or fall back to the count form.
   - The old "last N entries" had the same flaw, so this is not a regression. But the stated premise, "told already tracks position", does not hold in that case.

4. **LOW, F10, the `memory.swap.max` read (`internal/proc/processscope_linux_test.go`).** The tier-1 test fails where it should skip.
   - `memory.swap.max` does not exist when swap accounting is off. For example, the kernel boot parameter `swapaccount=0` removes it.
   - The test fails on `err != nil` instead of skipping. The `memory.max` read before it never had that problem.
   - `MemoryCapCoversTree` does not check swap accounting.

5. **LOW, F1 (`internal/reedcli/watchdog.go`, `enterSession`).** The daemon no longer waits for the watch goroutine.
   - The loop's deferred teardown cancels each context and waits on `wg`, but the OOM watch is not in `wg`.
   - A watch inside `Status` or a post when the daemon returns can outlive it. That is harmless but untracked.
   - Cancelling on departure also cannot interrupt a goroutine blocked in `Status`'s `flock`.

Checked and not findings: no tier-0 test is exposed to a real busctl spawn through the new `capPaneMemoryLocked` path. The other `display-message` fakes answer with `|`-separated strings that fail `strconv.Atoi`, so they take the warning path. F2's prefix check, F4's boundaries struct, F5's rows, F8's Info level, F9, F11 and F12 survive challenge.

Tool calls:
- One `grep` over `*_test.go` for `display-message` fakes, because my context did not show whether any other launch test answers `#{pane_pid}` with a plain number, which would now spawn a real busctl.
