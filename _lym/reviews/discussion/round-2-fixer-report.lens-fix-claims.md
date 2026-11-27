```yaml
lens: fix-claims
outcome: reported
```

I checked every claim the fix added about bounds, behaviour or code against HEAD. One LOW finding; nothing BLOCKING or MEDIUM.

**Claims checked:**
- **D12, holds:** "`runproc.End` sends SIGKILL 5 seconds after SIGTERM, well inside the command's 30-second bound." `internal/runproc/end.go:21` sets `endGrace = 5 * time.Second`, and `internal/desktophook/window.go:35` sets `commandTimeout: 30 * time.Second`.
- **D12, holds as a new requirement:** "ends one already running before it exits." `runWindowCommand` (`internal/desktophook/window.go`) runs the command through `exec.CommandContext` on a context of its own, and `AfterWindow` takes no context. So the child needs a new cancel path to end a running command. That is planned work, not a false statement about existing code.
- **D8 rationale and D10, consistent:** the statements about a parent under mode `off`, an overflowed parent and a homeless parent agree with D1, D8 and D9. "releases the lock and returns to the parent wait" states a design, not a claim about code.
- **Constraints, not checkable from code:** the `kscreen-doctor -o` and `kwinrc` lines come from the parent's reading in #2819. They match D5's logical sizes.

**Finding:**
- **LOW. D12, "`runproc.End` sends SIGKILL 5 seconds after SIGTERM".** The reason holds only for a child registered in `internal/runproc`. D10 registers it only "when the opening verb runs inside a run", so a SIGTERM from anywhere else comes with no 5-second SIGKILL behind it. The decision itself, running no `window_command` on any SIGTERM, still stands; only its stated reason covers fewer cases than the rule.
  - Evidence: D10's sentence "registered in `internal/runproc` when the opening verb runs inside a run", and `End(dir, owner, …)` in `end.go:60`, which reaches only processes registered under an owner.

**Tool calls:**
- Read `internal/desktophook/window.go` lines 1–33 and 57–120, to see how `window_command` is run and whether a caller can cancel it. My context held only the `productionTiming` lines.
