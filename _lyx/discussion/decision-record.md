# Decision record — pr-gate-run-identity

## Goal

A loom or darn run's landing rows act only on what belongs to the run itself.
Publish, PR-Align, PR-Gate, Finalize and the `lyx loom approve` and `lyx loom reject` verbs never take a pull request opened before the run started, for a run whose seed carries its start stamp, so a reused slug cannot adopt the earlier run's closed pull request (issue #692).
The alignment agent's decision snapshot moves only on a real decision for the task, never on a board note that merely mentions the slug.

## Scope

In:

- `landingshed.FindPullRequest` and every caller of it: Publish, PR-Gate, PR-Align, Finalize's pull-request close, and the `findPR` seams of `lyx loom approve` and `lyx loom reject` in `internal/loomcli`.
  All callers are bounded on purpose, so no landing step finds a different pull request than the one Publish created or refreshed.
- The run's start time handed to those callers by `internal/loomcli`'s wiring.
- Publish's stop when an older open pull request blocks the create.
- `alignmentSnapshot` in `internal/loomcli/alignmentwiring.go` and the decision-note title form it recognises.
- The orch role stencil's decision-note rule, which renders that form, and `internal/orchengine`'s role render, which fills its template key.
- One declaration of `decisionRecordedReason`, removing the duplicate.
- `doc.go` of `internal/landingshed`, `internal/loomcli` and `internal/alignment` where they describe the pull-request lookup or the snapshot.

Out:

- batten's refusal of a slug whose earlier run completed (`internal/battencli/arm.go`): it already refuses and names `lyx batten cleanup --apply` and the branch deletion as the way forward, which is the issue's second accepted expectation; it stays as is.
- A board-side refusal of a reused slug: the board does not know whether an earlier run landed, and batten's refusal already covers it.
- The bouncer escalation stencil's own definition of a recorded operator decision: it serves the arbiter, not the snapshot.

## Decisions

### D1. A run's pull request is one created at or after the run started

`FindPullRequest` takes a lower bound, the run's start time, and returns the newest pull request from head to base, whatever its state, whose `created_at` is not before that bound; an older one reads as none.
The bound is the `started_at` stamp of the pair's own run seed, read through `shedrun.ReadSeed` with `shedrun.SelfRunID`, the same stamp `internal/hubgeom` already uses as the pair's start time.
`internal/loomcli` reads it once when it wires the landing deps and the approve and reject verbs, and hands it to `internal/landingshed` as a told value in `landingshed.Deps`, so landingshed derives no path.
Every caller of `FindPullRequest` passes it, so all landing steps keep resolving "the run's pull request" the same way.

Rationale: the seed stamp is written once when the run is seeded, and `WriteSeed` never rewrites an existing seed's stamp, so it separates this run's pull requests from an earlier run's with no new state file.
A pull request opened by hand during the run, as the orch did with #691, is the run's, and a resumed Publish finds its own pull request again.
The run starts hours before Publish, so clock skew between the machine and GitHub cannot misplace a pull request this run created.

Bound: the filter only narrows what a lookup returns.
A missing seed, or one with no stamp or a stamp that does not parse, gives a zero bound, which filters nothing and keeps today's behaviour; the wiring logs a warning when that happens.
A new pair for a reused slug never reaches the zero bound, given the cleanup the first Auto-mode assumption names:
the loom and darn bootstrap writes it a fresh stamped seed before the landing wiring first runs, and the approve and reject verbs read that same seed later.
Only a seed that predates the `started_at` field, which `WriteSeed` leaves byte for byte when it agrees, or a seed removed or a stamp edited by hand reaches it.
A seed that exists but cannot be read or decoded fails the wiring with an error, and the approve and reject verbs refuse with it, so an I/O fault never silently reopens the adoption.

### D2. Publish stops on an older open pull request it cannot replace

When the bounded lookup finds none but the head branch still has an open pull request to the base created before the run started, Publish does not create one, since GitHub refuses a second open pull request for the same head and base.
It stops Stuck with a reason that names that pull request's URL and ends in a `way forward:` clause: close that pull request on GitHub, then resume the run with `lyx loom start`, the resume verb Publish's push-rejected stop already names.

Rationale: D1 makes this stop necessary, because the bounded lookup no longer adopts such a pull request, and a create would fail with a raw GitHub error that names no way forward.
Such a pull request belongs to an earlier run that was abandoned before landing or whose Finalize close failed, which only warns; landing through it would land through another run's pull request.

Bound: the stop applies only to an open pull request older than the bound; a closed or merged older one is ignored and Publish creates a new one.

### D3. A board note counts as a decision for a held task only by its title

The decision snapshot hashes the decision record and the board entries of kind note whose title starts with `Decision for <slug>:`, the slug followed by a colon, case-sensitive.
A note that names the slug anywhere else, in its title, brief or body, no longer moves the snapshot, and an entry of another kind never does.
`internal/alignment` declares the title form once, as a function of the slug, and the snapshot filter calls it.
The orch role stencil's decision-note rule, the branch where `lyx loom decision add` refuses for want of a decision record, renders the title form from `internal/alignment`'s declaration through a template key, as it already renders `{{.command_index}}`, so the form has one declaration.
The role stencil is hub-wide, so the key carries the form for the literal placeholder `<slug>`.
The key joins the role render's required markers, so an operator-edited override without it fails the render naming the marker, with the existing way forward of restoring it or deleting the override, rather than rendering silently without the form.

Rationale: the orch already writes such a note only for a held task without a decision record, so a fixed title form is the one signal both the orch and Go can see; a decision added to an existing record already moves the record's bytes.
The trailing colon keeps `Decision for foo:` from matching slug `foo-bar`.

Bound: the snapshot now ignores notes outside the form, so a decision recorded as a free-form note no longer turns an approve into an escalation.
That includes free-form decision notes already on the board for a task held when this lands; they stop counting at landing.
A decision note the operator writes on the board without the `Decision for <slug>:` title, such as one the bouncer escalation stencil counts as an operator decision by its body, no longer moves the snapshot either, so an approve given while such a note is added is applied.
That is the bound: only the title form counts, so an operator who wants the agent's approve held titles the note in that form, or records the decision through `lyx loom decision add` when a decision record exists.

### D4. `decisionRecordedReason` is declared once, in `internal/alignment`

The reason text moves to one exported constant in `internal/alignment`, which both `internal/landingshed/pralign.go` and `internal/loomcli/alignmentwiring.go` already import, and both use it.

Rationale: the reason belongs to the snapshot check that `internal/alignment` owns, and neither consumer then imports the other for it.

## Constraints

- `PATTERN-told-geometry`: landingshed takes the start time as a told value; only loomcli reads the seed.
- `PATTERN-shed-run-directory`: the seed is read only through `shedrun.ReadSeed`.
- `PATTERN-refusal-way-forward`: the entry binds `internal/shed*`, `internal/loom*` and `internal/webster*`, not `internal/landingshed`; D2's stop follows its form by choice, with the `way forward:` clause `contracts/specs/refusal-spec.md` defines and a test that reaches it, and the entry is not changed.
- `PATTERN-test-economy`, `PATTERN-test-speed`: extend the existing table tests in `internal/landingshed` and `internal/loomcli/alignmentwiring_test.go` rather than add new top-level tests where a row fits.

## Auto-mode assumptions

- A new pair for a reused slug writes a fresh seed only because `lyx batten cleanup --apply` removes the earlier run's directory, `_lyx/shed/<slug>`, which the landing merged into the parent; `WriteSeed` would otherwise keep the inherited seed and its old stamp.
  batten's completed-slug refusal requires that cleanup before a new run starts.
- GitHub's list endpoint, sorted by creation date descending, returns the run's own pull request within its first page.

## Open risks

- D1 rests on the seed's stamp being this run's: a change that lets batten start a reused slug without removing the earlier run directory, or a hand reset that removes and rewrites a pair's seed mid-run, moves the bound and makes D2 call the run's own open pull request older, and makes PR-Gate, PR-Align, Finalize's close and the approve and reject verbs find no pull request, where PR-Gate's existing nil-lookup reason names reopening it, which cannot help.

<!-- lyx:carry-over Webster-Review -->
- Webster-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/webster/round-1-review.md; fixer report: _lyx/reviews/webster/round-1-fixer-report.md.
  - reseed-after-publish-strands-own-pr (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Webster-Review -->

## Acceptance criteria

- With a closed, unmerged pull request for the task branch created before the run started, Publish creates a new pull request and ends Done.
- With a merged pull request for the task branch created before the run started, Publish creates a new pull request rather than ending Done on the old merge, and PR-Gate does not end Done on it.
- With an open pull request for the task branch created before the run started and none after, Publish stops Stuck naming that pull request's URL with the `way forward:` clause "close that pull request on GitHub, then resume the run with `lyx loom start`"; PR-Align does not align it, and Finalize does not close it.
- A pull request created after the run started, including one opened by hand, is found by Publish, PR-Gate, PR-Align, Finalize and the approve and reject verbs; a resumed Publish refreshes it instead of creating a second.
- A pull request closed unmerged after the run started still stops Publish with "the pull request was closed without being merged".
- A zero start bound, from a missing seed or one without a parseable stamp, keeps today's lookup and logs a warning.
- A seed that exists but does not decode fails the landing wiring and makes the approve and reject verbs refuse.
- A note whose body or brief names the slug but whose title lacks the `Decision for <slug>:` form leaves the snapshot unchanged, so an alignment approve is applied.
- A note titled `Decision for <slug>: …` added while the agent runs changes the snapshot and turns the approve into an escalation with the reason.
- A note titled `Decision for <slug>-other: …`, or a task entry titled `Decision for <slug>: …`, does not count for `<slug>`.
- A change to the decision record still changes the snapshot.
- The rendered orch role stencil names the decision-note title form.
- `decisionRecordedReason`'s text is declared once in the module.

## Notes for the plan writer

- D2 needs the newest open pull request older than the bound from the same list response the bounded lookup reads, so `FindPullRequest`'s return shape grows rather than Publish making a second GitHub call.
- The `alignmentwiring_test.go` row "an approve after a note naming the slug" asserts today's behaviour and flips under D3.
- `internal/loomcli/doc.go` describes the snapshot as covering "every board note naming the task" in two places; both change with D3.
- `internal/landingshed/doc.go` and `FindPullRequest`'s comment describe the lookup as "the newest pull request ... whatever its state"; both gain the run-start bound.
