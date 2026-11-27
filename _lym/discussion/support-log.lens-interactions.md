```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state, then checked the criteria and constraints against those decisions.

Pairs checked:
- D1×D2 (board and landing targets under the interval): held. D5 exempts the landing push from the interval wait.
- D1×D10 (an agent's own push next to the pusher): held. The pusher's fast-forward becomes a no-op.
- D2×D4 (interval shown in the bar): held.
- D2×D6 (the shared `fabric.push_ceiling_s`): held.
- D2×D5 (the landing skips the wait, then falls back to backoff): held for the landing target. Broken for Worktree-Teardown, see F4.
- D3×D4 (the ledger feeds the bar's minutes): broken, see F3.
- D3×D6 (one shared threshold, two notice sources): broken, see F2.
- D3×D7 (a refused target in the ledger): broken, see F6.
- D5×D6 (the local landing, Finalize's own records push, the PR wait): broken, see F1 and F5.
- D5×D7 (a fast-forward-only landing push meets a moved origin parent): held. It becomes a refusal with one notice.
- D5×D3 (the board `done` through the board target): held.
- D5 teardown × the dobby's landed-pair teardown × PATTERN-owner-machine reconcile: held. An ended entry whose parent holds the work goes on.
- D6×D7 (an auth failure moved out of the transient classes): held. D7 takes it out of D6's unbounded loop.
- D6×D8: held. The catch-up is not a step.
- D7×PATTERN-run-fsm takeover and webster reset: broken in part, see F7.
- D9×D5: held. They work on independent state.
- D11×D5 (the teardown test with pushes off): held.
- D12×D2 (the code-tip wait counts on the pusher now that pushes are less frequent): held. The 300 s ceiling stays well under `runfsm.CodeTipGrace` (30 min).

Findings:

F1 — BLOCKING — D5 against D6 and D5's criterion.
D5 says a local landing "returns done while origin is unreachable", but Finalize still pushes the run's records synchronously after the merge.
- `pushRecords` (`internal/landingshed/finalize.go:843`) is called at line 990, after `pushParent`.
- The exploration showed it marks an owed push `TransientGitTransport`.
- With origin down, that push turns into a git-transport stop. D6 then re-steps Finalize past its merge without end, and the D5 criterion cannot pass.

The record must also route Finalize's records push through the pair's pusher, or name it as a step that still waits under D6.

F2 — MEDIUM — D3 against D6, which share `fabric.push_notice_after_min`.
- During one outage the dobby posts its one summary. Each run the driver is waiting on also posts its own "waited past threshold" notice, and later its own "moving again" notice.
- With N runs at PR-Review or Finalize, the orch gets 1 + 2N messages. That is the per-item mail #238 objects to, and the "one summary" decision of D3.
- No criterion pins how the two notice sources relate. Either fold the driver's waits into the summary, or state the per-run notices as intended and bounded.

F3 — MEDIUM — D3 against D4, on the meaning of "owed".
- D3 records a target's first-owed time when a push fails as owed.
- Commits ahead of origin that wait out the interval, or that an agent made with plain `git commit` before the dobby's `StartIfAhead` starts a pusher, are not in the ledger. During that time the bar reads `sync: 0m` while work is only local, which defeats D4's stated purpose.
- "The ledger also keeps the time the hub last had no owed target" is one shared field that concurrent pushers would write. D3 puts per-target files under the hub's `.lym` and names no writer or lock for that field.

F4 — MEDIUM — D5's Worktree-Teardown against D1 and D2.
- "Worktree-Teardown pushes once with the interval wait skipped" leaves the mechanism open.
- If teardown pushes directly, as `PushUntilLanded` does today, it runs outside the one pusher, against D1.
- If it starts the pusher, the pusher's lock is non-blocking. A pusher already sleeping in its interval or backoff keeps the push, and teardown's "once, at once" push either does nothing or waits up to the ceiling.

The record should say which pusher pass the teardown triggers, and how it skips a wait another pass already holds.

F5 — MEDIUM — D5's PR wait against D6 and D7. Two gaps:
- "A landing that needs GitHub waits until its parent's owed landing push has landed" names no mechanism. Inside Finalize it would run under the parent lock, which D6 forbids. As a stop it needs a class D6 re-steps.
- If the parent's landing push is refused (D7: origin moved, or a rejection), the pusher stops until the record is cleared. The PR landing then waits with no bound and gets no notice of its own. D5 states no bound for this wait.

F6 — LOW — D3 against D7.
The record does not say whether a refused target stays in the ledger. If it stays:
- it gets the refusal notice and, 30 minutes later, a summary listing it again;
- the all-clear never comes until someone clears the refusal record.
The record should state either that a refusal leaves the ledger, or that the summary skips refused targets.

F7 — LOW — D7 against PATTERN-run-fsm's takeover.
- After a takeover, the new owner machine lacks the commits machine A pushed.
- D7's presence-in-this-repo test then refuses a reset that `resetUnnamedTip` makes today, such as for a webster reset done on machine A before the takeover. The run stalls on a refusal notice instead of converging.
- The record does not say this is accepted.

Tool calls made beyond my context:
1. grep of `internal/landingshed/finalize.go` for `pushRecords` and `pushParent`, to confirm whether the local landing path still pushes records synchronously after the merge. My context only had the subagent's line numbers, not the call order.
