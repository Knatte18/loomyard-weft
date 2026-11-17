```yaml
lens: fix-code-correctness
outcome: reported
```

Lens `fix-code-correctness`: I ran each changed function by hand on concrete inputs. The functions are `watchForOOMKill`, `killedAtOwnLimit`, the `runMemoryCapped` goroutine and its pipe handling, `enclosingTestBinaries`, and the `killTree` and `onKill` change. I also checked the new and changed tests against the old behaviour. I ran no code.

**Finding 1: MEDIUM, `internal/proc/memorycap_linux.go`, `watchForOOMKill`, `events.ReadAt(buffer, 0)`**
- **Summary:** `ReadAt` can lose the wake-up the poll relies on, so a kill can go unseen.
- **Evidence:**
  - `(*os.File).ReadAt` loops until the buffer is full or an error comes back. `memory.events` is far under 4096 bytes, so the first `pread` returns the content and a second `pread` at a non-zero offset returns EOF.
  - The poll's re-arm point comes from kernfs's per-open event counter. As I recall, the kernel samples it in `kernfs_seq_start`, which every `seq_read` calls, including that second read.
  - The `oom` and `oom_kill` increments are two separate notifies, a few microseconds apart. If one lands between the two preads, the first read has content without the increment, but the second read re-arms the counter after it. The next `unix.Poll` then does not fire.
  - The kill stays unseen until some later change to `memory.events`. The final `scopeHadOOMKill` after `wait` can't recover it, because the scope has been removed by then.
  - The result is `treeKilled == false` and no cap line for a real cap kill.
- **Change that avoids it:** read with one `unix.Pread(int(events.Fd()), buffer, 0)`. A single syscall leaves one re-arm point, taken at or before its content.
- **Not run:** I did not confirm the kernel behaviour. The window is narrow.

**Per-item checks with no findings:**
- **`killedAtOwnLimit`:** I traced "max 3\noom 1\noom_kill 1\n" to true. An empty line, a line with no space and a non-numeric value all `continue`. The `oom 0, oom_kill 1` row returns false, so the row fails against the old `oom_kill`-only code.
- **`runMemoryCapped` goroutine:**
  - **Deadline set after `wait`:** a start pipe left open by a stray writer is bounded by the `SetReadDeadline` call after `wait()`.
  - **`cgroupFound`:** it is buffered, so the send never blocks.
  - **Pipe closes:** the `watchEnded` and `endWatch` deferred closes and the explicit `endWatch.Close()` are harmless double closes.
  - **Error returns:** an error from `start` and the cancel path both return cleanly.
  - **Data-limited path:** it returns before any pipe is made.
  - **Shutdown:** `watchForOOMKill` re-reads once after `endWatch` closes, then sees `Revents != 0` and returns false. A removed scope returns false through the `ENODEV` branch with `count == 0`.
- **`enclosingTestBinaries`:** the loop and its `visited` cycle guard hold. For the nested tree it returns `[testBinaryPID]`. A parent missing from the sample ends the walk.
- **`killTree`:** `onKill` still runs before any SIGKILL, and `RunAlone`'s `GuardKilled` signature is unchanged.
- **`TestStartMemoryGuard`:** the nested row fails without the fix, because `guardKilled(testBinaryPID)` would read false. The no-apply row still asserts false for every pid.
- **`TestKilledAtOwnLimit`:** it is table-driven, parallel and tier 0.

Tool calls: none. My context already held every changed file.
