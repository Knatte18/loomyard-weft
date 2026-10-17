---
verdict: APPROVED
---

## Alignment at `pr-gate`: pull request 710 at `a27e7b3d7`

The diff carries out the board entry's three points and the record's D1 to D4, and it adds nothing the goal does not call for.

- Issue #692, a reused slug adopting an older PR (D1): `FindPullRequest` takes the run's start and skips a pull request created before it.
  Publish, PR-Gate, PR-Align, Finalize's close and the `findPR` seams of `lyx loom approve` and `lyx loom reject` all pass the bound.
  `internal/loomcli` reads the bound once from the seed through `shedrun.ReadSeed` with `shedrun.SelfRunID` and hands it to `landingshed.Deps.RunStart` as a told value.
  A missing seed or a missing or unparseable stamp warns and leaves the lookup unbounded; a seed that cannot be decoded fails the wiring and makes the verbs refuse.
- D2: Publish stops Stuck on an open pull request older than the bound, naming its URL, with the recorded `way forward:` clause.
  The older open pull request comes from the same list response, as the plan-writer note asks.
- Note-mention trigger, the entry's second point (D3): `alignmentSnapshot` counts only notes whose title starts with `alignment.DecisionNoteTitle(slug)`, `Decision for <slug>:`.
  The orch role stencil fills a new required marker `decision_note_title` from that one declaration.
  `decisionWayNoRecord` in `internal/loomcli/decision.go` changes too.
  It is the only other text that told a caller to record a free-form note, plan card 02 lists it, and it uses the same declaration, so it stays inside D3.
- D4: `DecisionRecordedReason` is now declared once, in `internal/alignment`, and both consumers use it.
- The doc changes cover `internal/landingshed`, `internal/loomcli`, `internal/alignment` and `internal/orchengine`, as the scope names.
- The Out list holds: the diff leaves batten's completed-slug refusal and the board alone.

Open risks carried over:

- The record's open risk is a re-seed after Publish moving the bound.
  The carried-over Webster-Review finding `reseed-after-publish-strands-own-pr` says the same.
  The round-1 fixer addressed it in `wireLanding`'s error text in `internal/loomcli/sharedbootstrap.go`, which says a re-seed strands the run's own pull request and how to restore `started_at`.
  The risk is accepted in the record and contradicts no decision.

## Overlap

`lyx loom peers` lists no parent commits (own row 22).
None of the three peers changes a path this run changes:

- `gate-wait-cache-cap` (row 0) changes `internal/gatecli`, `internal/gateslot`, `internal/shuttleengine`, `internal/shellguard` and Webster stencils.
- `stencilstore-dev-warning-noise` (row 0) lists no changed paths yet.
- `creel-cli-output-fixes` (row 4) lists no changed paths yet.

Their topics, the gate slot wait, stencilstore warnings and creel CLI output, do not touch the pull-request lookup, the alignment snapshot or the orch role's decision-note rule.
No overlap means no landing order to apply.
`landing-trailer-text` depends on this task and is not running; this run does not make it obsolete.
