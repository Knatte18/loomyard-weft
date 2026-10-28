```yaml
lens: interactions
outcome: reported
```

**Pairs checked**
- D1 × D2 (one Finalize merge path; flags, `taskMerger` and overlap note removed): held.
- D1 × D3 (the in-place large merge's finalize-mode fixer can escalate, and gets D3's prefix on loom and darn): held.
- D1 × D4 (the arbiter's resume with goto Finalize re-merges under the parent lock, gates and re-runs the fixer): held.
- D2 × D3 (`landingDeps` loses two fields and gains `EscalateJudgedFirst`; Notes say the drift-guard test follows both): held.
- D3 × D4, loom and darn (the escalate predicate feeds `loomArbiterSeam`; `HandsBack` false, so the hand-back row yields and the arbiter-first row fires): held.
- D3 pawl × D4 (`EscalateJudgedFirst` is false, so `LandingStopPrefix` makes the predicate false, the hand-back row fires first and the arbiter's `decision add` refusal on pawl is never reached): held.
- D3's batten re-step skip × D4's hand-on (after the arbiter hands on, the parked escalate is skipped by batten's re-step, and only `lym loom resume` moves it): held.
- D3's batten re-step skip × D4's driver build hand-over (the driver's hand-over row still re-steps once, and D4's Bound states it): held.
- D4's escalate-resume loop × D1's bound "the fixer's attempts and caps bound its unreviewed change" (the escalate gives back its turn; D4's Bound names the arbiter spawn cap as the loop's stop): held.
- D4's Reason-or-Error rule × the stop-building paths (envelope stop from a step or `resumeOpenAttempt`: `Reason`; `recheck` moved stop: `Error`): held.
- D4 × AC 7, its "starts" clause: broken (F1).
- D3 × Constraint "older `LandingStopPrefix` escalate halts stay hand-backs": held.
- D3 × Constraint `PATTERN-refusal-way-forward` (the loom and darn way forward names `decision add` plus resume; the pawl one names settling on the branch plus resume): held.
- D4 × PATTERN changes, `PATTERN-shed-stop-chain` "by its row and reason": held.
- D5 × D1–D4 (the docs list covers the orch stencil, recipe comments, rubric, fixer spec, overview and the doc.go files the decisions touch): held.
- AC 9 (pawl) × D3 pawl branch: held.
- AC 10 (other Parent-Align and Finalize blocked stops meet the tackler, and Webster goes to the arbiter first) × D4: held.

**Findings**

F1: LOW, consistency, `decision-record.md` Acceptance criteria, the driver bullet ("a driver that starts or re-checks at a halted escalate stop, built from the status, also sends it to the arbiter with no tackler call").
- **Issue:** The "starts" half contradicts D4's own Bound and the driver's start path.
  - A driver that starts at a blocked stop with no open attempt runs only `checkBlockedStart`. That check builds a status stop only to hand back on a `StartCheck` refusal (`internal/sheddriver/drive.go:618-629`). Otherwise the driver calls `Step` (`drive.go:187-191`).
  - Stepping a blocked status re-calls its current producer (`internal/shedengine/run.go:221-223`), so Finalize and its fixer re-run, and the stop decided is a fresh envelope stop.
  - A start that finds an open arbiter attempt rebuilds the stop from the attempt's envelope (`drive.go:648-689`), not from the status.
  - D4's Bound already says a new build "re-steps the escalate stop once before the arbiter sees it". So no start path decides a status-built escalate stop, and a test written to this criterion would pin a path that does not exist.
  - Only the re-check path (`recheck`, `drive.go:776-781`) and `handBackFailure` build a status stop. Of those, only `recheck`'s moved stop reaches `decideDrive`.
- **Fix:** Reword the criterion. A re-checking driver that finds the run moved to a halted escalate stop sends the status-built stop to the arbiter with no tackler call. A driver that restarts with an open arbiter attempt for the stop reuses that attempt.

Tool calls: none; my inherited context held every definition this lens needed.
