# Fixer report — plan, round 2

Every finding's premise held against the code; F2 was carried out under the parent's ruling (see Rulings).

## Changes

- **F1** (fixed as proposed): `06-adopt-step-order.md` drops the reconcile warnings from `stepReconcile`.
  `stepOwner` now calls `reconcileWarnings` at its end, after the read-back when one runs, over the reconcile report, `Inputs.OwnerStood` and the merged per-slug outcomes.
  The warnings join the owner step's part, ahead of the deploy warning.
  A sentence says the steps up to `owner` share one process.
  Card 7's two-process merge row now says those warnings are part of the earlier `owner` step.
- **F3** (fixed as proposed, narrowed by the self-check): card 6 seeds outcomes from the `Subjects` without `root`, plus `Inputs.Runs` and `Inputs.Paused`.
  Card 7's `envelopeOf` collects over the `Subjects` without `root` that the merged parts hold, so a stop before any slug field lists no slugs, as today's reconcile refusal does.
- **F4** (fixed as proposed, heading worded differently): card 7 drops the three repeated `TestRunWorker` rows.
  Its `TestAdopt` edit adds the persisted-record assertions to card 6's held-back row, card 6's failed-deploy row and `ReconcileErrorRefuses`.
  The heading reads "one row per worker behavior the `TestAdopt` rows do not reach", not the proposed "one row per D2 criterion …", since the remaining rows also pin D1 behavior (self-check).
- **F5** (fixed, (b) narrowed by the self-check):
  (a) card 1 drops the `internal/beamcli#TestDobbyRuns_PairAtItsOwnAnchor` Edit target and its Tests bullet;
  (b) card 8's `TestRunAdopt` end rows assert the exit code and that the envelope or error reaches the writer.
  The `stopped` row still asserts the record's way forward, which `runAdopt` renders and no other test pins.
  (c) card 9's tier1 status test asserts only the section's state and step.
- **F6** (fixed as proposed): card 6's Intent names the comments reworded with their symbols' edits: the `adopt.go` header, the `resumeSlug` doc, the `runBeamSteps` and `heldBeamWayForward` docs, and the `deps.go` header.
  The wording on `heldBeamWayForward` follows step 6: `lym loom resume`, and `lym tender resume` first when adopt paused the beam.
- **F7** (fixed as proposed): card 3 adds the Edit target `internal/lymdirs#AdoptDirName`, a Uses section listing `internal/lymdirs/doc.go`, an Intent sentence on `AdoptDirName`'s comment, and lymdirs in the ImpactSummary.
- **F8** (fixed as proposed, widened by the self-check): the overview's Shared Decisions bullet scopes `Request.Command` to the refusals and way-forward texts that the record, the worker, the follow and the stop add.
  The existing `reentry` text stays wherever it ends the takeover's own refusals and warnings, the reconcile warnings included.

## Self-check

- fix-consistency, LOW, the `TestRunWorker` heading names only D2: fixed as proposed, merged with fix-proposal 4 (worded "per worker behavior").
- fix-consistency, LOW, the `heldBeamWayForward` doc wording names `lym tender resume` unconditionally: fixed as proposed, merged with fix-claims F3 and fix-proposal 3.
- fix-claims, MEDIUM, `envelopeOf` over `Subjects` meets a subject with no outcome on an early stop: fixed as proposed, merged with fix-proposal 1.
  `envelopeOf` collects only the subjects the merged parts hold.
- fix-claims, LOW, the `reentry` exception names only refusals: fixed as proposed (refusals and warnings, the reconcile warnings included).
- fix-claims, LOW, the doc wording for `heldBeamWayForward`: merged with fix-consistency 2 above.
- fix-proposal, MEDIUM, F5(b) loses the stopped row's way-forward assertion: fixed as proposed (the `stopped` row keeps it).
- fix-proposal, LOW, the doc wording for `heldBeamWayForward`: merged with fix-consistency 2 above.
- fix-proposal, LOW, the `TestRunWorker` heading: merged with fix-consistency 1 above.
- fix-proposal, LOW, what happens to `StoppedPart` when the standing-move check refuses: fixed as proposed (`runSteps`' refusal clears `StoppedPart`, matching today's empty outcome on that refusal).
- fix-proposal, MEDIUM, `envelopeOf` with a subject that has no outcome: merged with fix-claims F1 above.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

- **F2** (decision): the parent's decision, recorded under "Added after Discussion (parent, 2026-11-09): A failed step keeps its gathered part in the end envelope", asked for:
  `runSteps` stores a failed step's gathered part in the record (`StoppedPart`), cleared when that step later succeeds;
  `envelopeOf` merges it after the done steps without listing the step as done;
  card 7's refused-reconcile outcome asserts the envelope keeps the reconcile report and the mutations with `partial` true;
  card 6's failed-deploy row keeps asserting the deploy outcome is named.
  Changes:
  - Card 3's `Record` gains `StoppedPart` (`json.RawMessage`).
  - Card 6's `runSteps` stores and merges the part, clears it when a later run of the step ends without error, and replaces it on a later stop.
  - Card 7's `envelopeOf` merges it after the done steps.
  - The overview's Shared Decisions bullet names it.
  - Card 7's `ReconcileErrorRefuses` row asserts the report, the mutations and `partial` true, with the faked reconcile returning mutations beside its error.
  - Card 6's failed-deploy row asserts the deploy outcome is named.
  Carried out in full; not flagged to the parent.
