# Decision record — twb-d0-sc

## Goal

A mid-run scope correction in a darn run reaches every later row that judges the run.
Darn-Write writes a short decision record in loom's format before it writes code, `lyx loom decision add --by parent` appends to it on a darn run, and the Darn-Review perch and the alignment agent measure the run against the board entry as that record amends it.
The darn recipe gains a PR-Align row, so a darn pull request is settled by the alignment agent like a loom one, and reaches the orch at PR-Gate only on an escalation.

## Scope

In:

- The darn writer stencil (`contracts/stencils/darn/darn-template-write.md`): a step, before any code edit, that writes the decision record and commits it alone.
- A Darn-Write handoff gate that holds the row until the record is well-formed.
- A record-only check in `internal/discussionparser`, used by that gate and by `decision add` on a darn run.
- `lyx loom decision add` on a darn run: works once the record is committed, while Darn-Write runs too, and refuses with a darn way forward before then.
- The Darn-Review perch reads the record: the rubric `loom-rubric-darn-review.md`, the Darn-Burler-Review `fasit` text in `darn-recipe.yaml`, and a `carry_over` on Darn-Bouncer.
- A `PR-Align` row in `darn-recipe.yaml` between Publish and PR-Gate, and a darn PR-Gate measure in the alignment prompt.
- The decision record's format moves out of `loom-template-discussion.md` Step 5 into a spec both writers point at.
- The orch stencil's procedure for a parent correction on a darn run.
- The docs `CLAUDE.md` names, listed under Constraints.

Out:

- Editing or amending a claimed board entry's body.
- Splitting darn into Discussion-Write, Plan-Write and Webster rows.
- Loom's gap before Discussion-Write has written its record.
- A support log, parent review or Discussion-Review perch in darn.
- Reading the record in darn's Describe row: it describes the final diff, and its measure is unchanged.
- Moving run verbs out of `lyx loom` (`run-verbs-shared`): the texts this task writes name `lyx loom decision add`, and that entry renames them when the verbs move.
- Ranking darn rows in `lyx loom peers`' landing order (D7).

## Decisions

### D1 — Darn-Write writes the record first, at loom's path, in loom's format

Decision: On a spawn that finds no committed decision record, the darn writer reads the board entry and writes `_lyx/discussion/decision-record.md` (the path `loomengine.DiscussionDecisionRecord` declares) with the seven required sections, tersely: the scope as it reads it, the calls it makes, and `None.` where a section has nothing.
It commits the record alone, as its own commit, before any code edit.
A spawn that finds an uncommitted record (a writer killed mid-write) completes it and commits it before anything else.
A spawn that finds the record committed (a rework, a prior-work continuation) reads it and never edits it; it reads every `### Added after Discussion` entry as amending the board entry.
The writer never authors an `Added after Discussion` entry or a carry-over block.
Rationale: The operator's Direction asks for the record in the same session, before code, in loom's format.
The shared path lets `decision add`, the alignment snapshot and prompt, and the review carry-over work on darn without new path wiring.

### D2 — The record format moves into a spec both writers point at

Decision: A new `contracts/specs/loom-discussion-spec.md` pins the decision record's sections and compaction rules and the support log's sections, taken from `loom-template-discussion.md` Step 5, and says a darn run writes the decision record only.
It is registered in `contracts/specs/specs.go` like every other spec.
In `loom-template-discussion.md`, only Step 5's format block is replaced by a pointer to the spec; the rest of the stencil is untouched.
The darn writer stencil points at the same spec and restates none of it.
Rationale: `PATTERN-producer-pointer-rule` forbids the darn stencil from paraphrasing the discussion stencil's format, and a whole interview prompt is no format contract to point at.

### D3 — A record-only check, and a `decision-record` gate on Darn-Write

Decision: `internal/discussionparser` gains a check over the decision record alone: the file exists and carries the seven required H2 sections.
`Validate` keeps its contract and composes it with the support log's existence check.
`resolveGateSpec`'s closed gate vocabulary in `internal/shedrecipe/entries_gate.go` gains `decision-record`, over the record path alone; its doc and its error text grow to seven values.
`darn-recipe.yaml`'s Darn-Write lists it first, before `verify` and `description`, with `attempts: 3`; once spent, it halts the run for the parent like the row's other gates, and `lyx loom resume` re-spawns with a fresh budget.
Rationale: Without a gate, a writer that skips the step leaves `decision add` refusing for the whole run, which is the bug.
A darn run has no support log, so loom's `discussion` gate would always fail there.
Bound: the gate checks presence and headings only; it cannot tell that the record was written before the code, which the stencil alone asks for.

### D4 — `decision add` works on a darn run once the record is committed

Decision: The decision verb reads the run's recipe from its seed (`internal/shedrun`); a seed it cannot read refuses with the existing retry way forward.
On a darn run it appends through the record-only check, with no support log required.
It refuses when the record is absent, or present but not committed at HEAD, with one darn way forward: Darn-Write writes and commits the record at its start; message the Darn-Write session with the call, and re-run once the record is committed.
The refusal for a running Discussion-Write stays loom's; a running Darn-Write does not refuse.
The caller guard, append-only entry and discussion-only commit (`commitDiscussion`'s pathspec) are unchanged.
The `decision: no decision record` row of `contracts/specs/refusal-spec.md` is rewritten for darn in the same commit, a row is added for the uncommitted record, and each carries a test.
The orch stencil (`contracts/stencils/orch/orch-template-role.md`) tells the parent, for a correction on a darn run, to record it with `decision add` and also message the running Darn-Write; its fallback to a board note keeps the pre-record rows and drops "any darn run".
Rationale: In darn the writer runs for the whole code phase, which is when the incident's correction came; refusing there would leave the gap open.
A message alone reaches only the live writer, and the record alone reaches only the later rows, so the parent does both.
Bound: the new edge lets the parent append while the writer runs; requiring a committed record keeps the append off the writer's draft, the writer never edits a committed record (D1), and the verb commits only the discussion directory, so it cannot sweep the writer's uncommitted code.

### D5 — The Darn-Review perch measures against the amended entry

Decision: The board entry's "the Darn review reads the record and holds it authoritative over the board body" is read as: the record's `Added after Discussion` entries, by the parent, the operator or the arbiter, win over the board body where they differ.
`loom-rubric-darn-review.md`'s measuring stick becomes the board entry as amended by those entries.
The record's other sections are the writer's claim, checked against the board entry like the change description, never the measure.
The record is never the subject of a finding, and the rubric tells a fix never to edit it.
Darn-Burler-Review's `fasit` instructions in `darn-recipe.yaml` say the same and name the record's path.
Rationale: The writer's own sections are written before any review, by the session under review; taking them as the measure would let a writer license its own scope.

### D6 — Darn-Bouncer carries open findings into the record

Decision: Darn-Bouncer gets `carry_over: Darn-Review`, so the findings still open when the perch closes land in the record's `## Open risks` as Go-written blocks.
The recipe comment that says darn has no record to carry over into is replaced.
Rationale: The board entry asks what the alignment agent reads that darn lacks, and the `pr-gate` read list names the record's carried-over open risks; the comment tied their omission to the missing record, which D1 adds.

### D7 — A PR-Align row in the darn recipe

Decision: `darn-recipe.yaml` gets `PR-Align` (engine `PRAlign`) with Publish's `on_done` pointing at it and its own `on_done: PR-Gate`, no `on_stuck`, outside the PR-Review segment, as in loom.
`landingAlignmentDeps` already builds PR-Align's seams for every recipe; the darn wiring changes only the prompt input: the plan directory and the support log read `none`, and the PR-Gate measure is darn's (D8).
`loom.yaml`'s `skip_alignment` and `alignment_pr_approve`, the hub-wide keys, govern darn the same way.
An agent reject routes through PR-Gate's existing `on_stuck: Darn-Write`, once per run; it is measured against the amended entry, so it never changes scope and records nothing.
`lyx loom peers` ranks loom rows only, so the landing order of an overlap involving a darn run reads as unknown order, which the alignment template already handles and which never changes a verdict.
Rationale: What darn lacked was the record and the plan the `pr-gate` judge reads; D1 supplies the record.
Bound: an agent approve now lands a darn task and batten's Land row deploys with no orch turn, bounded as in loom by `alignment_pr_approve`, the decision snapshot and head checks, and one agent reject per run.
With `darn.yaml`'s `review_max_bounces: 0` no review perch runs, and the alignment agent is the only judge of the diff before that landing.

### D8 — The `pr-gate` measure is chosen per recipe

Decision: The alignment template's `pr-gate` judge text becomes a marker filled from one of two fragment stencils the wiring picks by recipe, each registered in `contracts/stencils/stencils.go` and read at call time from the told stencils directory.
Loom's fragment holds today's text unchanged.
Darn's fragment: measure the diff against the board entry as amended by the record's `Added after Discussion` entries, which win over the body; the record's other sections are the writer's claim, never the measure; a scope deviation is a change the amended entry does not call for or a point it leaves out; and an open risk carried over from Darn-Review that names an unrecorded parent correction is escalated, never approved or rejected.
Rationale: Today's text measures against "the decisions recorded in the decision record" and Merge-Review's open risks, both loom-shaped; in darn the writer's decisions are unreviewed (D5).
Stencils carry no conditionals, and `PATTERN-stencil-ownership` keeps text out of Go.

### D9 — An in-flight darn run is restarted, not migrated

Decision: A darn run parked on Publish, PR-Gate or Finalize when this lands is restarted, as the recipe header already says for a row whose edges change.
Rationale: Publish's `on_done` changes, and such a run holds no record for PR-Align to read.

## Constraints

- `PATTERN-sole-parsers`: only `internal/discussionparser` checks the record; the darn-or-loom choice of check stays in `loomcli` and `shedrecipe`, which pass it paths.
- `PATTERN-stencil-ownership`: the fragments and the spec are registered seed defaults read at call time, never text in Go.
- `PATTERN-producer-pointer-rule`: both writer stencils point at the discussion spec (D2).
- `PATTERN-refusal-way-forward`: the darn refusals land with their `refusal-spec.md` rows and tests (D4).
- `PATTERN-no-denied-recovery`: the darn way forward names messaging Darn-Write and re-running the verb, no denied command.
- `PATTERN-told-geometry`: `discussionparser` and `alignment` keep taking told paths.
- `CLAUDE.md` "Docs land in the same commit": the darn recipe header comment; `doc.go` of `darnengine`, `discussionparser`, `alignment` and `loomcli` where they describe darn or the record; `docs/overview.md`'s darn paragraph and darn tail.
- Test budget: a package pushed past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- Only `Added after Discussion` entries outrank the board body (D5).
- `decision add` must work while Darn-Write runs (D4).
- Darn gets no support log (D3).
- Darn-Bouncer's `carry_over` is in scope (D6).

## Open risks

- The writer or a Darn-Burler-Fix round can edit the committed record and drop an `Added after Discussion` entry or a carry-over block; `discussionparser.Validate` at HEAD does not check entries against history, and the stencil and rubric alone forbid it.
- A parent append whose commit races the writer's own `git commit` fails one of the two; the verb reports the appended-but-uncommitted state with its way forward, and the writer retries its own commit.
- `twill-fork0` (at Plan-Write) also edits `loom-template-discussion.md`, `orch-split` will restructure the orch stencil, and `shed-tackler` touches `internal/loomcli/darnwiring.go`; each is a text overlap this run takes at Merge-In if they land first.

## Acceptance criteria

- A darn run's Darn-Write session writes and commits the decision record before its first code commit, and its handoff is held by the `decision-record` gate until the record carries the seven sections.
- A Darn-Write spawn that finds an uncommitted, partial record completes and commits it; a spawn that finds a committed record reads its amendments and leaves the record unchanged.
- The `decision-record` gate re-prompts on a missing record and on a missing section, passes a well-formed record with no support log present, and halts the run for the parent once its attempts are spent.
- `lyx loom decision add --by parent` on a darn run: refuses with the darn way forward when the record is absent or uncommitted; appends and commits only the discussion directory when it is committed, while Darn-Write runs and after.
- On a loom run `decision add` is unchanged: it still needs the support log and still refuses while Discussion-Write runs.
- The Darn-Review rubric and fasit name the amended board entry as the measure; a diff that follows a recorded parent amendment is not a scope finding.
- Darn-Bouncer's still-open findings land in the record's `## Open risks` as a carry-over block.
- The embedded darn recipe routes Publish → PR-Align → PR-Gate, an agent reject at PR-Gate reaches Darn-Write, and the darn routing with `review_max_bounces: 0` still builds.
- The alignment prompt of a darn run reads `none` for the plan directory and the support log and carries darn's measure; a loom run's prompt is unchanged.
- A loom Discussion-Write run still produces a record `validate-discussion` passes, with its stencil pointing at the spec; the darn writer stencil restates none of the spec.

## Notes for the plan writer

- `landingAlignmentDeps` in `internal/loomcli/landingdeps.go` builds PR-Align's seams; `wireDarn` in `internal/loomcli/darnwiring.go` fills darn-only seams.
- The decision verb is `internal/loomcli/decision.go`; its `decisionWayNoRecord` names darn today, and its paths are bound in its pre-run with no recipe input.
- The orch stencil's fallback clause is found by searching "any darn run".
- Tests that pin stencil wording are likely to need updating for the stencil edits.
