```yaml
lens: board-fit
outcome: reported
```

Lens: board-fit. I checked the record against every other board entry for planned work it duplicates or contradicts, and for dependencies it doesn't name.

1. MEDIUM: D1 (alignment agent goes whole) and Open risks/Out. The record never names note `regression-baseline-verify` (F18), which plans on the alignment agent as a consumer.
   - Evidence: F18's body says, for `expect-drift`, that "the drift report goes to the reviewers and the alignment agent, who judge whether it matches the task's goal". Under "On a failed compare" it says the report "goes to PR-Align, where the alignment agent judges it against the task's goal and escalates when it cannot tell".
   - D1 deletes that agent, so F18 loses the role it relies on.
   - The record's Out list has the generic "the orch rewrites other entries" line, but nothing tells the orch that F18's design needs a new judge, such as the Code perch's reviewers.

2. MEDIUM: D10 (darn writer checks its entry). It doesn't cover the planned `stitch`/piece runs of note `loom-beam-darns` (F20).
   - Evidence: F20 runs pieces "derived from darn", and its 2026-10-23 operator view says "darn itself may serve as the pieces' implementer".
   - A piece's scope is a pre-plan contract inside an umbrella that is aligned as a whole, and F20 says "alignment per piece is pointless".
   - D10 puts the board/peers check and the ask-and-wait into the darn writer stencil itself. A piece run on darn would then do a scope check F20 rules out, and could wait on its parent over sibling pieces that look like "overlapping in-flight runs" in `lyx loom peers`.
   - The record should say whether D10 applies to a darn run that has a parent beam, or note the interaction in Open risks.

3. LOW: Open risks, `umbrella-daemon` (T2). The record names the parent-review overlap but not the arbiter one.
   - Evidence: T2 folds in "The arbiter for every shed run": move the arbiter into shed, with its keys going into `shed.yaml`.
   - D8 adds `decision` as a cause `arbiterstop` refuses. If T2 moves eligibility, that refusal has to move with it, or the arbiter could settle operator-only questions.
   - The Open risks line about T2 should cover this too.

4. LOW: Out and Open risks. Note `creel-agents` (F16) loses part of its scope, and the record doesn't say so.
   - Evidence: F16's scope names "The parent-review gate (`internal/parentreview/gate.go` …) posts itself" and the stencil `loom-template-parent-review-delivery.md`. D2 deletes both.
   - The Out list names F16 only as "Moving parent messages onto creel".
   - New messaging D8/D10 adds also needs a line in F16: the Bouncer `decision` escalation notice, and the darn writer's `SendMessage` ask.

5. LOW: D2 (verbs removed). Note `run-verbs-shared` (E7) lists `review approve` among the recipe-neutral verbs to move, and names "parent review" as part of the shared layer above shed. D2 deletes both, so E7's open list shrinks. This isn't named in the record.

6. LOW: D5 (perch fan grows). This interacts with notes `lens-overlap-audit` (A6) and `review-setup-audit` (A3) without naming them as baselines.
   - Evidence: A6 recommends merging `goal-scope` and `board-fit` into one `board-scope` lens. D5 adds both separately to `review-discussion`, so a later trim has to change two fans.
   - A3 compares Discussion round-1 findings against a pre-change baseline, and states the Discussion writer's fan "runs every `review-discussion` lens". D5 keeps that true, since `twill-discussion` already holds both, but it changes the perch's round-1 makeup mid-audit.
   - A3 isn't named in the record. The Open risks line on fan growth could name both notes.

7. LOW: D11 (rename). Note `landing-escalate-arbiter` (E35) is about halts "at `Merge-In` or `Finalize`" and `landingshed.HandsBack`. The record lists it as out of scope, which is consistent. Its text will name a row that no longer exists, which falls under the orch's after-landing rewrite; that's acceptable as written.

No duplicates found: no other entry plans the Bouncer `decision` routing, the `board-fit` peers reading, the darn entry check or the rename.

Tool calls:
- One `lyx board get` grep over `regression-baseline-verify`, `review-setup-audit`, `claude-md-lyx-rules-out` and `finalize-discrepancy-doc`. My context held only the board list's one-line briefs for these four, not their bodies.
