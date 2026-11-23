# Run every strand pane in a memory-capped scope and report OOM kills

A runaway process in an agent pane could take the whole machine down: two detached `script -qfc "tmux attach"` processes reading `/dev/zero` grew to 6 GB in 3 seconds and froze the host.
Only test binaries ran under a memory cap before.

Now `reedengine` moves each fresh strand pane's shell into a transient systemd user scope, `lym-pane-<pid>.scope`, before it types anything into the pane.
The scope sets `MemoryMax` from the new reed config key `pane_memory_max_mib` and allows no swap.
The kernel then kills a runaway process inside that scope and the pane and its other processes live on.
Every hub and engine goes through the same spawn chokepoint, so none needs its own wiring.
The change needs no root: it talks to the user manager over `busctl`.

A pane whose scope cannot get the cap launches uncapped with a Warn that names the cause.
Causes include no `busctl` on PATH, a user manager that refuses the scope, and one that does not hand the memory controller to its scopes.
Windows always takes this path.

The reed watchdog daemon also reads the OOM kill count of each capped strand pane's cgroup.
For each new kill it posts a notice to the worktree's parent orch that names the strand, the pane and the killed process.
The process name comes from the kernel journal, and the notice falls back to a count when the journal does not name it.

## Design decisions

- `pane_memory_max_mib` defaults to 8192 MiB, taken when the value is zero or negative.
  The cap has to fit an agent session plus the builds and tests it runs.
  The value is a plain config key with no category, so a hub can change it without a rebuild.
- `proc.CapProcessMemory` and `proc.ProcessOOMKills` are the new platform-split primitives in `internal/proc`.
  The scope's OOM policy is `continue`, so a kill at the cap ends one process the kernel picks, never the whole scope.
  A scope is collected once it empties.
- The OOM watch starts in `enterSession`, in its own goroutine, because reading the strands waits on the worktree's operation lock.
  `watchdog: off` does not stop it, since it only reads.
- A kill counts as told only after its post succeeds, so a failed post is retried at the next read.
  A cgroup that is not a `lym-pane-` scope is skipped: it may hold other panes or the tmux server, and its kills are not this strand's.

## Check by hand

- Run a pane that allocates past the cap on Ubuntu 24.04 and on Ubuntu 26.04 with kernel 7.0.
  Confirm the process dies, the pane survives, and the orch receives the notice.
  No test here runs a real kernel OOM kill end to end.
- Start a pane on a host without the memory controller delegated to the user manager.
  Confirm it launches with the Warn.
- The degraded-launch Warn has no test of its own.
- The notice can name an earlier kill, or the wrong process, when several panes share a cgroup or when the journal line lands after the kill is counted.
  `paneOOMNoticeLine` handles the late-line case, and a shared cgroup is only partly covered.
- The delivery of the notice to the parent orch is untested end to end.
- The OOM step reads the strands, which waits on the operation lock.
  Check that it does not stall the watchdog loop.

## Merge danger

**Door:** two-way
**Blast radius:** reed
