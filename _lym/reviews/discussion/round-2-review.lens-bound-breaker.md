```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every bound, guard and claim about existing code in `decision-record.md`.

## Claims attacked

- D1: Guard key refused by the local layer, so `run_settings` cannot make a run autonomous. **Held** (PATTERN-config-layers Categories).
- D1: Finalize's notice leaves `autonomy` out, so autonomy never travels upward. **Held** as a design statement (finalize.go:621-658 notice is per key).
- D2: the notice carries the run's value, so the opt-out holds at an ancestor or prime orch. **Held** (orchroute/walk.go:107-117 routes upward; the value travels on the notice).
- D2: a load failure reads `false`. **Held**: it falls back to asking the operator.
- D3 Bound: "only the parent agrees to the rulings"; an unanswered ask stops the run. **Broken** (F-B5).
- D4 Bound: hold notices raise an unanswered ask again. **Held**: hold.go:115 and :336 notify only when `!spec.Interactive`, and D4 sets `Interactive` false.
- D4 Bound: only a recorded satisfied answer at the current head ends the review. **Broken** twice (F-B2, F-B3).
- D4: the answer verb records against the ask in the pull request record. **Broken** (F-B1).
- D4: `parent-review ask` is refused outside a PR-Review session. **Held weakly**: parentreview.go's caller check "guards against a mistaken call, not a forged strand name". A forged ask only re-asks, so nothing is let through.
- D5 Bound: class 5 bounds settlements at two per row, counted from the status history. **Broken** (F-B4).
- D5 class 1: the `owner check:` prefix exists. **Held** (finalize_owner.go:19 `OwnerHaltPrefix`).
- D6: one line per drain; re-entry and a dobby-down cycle post no second line; a reopened member re-arms. **Held**: members run far longer than the one-minute wake ceiling, so the open state is seen.
- D7 step 1: `judgeOpenable` passes a beam only behind its parent and refuses on conflict. **Held** (pr.go:242-257).
- D7: a moving parent cannot make the drain, merge-parent, drain loop unbounded. **Held**: each cycle needs a new real conflict, and an open merge task cannot coexist with a drained line.
- D8 Bound: a dropped relay still reaches Slack through the direct post. **Held** for severe-halt relays and merge-ready notices, which are the only ones D8 claims it for.
- Constraint: `internal/shellguard/rows.go` holds no row against the named verbs. **Held**: grep for resume, decision, circling, amend, selfreport, beam and parent-review found none.

## Findings

**F-B1 — BLOCKING, design, decision-record.md:128-130 (D4, answer verb)**
- Summary: D4 has the existing `lym loom parent-review answer <slug> --ask <n>` record its answer in the pull request record. That verb writes only burlerengine's run-wide ask record and takes no PR-Review ask.
- How the verb works today:
  - It writes through `burlerengine.AnswerParentAsk` into the one ask record that `ParentReviewFilesOf` locates. Code-review, discussion-review and drift-judge asks share that record and its numbering.
  - It answers only that record's latest ask.
  - It refuses an ask past its deadline and sets it timeout, and "every ask that goes out carries" a deadline.
- Counterexample: the drift judge's ask is ask 1 in the burler record. The PR-Review row's first ask is numbered 1 in the pull request record. The orch runs `answer <slug> --ask 1 --satisfied`. The verb refuses it, because the latest burler ask is ended. If it accepted it, it would answer the judge's ask. No satisfied answer ever reaches the pull request record, so `pr-continue` never ends the review.
- D4 marks no change to the answer verb, names no store split, and gives PR asks no deadline rule.
- Evidence:
  - internal/loomcli/parentreview.go:180-230 and :320-326
  - internal/burlerengine/parentreview.go:49-70 (`Deadline` "every ask that goes out carries one") and :336-366 (`AnswerParentAsk`: latest only, deadline refusal)

**F-B2 — BLOCKING, design, decision-record.md:131 and :146-147 (D4, ending at the current head)**
- Summary: the satisfied-answer check lives only in `pr-continue`. The row ends the review on `Continue && clean && gated` without re-checking the ask's head. A commit after `pr-continue` therefore lands a head the parent never reviewed.
- Counterexample:
  1. The parent answers satisfied at head A.
  2. The agent runs `pr-continue`, which passes and sets `Continue`.
  3. The agent commits B. Since head ≠ `GatedHead`, `sessionStopWanted` leaves the session running.
  4. The next wake gates B and sets `GatedHead = B`.
  5. The following wake sees `Continue`, a clean tree and `gated`, and calls `endReview`. B was never asked about.
- Evidence: internal/landingshed/prreview.go:635-650 (gate sets `GatedHead`), :690-699 (`sessionStopWanted`), :880-885 (`judge` ends on `Continue`).
- Fix direction: under recorded autonomy, have the row's `judge` end on `Continue` only when the newest satisfied ask's head equals `head == GatedHead`, or have `pr-continue` record the satisfied head and the row require it.

**F-B3 — BLOCKING, design, decision-record.md:139-140 (D4, closed pull request)**
- Summary: D4 lets `pr-continue` on a closed pull request end the review with no answer. Finalize then reopens and merges that pull request. Under autonomy this overrides a close made on GitHub with no human and no parent, while D4 calls a closed pull request to reopen a class 4 severe halt.
- Path:
  1. The pull request is closed on GitHub.
  2. The non-interactive session cleans the tree and runs `pr-continue`, which checks no answer here.
  3. `judge` sees `StateClosed`, `Continue`, a clean tree and `gated`, and calls `endReview`, setting `ReviewEnded`.
  4. Finalize reopens a closed pull request "whose review the operator ended" and merges it.
- Evidence:
  - internal/landingshed/prreview.go:860-871
  - internal/landingshed/finalize_pr.go:242-249 and :358-360 (it halts only when `!rec.ReviewEnded`)
- Fix direction: under recorded autonomy, refuse `pr-continue` on a closed pull request, or leave `ReviewEnded` unset there, so the class 4 halt is reached.

**F-B4 — BLOCKING, design, decision-record.md:174-177 and :186 (D5, class 5 count; related to prior F10's fixed-differently)**
- Summary: class 5 counts "two earlier halts at that row" in the status history. Repeated identical halts fold into one `HistoryEntry`, and a resume appends no entry, so the entry count at the row stays at 1 and the bound never fires.
- Why the halts fold:
  - `appendOrFold` folds a budget-exempt Stuck into the previous entry when Producer, Output, RoutedTo and GateAttempts match, and only increments `Repeats`. Every Halt is budget-exempt.
  - `RecordResumer` writes only `PendingBy`, not history.
- Counterexample: a row halts with the same output, and the orch runs `lym loom resume`. Repeat as often as you like: the history keeps one entry with `Repeats: n`. Counting entries never reaches 3, so the run never reaches the operator.
- D5 never says to count `Repeats + 1` per entry, and no acceptance bullet covers a folded halt.
- Evidence: internal/shedengine/run.go:706-739 (`appendOrFold`, `foldsInto`), :355-360 (Halt is `BudgetExempt`); internal/shedengine/goto.go:287-307 (`RecordResumer`); internal/shedengine/status.go:101-105 (`Repeats`).

**F-B5 — MEDIUM, design, decision-record.md:98-105 (D3 Bound; related to prior F7)**
- Summary: D3's Bound says only the parent agrees to the rulings, but the parent never sees them when the ask goes out as not-sent or no-parent. With `judge_interactive` forced off, the rulings then stand with neither operator nor parent, and D3 stops the run only on an expired ask.
- When the ask never goes out:
  - not-sent: the judge's own timeout leaves no room.
  - no-parent: no parent orch resolves, for example a hub with no recorded shortname.
- In both states the parent-agreement gate passes, and D3's autonomy stop covers only "does not answer by `judge_parent_timeout_min`".
- Evidence:
  - internal/driftengine/judgeparent.go:97 ("A limit at or before now … recorded not-sent and the entry passes"), :103-104 ("an ask that never reached the parent: the judge goes on without it"), :170, :252-256
  - internal/hubgeom/parent.go:41-44

## Tool calls

1. grep for hold-notice senders and the `Interactive` gating: to confirm D4's hold-notice claim under `Interactive` false, which no read in context covered.
2. grep of `internal/shellguard/rows.go` for the named verbs: to check the Constraint's no-row claim; the rows file was never read.
3. grep of `finalize_pr.go` for closed-PR handling and of `internal` for `owner check:`: to follow the closed-PR path one hop past PR-Review and to check D5 class 1's prefix.
4. read of `judgeparent.go` spans 95-135 and 210-260: to see how not-sent, no-parent and timeout asks are judged; only the call site was in context.
