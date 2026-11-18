```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the parts of F13's fix that the fixer took over from the reviewer's proposal.

F13 was fixed differently, not as proposed, so I only challenged the parts it took from the proposal: the clear moved to the row's next `Call` behind a history test, and the cap-halt Bound.

**[LOW] A goto during a marked halt makes the row send back a second time.**
- Changed text: D4, "A mark met with any other newest entry keeps all four, and the row handles it as its wake does, rerouting on a clean tree." The fix's list of such entries names the re-step, the crash, the dirty-tree resume and the cap-halt resume. It does not name a parent's `lym loom goto`.
- Case:
  1. PR-Review halts Stuck on a dirty tree with the mark for `pr-findings-1.md` set.
  2. The parent commits, then runs `lym loom goto Code-Bouncer` instead of `lym loom resume`.
  3. Code-Bouncer runs a full generation, since the send-back file was written after the settle. Drift-Recheck and Describe follow, and then PR-Review is entered again.
  4. The row's newest history entry is now the dirty-tree Stuck, not a reroute whose output is the mark's file. So it keeps the mark and reroutes a second time.
- Effect: Code-Review reviews the same findings file twice and the run spends one extra reroute. Nothing is lost, so this is waste rather than a wrong outcome.
- Evidence: D4's Bound sentence "A parent's `lym loom goto` back to Code-Review keeps the count" covers only the count. No sentence says what a goto does to a pending mark. Shed records a goto as its own entry (`internal/shedengine/goto.go:339` branches on entry outcomes), and that entry is neither a reroute nor the row's.

No other findings. I built two more cases and neither breaks the fix:
- **Folding.** A reroute entry has `BudgetExempt` unset, so `foldsInto` (`internal/shedengine/run.go:745`) never merges two reroutes. A stale output on the newest reroute entry cannot happen.
- **Output used elsewhere.** No reader of reroute entries uses their `Output` path, so setting it to the mark's file conflicts with nothing.

Tool calls:
- grep for `OutcomeReroute` readers across `internal`: my context did not show whether anything consumes a reroute entry's output path.
