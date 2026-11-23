```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes through one concrete input.

**Per function and branch: the case I ran, and whether it is correct**

- **`paneMemoryMax`**
  - 0 or -5: becomes 8192 MiB, which is 8589934592 bytes. Correct.
  - A huge value such as 2^44: `int64(mib)<<20` wraps around. busctl then gets a value it can't use for type `t`, and the pane starts with a confusing warning. NIT, not filed.
- **`capPaneMemoryLocked`**
  - `display-message` fails: Warn, then the launch goes on. Correct.
  - Output `""`: `Atoi` fails, Warn, return. Correct.
  - The cap call fails: Warn naming the cause (see F3 for the cause text). Correct apart from F3.
- **`capProcessMemoryPlatform`, `busctl` not on PATH:** returns `ErrMemoryCap` naming the cause. Correct.
- **`capProcessMemoryPlatform`, busctl fails (pid gone):** returns an error with busctl's output. Correct.
- **`capProcessMemoryPlatform`, busctl succeeds but the user manager's job fails afterwards** (for example, it cannot move the pid): `waitForScope` polls for the full 5 s and returns "still in X". That is a wrong or misnamed cause, and the launch stalls 5 s while holding the op lock. This is F3.
- **`capProcessMemoryPlatform`, scope without the memory controller:** `memory.max` is missing, so the error says the controller isn't handed down. Correct. The pane is left in an uncapped `lym-pane` scope, which matches the doc.
- **`capProcessMemoryPlatform`, page rounding:** `limit` is always a whole number of MiB, so `limit&^(pagesize-1)` equals `limit`. Correct.
- **`startScopeArgs`:** the property count is 5 and it matches the five pairs given. I ran it live on this host: `memory.max` read 268435456 and `memory.swap.max` read 0. Correct.
- **`waitForScope`:** an ended pid gives `!ok` and an error. A pid that joins gives the cgroup back. A pid that never joins times out at 5 s. Correct.
- **`processOOMKillsPlatform`:** a gone pid or a missing `memory.events` both return an error. Correct.
- **`oomKillCount`:** `"oom_kill 2"` gives 2, no line gives 0, and a non-numeric value gives 0. Correct.
- **`oomKillsInCgroupPlatform`:** on no match, journalctl exits 1 with empty output (checked live, `jrc=1`), which gives `nil, nil`. A timeout or any other exit returns an error. Correct.
- **`parseOOMKills`:**
  - A name that shares the cgroup's prefix is filtered by exact match. Correct.
  - A task name containing a comma is still captured, because the match is greedy. Correct.
- **`planPaneOOMNotices`, two strands in one cgroup:** if A and B both read cgroup C with 1 kill and nothing seen, the notice goes to A and B is silent. That misattributes the kill. This is F2.
- **`planPaneOOMNotices`, pane whose read fails transiently:** its cgroup is dropped from `seen`, and its earlier kills are told again on the next cycle. LOW, folded into nothing; not filed.
- **`paneOOMNoticeLine`, journal lags or is rate-limited:** with kill k1 already told and kill k2 new, the journal may so far hold only k1. Then `len(killed)=1 >= 1`, and the line names k1's process and pid as the new kill. This is F1.
- **`paneOOMNoticeLine`, other cases:** journal error, or fewer entries than new kills, both give the count-only form. Correct.
- **`reportPaneOOMKills`:**
  - `Status` takes a blocking `flock` and spawns tmux, inline in the discovery loop. This is F4.
  - A `NotifyAsker` failure is logged, but `seen` was already advanced, so the notice is lost for good. This is F5.
- **`watchedSession` built with `oomKills` on both `enterSession` returns:** the map is a reference, so updates made through the copy taken in `range known` persist. Correct.
- **`smoke_helpers_test` `addStrand`:** delegates to `addProviderStrand`, and the empty provider matches the old CLI path. Correct.

**Findings**

F1 — MEDIUM, design. `internal/reedcli/oomnotice.go:84-94` (`paneOOMNoticeLine`).
- **Summary:** the notice can name the wrong killed process.
- **How:** it names the last `newKills` journal entries for the cgroup whenever the journal holds at least that many. The journal is cumulative and lags the counter: journald reads `/dev/kmsg` asynchronously, and the kernel's `oom-kill:` summary line is rate-limited. So a second kill read before its journal line lands, or after a dropped line, is reported under an earlier kill's name and pid.
- **Evidence:** `oomnotice.go:88-93` compares `len(killed)` against `notice.newKills` and slices the tail. Nothing records how many journal entries earlier notices already named.
- **Fix:** keep per cgroup the number of journal entries already told, and name only entries beyond it. Otherwise fall back to the count.

F2 — MEDIUM, design. `internal/reedcli/oomnotice.go:55-78` (`planPaneOOMNotices`).
- **Summary:** kills are keyed by cgroup and attributed to the first strand that reads it.
- **How:** an uncapped pane (the degraded path, or a tmux without systemd support) shares its cgroup with other panes and the tmux server, or with an older session or app scope. Then:
  - one kill is told as the first strand's and never as the right strand's;
  - a first-seen shared cgroup counts from 0, so every earlier OOM kill in that long-lived scope is told as "at the memory cap of strand X", although no cap applied.
- **Evidence:** `oomnotice.go:62-70` keys `seen` by cgroup with no check that the cgroup is a `lym-pane-` scope. `panememory.go:121-140` leaves such panes in whatever cgroup tmux gave them.
- **Fix:** report only for cgroups whose base name starts with `paneScopeUnitPrefix`, and so only for panes that were capped.

F3 — MEDIUM, design. `internal/proc/processscope_linux.go:151-157` (`capProcessMemoryPlatform` / `waitForScope`).
- **Summary:** busctl's `StartTransientUnit` returns once the job is queued, so a job that fails later is never seen as failed.
- **How:** an example is the user manager refusing to move a pid that sits outside its tree, such as a pane in `session-N.scope`. The code waits the full 5 s `scopeJoinTimeout`, then reports "process N was still in <cgroup> when the wait for the scope … ended". The Goal asks for a warning naming the cause; this names a symptom instead. Every launch on such a host also stalls 5 s while `launchStrandLocked` holds the reed op lock.
- **Evidence:** the lines above (`processscope_linux.go:151-157`) and `waitForScope` (`processscope_linux.go:191-207`) never read the job result. `spawn.go:101` calls this under the op lock.
- **Fix:** read the job's outcome. Options: subscribe to `JobRemoved`, check `systemctl --user show -p ActiveState,Result <unit>` once the unit leaves activating, or poll the unit's state alongside the cgroup and return its failure result at once.

F4 — MEDIUM, design. `internal/reedcli/watchdog.go:624-626` and `internal/reedcli/oomnotice.go:101-107` (`reportPaneOOMKills`).
- **Summary:** the per-session OOM step now runs inline in the discovery loop and can stall it.
- **How:** `ws.eng.Status()` takes the worktree's reed op lock with a blocking `flock` (`lock.go:157` → `lock.go:36`). It then spawns tmux (`requireSessionLocked`, the pane listing) and can adopt or repair state (`spawn.go:373-393`). `OOMKillsInCgroup` can block for up to 10 s, and `NotifyAsker` posts through creel, all inline. A long `AddStrand` or `Resume` holding any session's op lock therefore stalls discovery, reaping and the replaced-binary check, and `ctx.Done()` goes unread meanwhile. The loop's own doc says reaps run off-loop for exactly this reason.
- **Evidence:** `watchdog.go:624-626` calls `reportPaneOOMKills` inline, and `lifecycle.go:1299` shows `withOpLock`.
- **Fix:** run the OOM check off-loop, as the per-session name-repair goroutines are, or take the lock non-blockingly (`TryAcquireWriteLock`) and skip the cycle when it is held.

F5 — LOW, design. `internal/reedcli/oomnotice.go:69-70,112-114`.
- **Summary:** a failed post loses the notice for good.
- **How:** `seen[cgroup]` is advanced in `planPaneOOMNotices` before the post. When `orchcli.NotifyAsker` then fails (creel down, a refusal), the kill is logged and never told to the parent. The Goal says the kill must reach the run's parent.
- **Evidence:** `oomnotice.go:70` updates `seen` before `oomnotice.go:112` posts, and a failure there only logs a Warn.
- **Fix:** advance `seen` only once the post succeeds, or roll back to the old count on a post error.

Tool calls:
1. Read `loadOrInitStateLocked` and `requireSessionLocked` in `internal/reedengine`, to learn whether the watchdog's new inline `Status()` call spawns processes or writes state. My context showed only `Status` itself, not these two callees.
