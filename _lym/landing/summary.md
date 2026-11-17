# proc: keep a capped scope's kill off the desktop's low-memory notice

`RunMemoryCapped` runs a command in a transient systemd user scope with `MemoryMax` as the cap.
When the kernel killed a process at that cap, the unit ended with the result `oom-kill`.
KDE Plasma's out-of-memory notifier reports every such unit as "terminated by the Linux kernel because the system is low on memory".
The host was not low on memory: the kill was the scope's own memcg limit.
Each gate run of the cap's tests raised several notices, and every capped mutant of `lym gate redundant` would raise one more.

## What changes

- `memoryScopeArgs` passes `-p OOMPolicy=continue`, so a kill at the cap never fails the scope with `oom-kill`, and the notifier stays quiet.
- With that policy the scope survives the kill, so anything the command started would outlive it.
  `runMemoryCapped` now writes `1` to the scope's `cgroup.kill` after any kill at the cap, which ends what is left.
  A scope already gone is no error.
- The start script writes the scope's cgroup listing to file descriptor 3 instead of one blank line.
  `scopeCgroup` takes the cgroup path from it, so the caller knows which `cgroup.kill` to write.
- The scope kill follows two cases: the command itself died of SIGKILL, or `memory.events` shows `oom_kill > 0`.
  The second covers a child of the command killed at the cap while the command went on and exited by itself.
  `killed` still reports only the command's own SIGKILL, as before.

## Design decisions

- Keeping the scope in `app.slice` and changing the policy was chosen over a slice of lym's own.
  The notifier keys on the unit's result, not its slice, so a different slice would not silence it.
- `memory.events` is read rather than relying on the exit status alone, because a surviving command can exit 0 after its child was killed.
- The `RLIMIT_DATA` fallback for hosts whose user manager cannot cap a scope is untouched.

## Tests

- A kill of the command at the cap leaves no process the command started.
- A kill of a child while the command runs on lets the command finish with no error, then ends the scope and the process it left.
- The existing cap test now requires exactly an `*exec.ExitError` from a killed run.

## To check by hand

- The desktop notice itself: run the `proc` tests, or a capped `lym gate redundant`, on a KDE Plasma host and confirm no notice appears.
  No automated test observes the notifier.
- `leftover-kill-test-real-time-bound` (`internal/proc/memorycap_linux_test.go`):
  `requireProcessGone` waits up to a real minute for the left-over `sleep` to vanish, a wall-clock bound in a test.
- `oom-kill-of-child-leaves-scope-running` (`internal/proc/memorycap_linux.go`):
  when a child is killed and the command goes on, the scope runs until the command ends or the scope kill; check that this is the wanted behavior.
- `oom-policy-continue-unpinned` (`internal/proc/memorycap_linux.go`):
  confirm the tests pin `OOMPolicy=continue` strongly enough that dropping it fails one.
- `scope-start-script-repeats-cgroup-path-literal` (`internal/proc/memorycap_linux.go`):
  the cgroup path handling around the start script, `scopeCgroup` and `killScope` repeats path pieces; check whether one definition should serve them.
