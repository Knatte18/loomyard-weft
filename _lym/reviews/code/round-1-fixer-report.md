# Fixer-report — round 1

All six findings are fixed. The gates for `internal/sheddriver`, `internal/shedarbiter`, `internal/shedadapters`, `internal/shedrecipe` and `internal/loomcli` pass at tier 0.

## Changes

- **F1** (commit `dfe77aed`): `arbitrate` no longer sends the outcome line.
  It records the awaiting stop and the verdict path in `goDriver.pendingNotice`.
  `run` calls `noticeArbiterSettled` after the step that carried the arbiter attempt came back neither busy nor refused, so a refused consume hands back with no outcome line.
  The `noticeArbiterSettled`, `arbitrate` and `goDriver` comments and the `sheddriver/doc.go` sentence say when it is sent.
- **F4** (commit `408a3f21`): `TestGoDriver_Settle` gains rows for an arbiter resume with no orch recorded, with an `OrchRecorded` error, with a failing `Notify`, at a blocked stop, and with the step refusing the attempt.
  The harness gains `orchRecordedErr` and `notifyErr` stubs and the `wantNotice` and `wantNoNotice` assertions over the lines `Notify` received.
  The success row asserts the exact line.
- **F3** (commit `256cb91d`): `bouncer-fragment-decision.md` (header and body), `shedadapters/doc.go`, `loomcli/circling.go` and `shed-recipe-spec.md` no longer call a decision finding the parent's alone.
- **F2, F5, F6** (commit `de5cb0d2`, all in `loom-template-arbiter.md`; one commit because the three edit one file):
  - F2: the `hard-cap` rule and the `circling` at-cap branch accept only when the finding is wrong or the fix in place answers it, record a decision there only to leave the ruling in the record, and hand on a fix still to be made.
  - F5: the `decision` rule reads the brief's Refusal section before choosing accept.
  - F6: the override of the brief's steps is a list.
    The self-check changed it, see below.
- **Self-check fixes** (commit `08bdc788`): see the Self-check section.

### Fixed differently

- **F6**: the reviewer's list named four brief rules and said every other line of the brief still binds.
  The proposal-lens showed that leaves the brief's `--by parent` recording and its resume step binding against the stencil's own `--by arbiter` recording and its ban on resuming.
  The sentence now scopes the override to the brief's "What to do" section: it lists settling only on an operator decision, never counting a `--by arbiter` entry, recording with `--by parent`, resuming the run, never accepting over an open design-wrong finding, and the report-back ending.
  It keeps the decide options, the materials list, the refusal text, the scope-widening ban and the `budget` rule binding.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `loom-template-discussion.md:107` keeps the "only the run's parent" framing: disputed, see below.
- fix-consistency, LOW, the `circling` bullet's "Otherwise hand on" read as covering only the at-cap sentence: fixed, now "In every other case hand on."
- fix-proposal, MEDIUM, the F6 override list is incomplete against the brief's recording and resume steps: fixed differently, see above.
- fix-proposal, LOW, the decision fragment promises an arbiter that may be switched off: fixed, now "Where an arbiter runs, it settles each finding the board entry and the decision record answer before the parent sees it. Put each remaining finding to the operator, unless a recorded operator decision answers it."
- fix-affected-set, LOW, `internal/shedrecipe/entries_bouncer.go:30` says the round escalates to the run's parent: fixed.
- fix-affected-set, LOW, `internal/shedadapters/bouncer.go:158` says the round escalates to the run's parent: fixed.

Every fork returned a report; their reports are in `round-1-fixer-report.lens-<lens>.md` beside this file.

## Disputed

- **fix-consistency LOW on `contracts/stencils/loom/loom-template-discussion.md:107`**, ground `premise false`.
  Step 4 of the Discussion template tells the Discussion writer that a question only the run's parent can settle, such as the entry outdated by landed work, is recorded under `## Open risks` and not settled from its own exploration.
  That is the writer's rule for its own questions during discussion, not the routing of a review's `decision` finding, so the arbiter-first routing does not make it false.

## Deferred

None.
