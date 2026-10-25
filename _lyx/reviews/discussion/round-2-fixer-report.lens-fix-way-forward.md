```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every refusal, next step, row or test this fix introduced can be reached and makes progress.

**MEDIUM — D8 second bullet, "A judgment the refusal rejects takes the refusal's re-judge first … while the escalation record and decision file stay for the re-judged round … a refusal that holds over the unchanged review halts the segment at its budget"**
Summary: when the refusal holds over a recorded decision, the run halts with no way forward. The parent cannot replace the kept decision, and each resume re-judges the same review into the same halt.
Evidence:
- `internal/shedadapters/circling.go`, `RecordCirclingDecision`: "It returns ErrCirclingDecided when a decision file for the round already exists; the existing file is left untouched".
- So once a parent has recorded `accept`, they cannot switch to `continue` to send the BLOCKING `design` finding to a fixer.
- A re-judge runs no fixer, and the review does not change. A judge that keeps converging over that finding, or keeps dropping the key, hits the refusal again after every resume.
- The decision file stays, so the round never re-escalates either.
- Before the fix, a refused accept below the cap ran another round, and that round made progress.
- The brief's added note, "the round is then re-judged before the recorded answer acts", does not say how the parent gets out of a refusal that holds.
- Missing: a way forward that makes progress from that state, against `PATTERN-refusal-way-forward`. For example, a held refusal could retire the decision file after the re-judge, or the halt could name a verb that replaces it.

**LOW — D10 no-parent bullet, "so the gate exhausts and halts the run … with the gate's findings kept and the stop report in the change description for whoever resumes it"**
Summary: the new halt does not tell the person who resumes the run how to proceed.
Evidence:
- D10 says the stop report names only "the outdated entry and what overtook or contradicts it".
- What lets a resume pass is a `Decision for &lt;slug&gt;:` board note followed by `lyx loom resume`. D10's timeout line supports this: "a resume … repeats the check and finds an answer the parent left as a decision note".
- Neither the stop report nor the stencil instruction the fix added names that step. The exhausted gate's own findings name only the missing record.
- So a reader who resumes without writing a note sends the writer through the same stop and three more re-prompts.

Tool calls:
- `sed -n 40,100p internal/shedadapters/circling.go`: read whether a recorded circling decision can be replaced. My context held only the line numbers from grep, not `RecordCirclingDecision`'s contract.
