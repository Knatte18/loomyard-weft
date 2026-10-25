```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision's mechanism run end to end, checking that each step gets the input it needs and that the end state meets the decision's purpose.

**Findings**

1. **BLOCKING — D8 and its acceptance criteria: escalating over a CONVERGED verdict collides with the Bouncer's entry-time settled path.**
   D8 escalates "whatever the verdict" and assumes every later Call replays the Awaiting or acts on the recorded decision.
   That is not what the code does when the judge's verdict was CONVERGED:
   - `settledGeneration` (`internal/shedadapters/bouncer.go:833`) returns true for any round whose recorded verdict is CONVERGED. The circling-decision check only runs for other verdicts.
   - So the next Call goes into `settledEntry` before the escalation is ever read. This happens on a resume after `lyx loom circling continue` or `accept`, and on a driver restart.
   - If the parent ran `lyx loom decision add`, the committed record changes the live subject. The doc says a generation record whose judge subject differs from the live one "start[s] a new generation". That archives the run directory, which holds the escalation record and the parent's continue or accept file. An amendment generation then runs under `review_amend_max_bounces` instead.
   - If the record did not change, the case is `entryHarvest`, which calls `settleUnlessRefused` and settles the CONVERGED verdict. Nothing in the record makes that path honour the pending escalation.
   - With no generation record, it is `entryClear`: archive and re-seed.
   - The CONTINUE and CIRCLING cases do follow D8's described path, because `settledGeneration` is false for them.
   - The record is silent on all of this. These acceptance criteria break for the CONVERGED case: "re-call returns the same Awaiting", "continue … runs another round" and "accept settles …".
   - Fix: the decision escalation must either keep the round out of `settledGeneration`, for example by rewriting the verdict or having `settledGeneration` read a pending escalation, or D8 must state how the settled-entry cases treat an escalated CONVERGED round.

2. **MEDIUM — D8 accept path: a LOW `decision` finding is not carried into Open risks.**
   D8 fires on a `decision` finding of any severity. The acceptance criterion says accept "carries the finding into `## Open risks`".
   But the carry-over only lists ledger findings that are open at MEDIUM or worse, or unlabelled (`internal/shedadapters/doc.go`, Carry-over).
   A LOW `decision` finding that triggered the stop is withheld from the fixer and then dropped on accept, so the parent's accept leaves no trace for Plan-Write.
   Fix: either carry `decision` findings at any severity, or reword the acceptance criterion and record the bound.

3. **LOW — D10 timeout path: the writer re-asks after a timeout.**
   If `writer_timeout_min` expires while the darn writer waits, the respawn still has prior work `(none)`, because nothing was committed.
   So the first-spawn condition holds again, and the writer re-reads the board and asks its parent again.
   That is harmless but unstated. The parent's earlier `SendMessage` answer is lost with the old session.
   The record should say the answer survives only by message or a `Decision for &lt;slug&gt;:` board note.

**Decisions walked**

- **D1:** case is the orch role render and `decisionWayNoRecord` after the package is deleted. `DecisionNoteTitle` moves; purpose met.
- **D2:** case is a run held in a parent-review round at upgrade. The writer's outputs exist, the gate list is only `discussion`, so the run passes to Discussion-Bouncer; purpose met.
- **D3:** case is a hub `loom.yaml` holding `skip_alignment`. Reconcile drops it before the strict load; purpose met.
- **D4:** case is the wakes report over an archive tag holding `round-1/delivery.json`. The read-only parse stays in reportengine; purpose met.
- **D5 and D6:** case is the default fan `["review-discussion", ""]`. Round-1 forks run `board-fit`, which calls `lyx loom peers` from the task worktree and gets the parent commits; purpose met.
- **D7:** case is a reviewer that would file the writer's choice of a key name as `decision`. The rubric's segment definition excludes it; purpose met.
- **D8:** case 1 is a CONTINUE round with one `decision` finding: Awaiting, then continue gives Stuck and a new round; purpose met. Case 2 is a CONVERGED round with one `decision` finding followed by `decision add` and `continue`: the run directory is archived as a new generation; **purpose not met** (finding 1).
- **D9:** case is a writer's `board-fit` fork reporting the entry overtaken. It goes into Open risks, and the perch raises it as `decision`; purpose met, subject to D8.
- **D10:** case is a first spawn whose entry is overtaken by a landed commit. Darn-Write's only declared output is the description path (`darnengine/spec.go:116`), so a turn end before any write is held, and the answer goes into the record; purpose met (finding 3 aside).
- **D11:** case is a hub `gate.yaml` holding `merge-in: {tiers: [0]}`. Reconcile renames the key and gateslot's `landing` call reads `parent-align`; purpose met.

**Tool calls**

1. Grepped `Bouncer.Call` for the ordering of clear and escalation: my context had the docs, not the entry code.
2. Grepped darnengine for output files: needed to know whether a turn end before any write is held.
3. Read `bouncer.go:395-470` (the entry-time settled, clear and harvest branches): needed to trace a re-call over an escalated round.
4. Grepped for how Darn-Write declares its outputs: found `Outputs: DescriptionPath`.
5. Read `settledGeneration`: needed to know whether a CONVERGED verdict alone counts as a settled generation.
