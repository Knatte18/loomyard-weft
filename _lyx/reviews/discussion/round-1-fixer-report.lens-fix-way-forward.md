```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, every way forward the round-1 fix introduced into `decision-record.md`.

Ways forward checked:
- D5: at a `Drift-Await` halt, `lyx drift capture` then `lyx loom resume`. Reachable: the D4 guard admits a halted `Drift-Await` on the branch point, and the resume re-reads a passed record.
- D5 adopt paragraph, `awaited_at` unset: the halt on the missing record leads to capture and resume on the adopting machine. Reachable, since no code has changed before Webster.
- D5 adopt paragraph, `awaited_at` set: `Drift-Verify` halts naming `lyx loom goto Describe`. Reachable.
- D5/D6: `lyx loom goto Webster` defers the question to a second halt at `Drift-Verify`. Reachable, with one problem (finding 2).
- D6 full-scope decision halt: `lyx loom resume` to stand by the acceptances. Reachable: the resumed attempt's mechanical compare skips the accepted tests and returns Done.
- D6 full-scope decision halt: `lyx drift withdraw` and resume to reject an acceptance. Reachable, but it does not reliably make progress (finding 1).
- D6 full-scope failing final compare as a decision halt: the parent's accept and resume, or fix and resume. Reachable.
- D6: `compare` and `capture --side` refuse outside `Drift-Verify`, and the counter resets at each fresh spawn. Reachable. At a halt past the budget, the resume's mechanical compare does the job.
- D7: `lyx drift withdraw` row guard. Reachable at a `Drift-Verify` halt.
- Acceptance criteria rows added or rewritten: each matches a reachable state above.

Findings:

1. MEDIUM. D6, the new sentence "`lyx drift withdraw` and resume to reject one", with D7's withdraw verb.
   The parent's rejection does not last.
   After the withdraw and resume, the compare fails and a fresh agent spawns with a reset budget. Under D7 that agent may run `lyx drift accept` on the same test again, which ends in the same decision halt.
   Evidence: D7 sets no bar on accepting a test again after it was withdrawn. D6 resets the counter "at each fresh spawn of its agent".
   So the "reject" way forward can send the parent around the same halt again, with no way to make the rejection hold.

2. MEDIUM. D5, the new sentence "After `lyx loom goto Webster` past a failed or missing capture, `Drift-Verify` halts a second time". The review's Rejected section says F8's fix covers the extra walkthrough claim.
   The goto skips `Drift-Await`, so `awaited_at` is never written.
   Every later Webster reroute goes back through `Plan-Bouncer` → `Drift-Await`. That row reads the failed or missing record with HEAD moved, and halts again naming `lyx loom goto Webster`.
   The chosen option (a) defers only the `Drift-Verify` question. It does not stop the repeated `Drift-Await` halts the review said F8's fix would cover.
   Evidence: the loom recipe has Webster `on_reroute: Plan-Burler-Review` (recipe header line 26, line 350). D5's `awaited_at` sentence covers only reroutes after a passed await.

3. LOW. Acceptance criterion "After an adopt on another machine, … with it set `Drift-Verify` halts naming `lyx loom goto Describe`".
   It is written for loom only. In darn, D6 names `Code-Bouncer`, and darn has no `Drift-Await`. A reader may take the criterion to cover darn and expect `Describe`.

Tool calls: none.
