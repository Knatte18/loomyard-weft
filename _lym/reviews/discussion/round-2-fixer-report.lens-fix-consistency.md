```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the round-2 change set still agrees with the text around it in `decision-record.md` and `support-log.md`.

**1. MEDIUM — D4 still says "ask 1", which the new ask store does not produce.**
- Changed text: D4's bullets "The row sends ask 1 when it opens the review." and "a PR-Review entered again after a send-back sends a new ask 1".
- Problem: the fix moved PR-Review asks into burlerengine's run-wide ask record. That record numbers every ask one past the run's latest (`internal/burlerengine/parentreview.go` `RecordParentAsk`), so a review's first ask comes after the run's earlier Code-Review, Discussion-Review and judge asks and is never numbered 1.
- The same bullet now relies on run-wide numbering: "with an earlier Code-Review or judge ask in the run … `--ask <n>`". The two statements contradict each other.
- Suggested change: say "the review's first ask" instead of "ask 1".

**2. LOW — the shell-guard constraint does not cover the new await call.**
- Changed text: D4 now has the PR-Review session wait "with `lym loom parent-review await`".
- Problem: the Constraints bullet that clears the shell guard still lists only `lym loom parent-review ask` (new) as open to a PR-Review session. Nothing says `parent-review await` is open to the `pr-reviewer` strand.

**3. LOW — the D4 stencil acceptance bullet misses the await.**
- Changed text: D4's re-ask bullet, which now adds `parent-review await` after each ask.
- Problem: "Review: D4 — the PR-Review stencil, under autonomy, asks the parent after each fix with `lym loom parent-review ask` and runs `lym loom pr-continue` only on its satisfied answer" still leaves out the await the stencil must now direct.

**4. LOW — the support log's takeover trade-off still has one step.**
- Changed text: D4's takeover is now two steps: `lym tender pause <slug>`, then `lym loom goto`.
- Problem: `support-log.md` Rejected alternatives still reads "takes a running PR-Review over by sending the run back to PR-Review". It names no pause first.

**5. LOW — D4 calls the answer verb unchanged, but its help is now wrong.**
- Changed text: D4's "The answer verb is unchanged".
- Problem: the `parent-review` group Long help and the `answer` help name only "a Discussion-Review or Code-Review round's ask … or the drift judge's ask" (`internal/loomcli/parentreview.go:332-335`). Neither the record nor D9's exception list names updating that help for PR-Review asks.

No further tool calls were made.
