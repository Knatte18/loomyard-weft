---
verdict: APPROVED
---

No findings.
Range: `git diff $(git merge-base main HEAD)..HEAD`, `product.parent` being `main`.

## Rejected

- Window listing taken before the new window opens: `newAgentWindowTakesViewerLocked` runs ahead of `newWindowLocked` in `openOwnWindowLocked` (`internal/reedengine/spawn.go`), so `relevantWindow` and `driverWindows` see the windows and panes as they stood before the spawn, and no new tmux call follows the spawn.
- Lock discipline and failure path: the helper keeps its `Locked` contract, adds one `listSessionWindows` read, and on a failed listing returns false and logs at Warn, as the summary claims.
  The listing duplicates one `openOwnWindowLocked` already takes, a cost the diff does not need to remove.
- Moving only a viewer on the previous newest agent window or the Driver window: `newAgentWindowSelected` adds exactly the `relevantWindow` and `driverWindows` cases, so a viewer on an older live agent window or the shell stays, and `TestNewAgentWindowSelected` pins both, including the driver pane in the shell window.
- Driver match: `driverWindows` goes through `agentname.Parse` and `agentname.MatchesRole` with `agentname.RoleDriver` (`PATTERN-agent-name`) and counts only own-window, live panes.
  A strand joining a shared group window opens no window, so no switch happens, which the decision record scopes out.
- Replace subtest expectation: ReplaceStrand removes the old strand, which falls back to the newest live agent, and the replacement then takes the viewer from there, as the summary and the decision record's open risks say.
- `PATTERN-test-economy`: the `TestWindowOrder_RealTmux` subtests extend the existing focus subtests and pin real tmux selection, and the unit table pins the rule's cases without a second copy at the same tier.
  The budget row rises by one for the single new top-level test, as the budget rule requires.
- Docs: the focus-rule paragraph in `internal/reedengine/doc.go` states the widened rule, and no other doc or `PATTERN.md` text describes the old rule; the `spawn.go` comment still holds.
- `PATTERN-wait-mark-display-only`: the rule reads reed state, window ids, strand roles and pane liveness only.
