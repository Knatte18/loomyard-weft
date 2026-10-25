```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I tested each proposal the fixer carried out by trying to build a case where it is wrong.

Proposals checked: F1, F2, F3, F4, F6, F7, F8, F9, F10, F11 and F12.
I built breaking cases for F1, F8, F10, F3 and F9.
F2, F4, F6, F7, F11 and F12 held: for each, I could not build a case that breaks it.

**MEDIUM: F1's re-judge cannot fix the refusal it reacts to.**
- Changed text: D8, second bullet: "A judgment the refusal rejects is not passed: the accept retires … and the round is re-judged; the re-judged round escalates `decision` again".
- Case: a CONVERGED round whose review holds a BLOCKING `design` finding. `judgmentRefusal` rejects it for "review finding %q is BLOCKING", and that cause comes from the review file, which `rejudge` never touches. It archives only the judge outputs (`archiveStaleOutputs(judgeOutputs(...))`). A judge that converges again gets the same refusal.
- The loop is: escalate, parent accepts, re-judge, escalate again. Each pass costs the parent a decision and one counted bounce unit, and it ends in a budget halt ("a segment at its budget halts on it", `rejudge` doc).
- Only `continue` makes progress, but the bullet presents the re-judge as the remedy. It should say that an accept over a review-BLOCKING refusal is refused, with `circling continue` as the way forward, or that it runs as a continue.

**MEDIUM: F8's no-parent path builds a plainly outdated entry with nothing to stop it.**
- Changed text: D10, "With no parent, it does not wait … works the entry as written".
- Case: the entry was overtaken by landed work, or another board entry contradicts it, which is D6's "plainly outdated". The writer now builds the work anyway.
- Darn's Code-Review perch takes the board-entry snapshot as its scope (D10 rationale, darn-recipe Code-Burler-Review fasit). So it reviews the outdated work as in scope, and the run can land work that duplicates or contradicts what the parent already has.
- The text this replaced, "stops and reports" (matching the stencil's existing "With no parent to ask, stop and report the step"), never built the work. The review's other option, "write the change description … as its stop report", ends the run without building anything; this one trades the silent hold for a wrong landing.

**LOW: F10's snapshot is lost whenever the escalation record is retired and written again.**
- Changed text: D8, accept bullet: "The escalation record keeps the titles … each carried finding names the entries recorded since".
- Case: after the first escalation, the parent runs `lyx loom decision add` and then accepts. The F1 refusal path retires the record (the `retireContinue`-style archive) and the round escalates again with a fresh record. That record's snapshot already includes the parent's answer, so the next accept sees "none recorded since" and carries the finding bare.
- That is the answered-but-open Open risks line F10 was meant to prevent. F6's hard-cap re-escalation, which also runs `retireContinue`, loses the snapshot the same way.

**LOW: F3's cap of four notes can still stop a run on a question already answered.**
- Changed text: D6, `goal-scope` "up to four `Decision for &lt;slug&gt;:` notes for it, nearest first".
- Case: a fifth, older note answers a question. `goal-scope` never reads it and raises a `decision` finding. D8 escalates "whatever the judge ruled", so the judge seeing the reach in the lens's list does not stop the escalation.
- The added "What a fork can miss" line names this miss, but D8's bounds line does not count it among what can stop a run. "Nearest first" is also undefined for notes: newest, or closest in topic?

**LOW: F9's "knowingly" claims an awareness nothing provides.**
- Changed text: D5, "such a hub runs its old text in `review-discussion` until it drops the entry, knowingly".
- The same sentence says reconcile reports nothing for a name the template holds (lenses doc: "Every other entry is kept as written"). With no report, nothing tells the hub its tuned emphasis text now runs inside a production fan.

Tool calls:
- grep of `internal/landingshed/mergein.go` and `fixerrun.go` for `RemoveFixerAttempts` and `FixerAttemptsPath`: my context did not show whether Merge-In removes its own attempt record, which F4's "removes the old record wherever it removes its own" depends on. It does, at mergein.go:208.
