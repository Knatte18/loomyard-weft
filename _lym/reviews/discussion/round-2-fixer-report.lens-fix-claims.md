```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking that each claim the fix added or rewrote holds against the code at HEAD.

**1. MEDIUM — D4, the takeover sentence: "`lym tender pause <slug>` stops the waiting run, which leaves it paused at PR-Review, where goto admits the row, then `lym loom goto` sends it back to PR-Review."**
`lym tender pause` does not put a waiting PR-Review run into the paused state, so the takeover the fix names cannot be carried out as written.
- `lym tender pause` records a pause and holds agent and network work. It does not cancel the driver (`internal/tendercli/cli.go` pauseCmd Long).
- A covered PR-Review row stays inside `Call`, in `holdWhileCovered`, sleeping while the pause covers the run (`internal/landingshed/prreview.go:493-523`). The run never returns a step, so its status is not set paused.
- Only a cancelled context sets paused (`internal/shedengine/run.go:389`).
- `Goto` takes the run lock first and refuses with `ErrShedBusy` while the driver holds it (`internal/shedengine/goto.go:80-86`).
- So the goto is refused busy. The paused, blocked or failed state that `gotoAdmitted` admits is never reached this way.

**2. MEDIUM — D4, the answer bullet: "a PR-Review entered again after a send-back sends a new ask 1". The surrounding text keeps "The row sends ask 1 when it opens the review" and puts each ask's number on its `[pr-review request]` line.**
PR-Review asks now live in burlerengine's run-wide record, so the row's first ask is not numbered 1 once the run has made any earlier ask.
- `RecordParentAsk` numbers an ask one past the record's latest, across every tag and seed (`internal/burlerengine/parentreview.go:200-213`).
- After a Code-Review, Discussion-Review or judge ask, the row's first PR-Review ask gets the run's next number.
- "ask 1" is only right as the first ask within the PR-Review seed, which the text does not say. A reader takes it as the `--ask <n>` value.

**3. LOW — D4, the closed-pull-request bullet: "Finalize never reopens the pull request (`internal/landingshed/finalize_pr.go`)", repeated in the acceptance bullet "Finalize never reopens it."**
"Never" overstates the code.
- Finalize reopens a closed recorded pull request whenever the parent's decision is merge-at-once, whatever `ReviewEnded` says (`internal/landingshed/finalize_pr.go:358-364`).
- What the fix secures is narrower: with `ReviewEnded` unset and no merge-at-once, Finalize halts.

**Claims checked and found true:**
- `WithPresence` returns the pointer unchanged outside a slack presence.
- Halts fold: a halt's entry is a budget-exempt Stuck (`internal/shedengine/run.go:368`), and `appendOrFold` folds such entries.
- `RecordResumer` writes no history entry.
- The judge ask ends no-parent on a failed post (`postJudgeAsk`).
- The not-sent and no-parent states pass the parent-agreement gate.
- `settleSendBack` clears the session fields.
- `LatestTagged` exists.
- The `Digest` field is documented as naming content that can change under an ask.
- The sibling parent timeouts are `CategoryNone` and validated at least 1.
- The answer verb answers only the latest ask before its deadline.

Further tool calls:
- Grepped `BudgetExempt` and read `finalize_pr.go:340-370`: needed to check whether a halt folds and the reopen paths; my context lacked both.
- Located the tender pause code: needed to check the takeover claim; my context had no tender code.
- Read the `tender pause` help and `run.go:380-395`: needed to check whether tender pause cancels the driver into the paused state; my context lacked both.
- Read `goto.go:72-110`: needed `Goto`'s lock and state checks; my context held only `gotoAdmitted`.
