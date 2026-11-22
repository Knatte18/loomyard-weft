```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. For each decision I ran one concrete case step by step against the code in context.

**List: decision, case used, purpose met or not**
- D1. Case: a hub default of `true`, a beam whose worktree file says `false`, and a task with `run_settings` `{"beam":{"autonomy":true}}`. A Guard key is denied at the local layer, the worktree layer beats the default, and the fork-point read carries the file. Purpose met.
- D2. Case: beam B resolves `false`, B's orch is down, and the prime orch resolves `true`. `FromPair` walks up to the prime, and the notice carries B's `false`, so the prime relays the halt. Purpose met.
- D3. Case 1: the parent does not answer the judge ask by `judge_parent_timeout_min`. The run halts and the resume asks again. Met. Case 2: the judge's own time limit leaves no room for the ask (not-sent), or no parent orch resolves (no-parent). Purpose not met (F-W3).
- D4. Case: autonomous member, the row sends ask 1 at head H1, the agent fixes to H2, sends ask 2, and the orch runs `parent-review answer <slug> --ask 2 --satisfied`. Purpose not met: the answer verb reads a different store (F-W1). Case: the session is stopped and resumed around a wake read while autonomy is toggled. Purpose at risk (F-W4).
- D5. Case: Drift-Verify halts, the orch runs `lym loom resume`, it halts again the same way, resume, halts a third time. Purpose not met: the halts fold into one history entry, so class 5 never fires (F-W2).
- D6. Case: the last member lands, the beam has commits beyond its parent, and no ready record stands. The line is posted and the marker written. A re-entry finds the marker and posts nothing. A member reopening removes the marker, and the next drain posts again. Purpose met.
- D7. Case: drained, `pr open` refuses on a conflict, `merge-parent` files a released merge task, which lands and drains the beam again, then step 2 and step 3. This follows `judgeOpenable`'s order (open merge task first, then conflict). Each loop needs a fresh conflict on the parent, so it never repeats on lag alone. Purpose met.
- D8. Case: presence slack, the orch's worktree resolves `true`, and an unmarked reply about a settled halt. The hook drops it, and a relay also goes out by direct post. Purpose met.
- D9. Case: a hand-edited worktree file flips the value. The next halt notice and `lym config beam --resolved` read the new value, and the launch line only informs. Purpose met.
- D10. Case: the two glossary lines are added. Met.

**Findings**

F-W1: BLOCKING, design, `_lym/discussion/decision-record.md:128`.
D4's answer step cannot record the parent's answer in the pull request record, so autonomous PR-Review never ends.
- D4 has the orch answer with `lym loom parent-review answer <slug> --ask <n>` and says the verb records the answer "against that ask in the pull request record".
- The existing verb reads and writes only burlerengine's never-tracked ask record: `burlerengine.ReadParentAsks` and `AnswerParentAsk` (`internal/loomcli/parentreview.go:197-210`).
- `AnswerParentAsk` answers only the run's latest ask in that record, by number, and only before its deadline (`internal/burlerengine/parentreview.go:336-360`).
- In the walk, PR ask 2 is in the pull request record only:
  - If the burler record holds no ask, the answer is refused with no-ask.
  - If it holds the run's earlier Code-Review or judge asks, numbered 1..k, `--ask 2` collides with one of those or is refused as not the latest.
- So the satisfied answer is never recorded, and `pr-continue` refuses for good.
- D4 does not mark the answer verb as changed, does not say how it tells a PR ask from a review or judge ask in the same run, and does not say whether a PR ask carries a deadline. Every burler ask carries one (`parentreview.go:65`).

F-W2: BLOCKING, design, `_lym/discussion/decision-record.md:174-177`.
D5's class 5 count, two earlier halts at the row in the status history, never reaches two for a halt that repeats after a plain resume.
- A halt is a budget-exempt Stuck: `halt` sets `Halt` (`internal/landingshed/stuck.go:55-58`), and `appendHistory` marks it exempt (`internal/shedengine/run.go:369`).
- `appendOrFold` folds a new entry into the last one when both are budget-exempt Stucks with equal Producer, Output, RoutedTo and GateAttempts. It bumps `Repeats` and appends no entry (`run.go:706-739`). The reason text is not compared.
- `lym loom resume` writes no history entry; `RecordResumer` only sets `PendingBy` (`internal/shedengine/goto.go:287-307`).
- So halt, resume, halt, resume, halt at one row leaves one history entry with `Repeats: 2`. An orch counting entries sees one earlier halt forever, and the Bound's "two per row of a run" does not hold.
- D5 must name `Repeats`, plus `LastAt`, as part of the count, or count from another source.

F-W3: MEDIUM, design, `_lym/discussion/decision-record.md:98-104`. This may be F7 still open rather than a new finding.
D3's Bound covers only an ask the parent leaves unanswered past `judge_parent_timeout_min`. Under autonomy, the judge's rulings still stand with no one agreeing in two other cases:
- The ask is never sent because the judge's own time limit leaves no room (`ParentAskNotSent`, `internal/driftengine/judgeparent.go:170`).
- No parent orch resolves (`ParentAskNoParent`, `judgeparent.go:175-177,203-206`).

In both cases the judge's prompt says to end with its rulings as they stand (`judgeparent.go:254`). That contradicts D3's "only the parent agrees to them", since the judge's session was forced non-interactive.

F-W4: MEDIUM, design, `_lym/discussion/decision-record.md:117-120`. This may be F6 still open.
D4 records the run's autonomy "when PR-Review starts its session", but the row starts or resumes the session many times within one review:
- `ensureSession` builds a fresh `sessionSpec` on every resume (`internal/landingshed/prsession.go:64-101`).
- It is reached from `holdWhileCovered`, which stops and resumes the session around a pause (`prreview.go:493-523`).
- It is reached from `resumeSession` after every wake that stopped the session around a read that might end the review (`prreview.go:664-678`, `prsession.go:226-237`).

If each start records the value anew, a toggle takes effect at the next stop and resume, against D4's own acceptance bullet that a toggle changes nothing. If only the first start records it, D4 must say that a resumed session takes its `Interactive` from the recorded value.

The goto takeover has the same gap:
- A goto onto a waiting run is not admitted. `gotoAdmitted` admits only paused, blocked or failed runs (`goto.go:310-320`), so the operator must stop the run first.
- The re-entry then resumes the recorded `SessionID`; it does not start a new session (`prsession.go:80`).

Tool calls:
1. grep of `internal/landingshed/stuck.go` `halt` and the not-sent and no-parent states in `internal/driftengine/judgeparent.go`: to confirm that a halt's history entry is budget-exempt with nothing distinguishing it for folding, and which judge ask end states let the rulings stand. My context showed neither.
