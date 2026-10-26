```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F2–F5 as carried out. I made two reads (listed at the end).

Proposals checked: F2, F3, F4, F5.

**[MEDIUM] F5: the new row cannot catch the defect the finding names**
- Changed text: the `runStdin hands a retry the same stdin` subtest in `internal/reedengine/overlay_test.go`.
- Summary: the row goes through `execHook`, which receives the `stdin []byte` directly, so it never reaches the per-attempt reader and command the finding says must be rebuilt.
- Evidence: `overlay.go:125-126` calls `p.execHook(false, stdin, args...)` with the original slice before any reader is built.
  A production defect where a retry reuses an exhausted `bytes.Reader` or a spent `exec.Cmd` would still pass this test, because the hook is handed the same slice every time.
  The finding's premise is "`runStdin` rebuilds its command and reader per attempt".
  Only the real exec path exercises that.
  The row pins that the slice is passed unchanged, not that the replay works.
- What would catch it: an exec-path test, such as a fake tmux executable that records its stdin per invocation and exits EAGAIN-style on the first.
  If that is unreachable, the row's name or comment should say it pins the hook seam only.

**[LOW] F4: a single-wait removal row still passes with the defect in place**
- Changed text: the `a strand removal that succeeds on a retry reports the strand removed` row in `TestStartup_PaneNotLiveMidStartup`, with `wantRemoveWaits: {startupRemoveFirstDelay}`.
- Summary: the suffix check is satisfied by the poll sleep that comes before the teardown.
- Evidence: the status queue is live then dead, so tick 0 sleeps `base` = 1 s (`wait.go:719`) and tick 1 abandons.
  The sleeps are `[1s]` without any removal wait and `[1s,1s]` with one.
  Both end in `[1s]`, so deleting `run.clock.Sleep(delay)` at `wait.go:812` leaves this row green.
  The rows with two or more waits are safe, because their suffix is longer than the one preceding poll sleep.
- What would catch it: compare the whole sleeps slice, or compare the sleeps beyond the poll sleeps recorded before teardown.

**[LOW] F2: the rewrite is silent if the template line drifts**
- Changed text: `strings.Replace(cfg, "start_retries: 1", "start_retries: 0", 1)` in `providerlessShuttleConfig`.
- Summary: if the template line changes, the replace does nothing and the smoke fixtures go back to a double launch with no signal.
- Evidence: the sibling `claudeConfigWithBinary` panics on drift (`smoke_helpers_test.go:112-114`), but this helper does not, and the `startup_timeout_s` rewrite beside it is equally silent.
- Not worth a fix on its own, since the proposal followed the existing pattern.

**F3:** no finding. The doc-comment move is mechanical, and the comment now sits above `type keyTimesReed`.

Reads, 2 of 6 allowed:
1. `sed -n 118,145p internal/reedengine/overlay.go`: my context had only a grep excerpt of `runStdin`, not the exec path that builds the reader.
2. `sed -n 708,722p internal/shuttleengine/wait.go`: I re-checked the `base` sleep for tick 0; my context already showed this, so the call was redundant.
