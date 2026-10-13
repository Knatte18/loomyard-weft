# Arbiter re-launch, scope-guard deviations, alignment landing order

Three rough edges around the arbiter and the alignment agent.

## What changes

- A `resume` re-launch whose arbiter verdict is already in effect is admitted instead of refused as "the run moved since the arbiter spawned".
  The tackler's consume takes the same admit.
- The arbiter verb refuses a `blocked` Merge-In stop whose verify-failure record names a target, as it already did for Publish, so the arbiter no longer races batten's verify retry.
- The arbiter and alignment stencils name `lyx board get '{"slug":"…"}'`, the JSON payload the verb takes, instead of a bare slug.
- The Webster deviation list comes from the scope guard alone.
  The implementer fork reports `status` and `head_sha` only.
  `record-batch` stores the guard's deduplicated, sorted file list in the batch digest's `deviations` field, and the Master stencil reads it there.
- The alignment agent settles cross-run landing order itself and escalates only on a design conflict or an open question.
  `lyx loom peers` gains `row` per peer and `own_row` for the caller, documented in `internal/loomcli/doc.go`.

No issue is linked to this task, so there is no `Closes` line.

## Design decisions

- **Verdict in effect.**
  The admit holds when the verdict names a goto target, the run's producer is that target, its state is `paused`, and every history entry after the attempt's recorded length is a `goto` whose last entry names the target.
  The re-launch then performs no second goto.
  `shedengine.Relaunch` learns this from its admit hook through a generic signal; the verdict logic stays in `loomcli`.
  On this admit only, the pause check refuses a pause condition present now and absent at spawn; an unmoved run keeps today's equality check.
  A parent's goto to the same row is admitted too, and the re-launch steps it without waiting for the parent's resume.
- **Mover.**
  The second actor is batten's first-verify-failure retry.
  A Merge-In or Publish halt is ineligible only when the failure record belongs to the halt: its row is the current producer, its head is the pair's HEAD, and it was written during the halting attempt.
  This narrows the Publish refusal: a rejected push or a tree dirtied after the verify, with a stale record naming a target, is now eligible.
  Without batten, or with its retry spent, a Merge-In verify halt of this shape parks for the parent, as Publish does.
  The incidents' attempt records are gone, so this rests on the code path and the incidents' shape.
- **Scope guard.**
  Findings carry their file as a structured field.
  The guard also reports a changed file the delta gives no symbols for, unless a card target covers it: a file target by equality, a package or symbol target for files directly in its package directory, an external test package through planparser's one external-test unit mapping.
  The scope findings no longer ride record-batch's warnings.
  The reader surface drops to file granularity, and the list stays informational.
  `ParseReport` ignores a `deviations` key from an older fork.
- **Landing order.**
  The row order comes from the recipe embedded in the running binary through `lyx loom peers`, so a hub with no loomyard checkout has the same source and the stencil restates no rows.
  Darn peers are read against the loom recipe: a shared row name takes loom's position, a darn-only row reads 0.
  Ties and unknown orders are reported with the verdict unchanged.
  A landed parent commit is compared by subject and diff against this run's decision record.
  The rule applies at `parent-review` and `pr-gate`.
  A text-only overlap that interacts in behaviour is caught only by verify at the second run's Merge-In, and two runs changing the same PATTERN entry the same way no longer escalate.

## To check by hand

- The step-after-goto admit condition in `internal/loomcli/arbiterconsume.go`: a review flagged that the "step run after the goto" case may not be fully covered by the history-entry conjunct.
  Check that a run moved by a goto and then stepped is refused as moved.
- The `webster/summary.md` run record lists scope warnings for `haltAtVerifyRow`, `verdictInEffect`, `loomRowNumber`, `filesWithSymbols` and `targetsCover` as touched outside the batch's targets; confirm they are helpers of the changed code.
- The unit tests and the integration suite were not run for this description.
