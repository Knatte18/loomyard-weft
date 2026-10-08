# Webster state across replans: goto to Plan, late plan reviews, rebaseline clarity

`webster-replan-state` · Task · bug, enhancement, webster, loom, burler

Depends on: [`gate-test-scope`](design-gate-test-scope.md)

## Goal

- A `lyx loom goto` back to Plan after Webster started leaves webster state consistent, so Webster never hard-fails on a plan fingerprint mismatch (#466).
- A Plan-Review round after the plan is implemented is not dispatched as a fix round that edits the plan webster recorded (#465).
- Rebaseline logs and names the silent overview-frame fallback when the baseline copy has no parseable Card Index.
- `commitsNamedBy` classifies every `head_sha` prefix candidate in one `git cat-file --batch-check` call.
- `verify-nested-module` (`crossModuleCommands` in `internal/planparser/verifymodule.go`) splits a verify chain at `||` and `|` as well.

The merged notes keep their own Seen sections below.

## From `goto-plan-stale-webster`

Imported from [issue #466](https://github.com/Knatte18/loomyard/issues/466).

## What happened

A loom run completed the Plan, Batchifier and Webster rows, then the Publish row went stuck and the run routed back (`goto`) to `Plan-Bouncer` several times.
The Plan loop edited the plan, and `Batchifier` re-ran and finished (`done`).
The next `Webster` step then halted the run as `failed` with anomaly `producer-hard-failure`:

```
webster: on-disk plan fingerprint does not match this run's recorded state: on-disk plan fingerprint e5774b7e... does not match this run's recorded fingerprint c14834a3...; the plan changed since state.json was created; way forward: 00-overview.md changed and is never rebaselined; restore it with "lyx webster restore-plan", or 1) lyx webster reset --to start; 2) re-step the Webster row
```

The run had 49 history entries when it halted.

## Why this is abnormal

The loom's own routing sent the run back through Plan after Webster had already created `state.json`, and then re-ran Webster against that stale state.
The recorded fingerprint can only match if the plan is untouched, so a legitimate Plan-loop edit made the next Webster step fail by construction.
Neither the `goto` to Plan nor the re-run `Batchifier` row reset or rebaselined webster state, so the failure was foreseeable from the recipe's own wiring rather than an operator or task-code error.
The error text offers manual recovery (`lyx webster restore-plan`, or `lyx webster reset --to start` and re-step), which a driver could have applied itself.

## What to do

When the loom routes back to a row upstream of Webster after Webster state exists, reset webster state (equivalent to `lyx webster reset --to start`) as part of that route, or have the Webster row detect a changed plan produced by an upstream re-run and rebaseline instead of failing.
Alternatively, make the recipe validate at the `goto` that a Plan re-entry invalidates downstream webster state.

Provenance: task slug `shuttle-send`; notes: `loom-halt.md`.

## From `plan-review-after-implemented`

Imported from [issue #465](https://github.com/Knatte18/loomyard/issues/465).

A Plan-Review round ran over a plan that was already approved and implemented by webster.
The review gained findings, but the fixer's only write surface was the plan, and webster could not accept a new card on this run's record.
`lyx webster rebaseline --card=[11,12]` was refused:

```
webster: the edited plan changes the cards of a batch this run already begun: 00-overview.md changed and this run recorded no overview frame, so the change cannot be confined to its Card Index; it carries the plan's integration verify and is never rebaselined; way forward: restore 00-overview.md, or 1) lyx webster reset --to start; 2) lyx webster run
```

The parent stopped all fixes mid-round, so every finding ended deferred.
The fixer also spent its orientation designing a new card that was then thrown away.

Why this is abnormal: the loom dispatches a review round whose fixes cannot land.
A frozen plan (webster already begun) makes the fix round unwinnable, and the only advertised way forward (`lyx webster reset --to start`) discards the implemented work.
The run records no overview frame, so a legitimate edit to the overview's Card Index cannot be confined and is refused outright.

What to do:
- Skip the fix-round dispatch, or tell the round prompt up front that the plan is frozen, when the run's webster record has begun a batch.
- Record an overview frame when webster begins a batch, so a Card Index change can be confined and rebaselined.

Provenance: task `shuttle-send`; notes `burler-plan-r1.md`, `webster-refusal-rebaseline.md`.

## From `overview-frame-fallback-log`

## Seen

The PR-Gate review of `webster-record-batch` (PR #473) found that `recordedOverviewFrame` in `internal/websterengine` returns an empty frame without logging when the stored baseline copy has no parseable Card Index, so the reason a rebaseline refuses on an old record is invisible.

## Idea

Log the fallback at Info with the baseline path, and name it in the rebaseline refusal.

## From `head-sha-batch-check`

## Seen

The PR-Gate review of `webster-record-batch` (PR #473) found that `commitsNamedBy` in `internal/websterengine/gitwrap.go` runs one `git cat-file -t` per object a short `head_sha` prefix names, and a 4-character prefix in a large repository can name many.

## Idea

Classify every candidate in one `git cat-file --batch-check` call.

## From `verify-module-or-pipe`

## Seen

The PR-Gate review of `plan-gate-checks` (PR #475) found that `crossModuleCommands` in `internal/planparser/verifymodule.go` splits a verify chain only at `&&`, `;` and newlines, so a `go` command after `||` or a pipe is read as part of the previous segment and its module is never checked.

## Idea

Split at `||` and `|` too.
