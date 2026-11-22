```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked each pair of decisions that act on the same state, key, verb or record.

**Pairs checked**
- D1×D2 (the worktree layer gives the run reader's value): held.
- D1×D9 (Finalize's config notice leaves `autonomy` out, and the orch reads at call time): held.
- D1×D3 (Guard key; the forced keys follow the parent worktree's value and never `run_settings`): held.
- D2×D3 (the halt notice and the forced keys use one run reader): held.
- D2×D4 (call-time reads, except PR-Review's recorded value): broken, see F-I3.
- D2×D5 (a halt is handled under the value its notice carries): held.
- D2×D6/D7 (the beam's orch reads its own worktree's value while the beam's Finalize halt is judged under the run's value): held. The mix is stated, and both readings route to the operator correctly.
- D2×D8 (an autonomous orch relays the halts of a run that resolves `false`): broken, see F-I4.
- D3×D4 (forced `pr_review_ask_parent` and D4's asks share the `parent-review answer` verb with the judge's and the review rounds' asks): broken, see F-I1.
- D3×D5 (the expired judge ask halts at Drift-Verify, then resume and re-ask, counted toward class 5): broken, see F-I2.
- D4 `pr-continue` × D4 gate hint (mailed head re-gate split from the ending): held.
- D4×D5 (a closed pull request and a required human approval are class 4; hold notices re-raise an unanswered ask): held.
- D4×D9 (the "Reviewing a pull request" exception, and `role-autonomy.md` gives the answer): held.
- D5×D8 (the relay opens `[operator] severe halt`, which matches the hook's marker): held.
- D6×D7 (drain line → steps → merge-parent → reopen → drain again): held. Step 1 runs only on a conflict refusal, so each merge-parent cycle needs a new conflict. The loop is bounded by conflicts and never runs on lag alone. No count caps repeated conflicts on a moving parent, but each cycle is real work the operator would also have to do.
- D6 marker × D7 "first step not done" after a takeover: held. The extra line re-runs idempotent steps and re-notifies once, as D6's Bound states.
- D7×D8 (merge-ready notice is marked and posted directly): held.
- D9×D8 (the hook and the orch both read at call time): held.

**Findings**

F-I1 — BLOCKING, design, `_lym/discussion/decision-record.md:128` (D4, the answer bullet)
- **Summary:** D4 sends PR-Review answers through the existing `lym loom parent-review answer <slug> --ask <n>` and keeps the asks in the pull request record. That verb reads and writes only burlerengine's one ask record per run. It answers only that record's latest ask, and only before the ask's deadline; past the deadline the ask is set timeout and the answer refused.
- D3 forces `judge_ask_parent`, so the judge's asks fill that record, as the review rounds' asks already do. A PR-Review ask numbered 1 in the pull request record therefore hits the burler record's latest ask: "not latest" or "already done".
- The record never says the answer verb changes, which store it writes, how ask numbers from the two stores are told apart, or what deadline a PR-Review ask carries.
- **Evidence:** `internal/loomcli/parentreview.go:180-230` (`parentReviewAnswerVerb` calls `burlerengine.ReadParentAsks` and `AnswerParentAsk`). `internal/burlerengine/parentreview.go:336-358` (latest-only, deadline). `internal/burlerengine/parentreview.go:48-70` (one record per run; every ask carries a deadline).

F-I2 — BLOCKING, design, `_lym/discussion/decision-record.md:177` (D5, the class 5 count)
- **Summary:** Class 5 counts "two earlier halts at that row" in the status history. A halt is a budget-exempt Stuck, and `appendOrFold` folds a halt into the previous entry when both are budget-exempt Stucks with the same producer, output, `RoutedTo` and `GateAttempts`. It only bumps `Repeats` and `LastAt`.
- `lym loom resume` writes no history entry: `RecordResumer` sets only `PendingBy`. So halt → resume → the same halt at the same row leaves one entry.
- D3's new repeating stop shows it: an expired judge ask, resumed and expired again at Drift-Verify, never reaches two earlier entries. The orch then settles it without limit, which voids the Bound's "two per row of a run".
- The record must say the count includes `Repeats`, or count from another source.
- **Evidence:** `internal/shedengine/run.go:706-739` (`appendOrFold`, `foldsInto`). `internal/shedengine/run.go:359-370`, `416-418` (a halt entry is budget-exempt). `internal/shedengine/goto.go:287-306` (`RecordResumer` writes no history). `internal/shedengine/status.go:101-102` (`Repeats`).

F-I3 — MEDIUM, design, `_lym/discussion/decision-record.md:117-120` (D4, recording the autonomy at session start)
- **Summary:** D4 records the autonomy "when PR-Review starts its session" and says a goto "starts a new session". In the code, the row rebuilds the session spec on every `ensureSession`:
  - the resume after `holdWhileCovered`;
  - `resumeSession` after each stop that judges the pull request;
  - the resume of a recorded `SessionID`.
- Only a send-back clears the session fields, so a goto onto PR-Review resumes the recorded session rather than starting a new one.
- Either the value is re-recorded on those resumes, which breaks D4's "governs the review to its end" and the toggle criterion, or it is not, and D4's goto takeover is false. The record does not say which spec path reads the recorded value or what clears the session on a goto.
- **Evidence:** `internal/landingshed/prsession.go:64-102` (`ensureSession` → `sessionSpec` on each resume), `226-237` (`resumeSession`). `internal/landingshed/prreview.go:122-125` (only `settleSendBack` clears `StrandGUID`, `SessionID` and the other session fields), `503-506`, `675-678`.

F-I4 — MEDIUM, design, `_lym/discussion/decision-record.md:241-248` (D8, the backstop)
- **Summary:** D2's Bound has an orch whose worktree resolves `true` relay a `false` run's halts to the operator as today: design questions, the resume word. These are neither severe-halt relays nor merge-ready notices.
- With a slack presence, D8's hook then drops any such reply without `[operator]`, and D8 gives the direct `<shortname>:SLACK` post only to severe-halt relays and merge-ready notices.
- So the Bound's "a relay or notice the hook drops still reaches Slack through its direct post" does not hold for this message class, which D2 itself creates.
- **Evidence:** `_lym/discussion/decision-record.md:72-74` (D2 Bound), `241-247` (D8 direct-post scope). `internal/slackcli/hook.go:162-198` (the hook is the only Slack path for turn replies).

**Tool calls**
1. Grepped `internal/loomcli` for resume and goto history writes, to see whether `lym loom resume` appends a history entry. My context held only `RecordResumer`.
2. Grepped for writers that clear the PR record's session fields, to see whether a goto starts a new PR-Review session.
3. Read `internal/loomcli/resume.go`'s function list, to confirm resume writes no history entry.
