```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked each sentence the fix added or rewrote that asserts a bound, a behaviour or a fact about the code against HEAD.

**Findings**

1. **MEDIUM**: Decision 1's new Bound: "A beam dobby spawned inside a teardown of its own beam escapes that teardown and exits at its first probe tick, which finds no worktree or another owner."
   - The loop has no worktree-existence exit, so the claim, taken over from the reviewer's proposal, states a behaviour the code does not hold.
   - The only probe-tick exit is `beamOwnedElsewhere` returning true (`internal/dobbyengine/loop.go:212-217`), which needs a non-empty owner that differs from this machine.
   - A missing worktree only makes `d.tick` fail, which `d.failures.Fail("...a tick failed; the loop goes on"...)` logs once per episode before the loop continues (`loop.go:220-224`).
   - Nothing in `internal/dobbyengine` stats `WorktreeRoot` apart from `start.go:273`, which only passes it as an argument.
   - So after a same-machine teardown, an escaped dobby keeps running, failing a tick on every wake.
   - The bound must either name that outcome, or the record must add an exit for a missing worktree.

2. **LOW**: Decision 2's re-exec test: "holds this process's pid and this process's own start ticks", with "the state record gains that start-ticks field, written by `recordStart`". Decision 3's wiring-failure notice uses the same test.
   - A state record written by a binary from before this change carries no start-ticks field, so the tolerant decode reads it as empty.
   - The first re-exec onto the binary that ships this change therefore reads as a fresh start.
   - It waits only `lock.ProbeGrace`, which is the incident's own path, and a wiring failure on that exec posts no failed-re-exec notice.
   - The fix states no bound for this one-time transition. It needs either a bound, or a fallback to pid alone when the field is empty.

Checked and holding:
- `hubreconcile.EnsureOrWarn` is what the batten and loom arms call (`battencli/arm.go:345`, `loomcli/arm.go:153`).
- Only `settleTidy` clears `LandedSinceTidy` (`tidy.go:203`), and neither the owned-elsewhere exit nor `recordStart` touches it.
- The `failures` and `ownerRead` episodes log through `logger` (`loop.go:222-224`, `250-254`), whose stderr half goes to `os.Stderr` outside tests (`logger.go:161-167`).
- `runproc` records are keyed on pid plus `proc.StartTime` (`record.go:99`), so the claim that a re-exec keeps the old image's record holds.
- A hub may hold more than one slot (`gate.yaml` `slots: 3`).

Tool calls:
- Read the episode use in `loop.go`, `settleTidy` and `logger/episode.go`, to verify the claims added to Decisions 6 and 8.
- Read `beamOwnedElsewhere` and the logger's stderr wiring, to verify the owner-read and stderr claims.
- Read the loop body and searched `dobbyengine` for a worktree-existence check, to verify Decision 1's teardown-escape bound.
