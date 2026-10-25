```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the ways forward the fixer's change set introduced (fix-way-forward lens).

**MEDIUM: D8's refused accept has no exit at or past the hard cap.**
- Changed text: D8's new second bullet ("A judgment the refusal rejects is not passed: the accept retires … and the round is re-judged; the re-judged round escalates `decision` again"), read with the hard-cap bullet ("at or past it, accept only").
- Summary: At or past the hard cap the parent can only choose accept. If the re-judge returns the same CONVERGED verdict over the same BLOCKING review finding, the refusal rejects the accept again, and the round re-escalates with accept as the only option again. The review is unchanged, so nothing in the loop moves the run forward.
- Evidence:
  - `judgmentRefusal` (`internal/shedadapters/bouncer.go`) refuses CONVERGED whenever the review file carries a BLOCKING finding. A re-judge archives only `judgeOutputs`, so the reviewer's review still holds that finding.
  - `rejudge` spends a counted bounce unit and "a segment at its budget halts on it". After a halt, `lyx loom resume` re-enters the same cycle.
  - The new brief names the refusal cause, but at the hard cap it offers no action that avoids the refusal.
- What would fix it: give that case a way out, for example offer `continue` while the refusal would reject the judgment, or name the way forward in the brief.

**LOW: D8's "entries recorded since" anchor is reset whenever the escalation record is rewritten.**
- Changed text: D8's accept bullet ("The escalation record keeps the titles of the record's `Added after Discussion` entries at escalation time, and each carried finding names the entries recorded since").
- Summary: Two paths archive the escalation record and write a fresh one:
  - the hard-cap path (`retireContinue` followed by a fresh `escalate`);
  - the new refused-accept retire and re-escalation.
  The fresh record takes a new snapshot. Entries the parent recorded in answer before its `continue` or `accept` then count as "at escalation time", so a later accept carries the finding bare and drops the pointer to the parent's answer.
- Evidence: `retireContinue` (`bouncer.go`) moves the escalation record, the notice and the decision file aside, and `escalate` writes a new record.
- What would fix it: carry the original snapshot across a re-escalation, or anchor on the round's first escalation.

Tool calls: none; my context already held `bouncer.go`'s settle, refusal, re-judge and `retireContinue` code, and the changed decision record.
