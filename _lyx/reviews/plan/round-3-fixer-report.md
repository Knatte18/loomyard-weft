# Fixer report — plan, round 3

## Fixed

### F1 (BLOCKING, scope) — `lyx fabric shortname` lost the push-path drop from its envelope

Confirmed against the code: `runShortname` in `internal/fabriccli/shortname.go` builds `rec := fabricengine.NewMutations(l.HubPath)` and pushes through `Bolt.Push` before returning `okWithRecord`/`errWithRecord` with that record.
The same gap holds for `CloneAndWire` in `internal/fabriccli/clone.go`, which holds a `rec` for `lyx fabric clone`'s envelope and pushes through `b.Push`; it is fixed with the same rule so the method, not the one instance, is corrected.

Card 13:

- `Edit:` gains `internal/fabriccli#runShortname` and `internal/fabriccli#CloneAndWire`.
- `Intent:` drops `shortname` from the recordless callers and states the rule: every caller that holds a mutation record for its envelope switches its Bolt push to `PushRecorded` with that record, naming `runReconcile`'s two pushes, `runShortname`'s and `CloneAndWire`'s; the "records nothing" comments at those pushes gain that a drop is recorded.
  `doc.go` states the rule rather than enumerating pushes.
- `ImpactSummary:` names the three verbs whose `mutations` may now carry `commits_dropped`.

`00-overview.md`'s Card Index line for card 13 now says the drop is recorded in the envelope of each fabric verb that holds a record, not only `lyx fabric reconcile`'s.

### F2 (LOW, consistency) — the pull-first drop of a recordless caller was visible nowhere

- Card 7: `pullFirst` logs a drop at Info with the dropped SHAs beside its `KindCommitsDropped` entry, so a caller whose record reaches no envelope still sees the reset.
- Card 8: the pre-run's `PullThenCommitWritten` call passes a local `NewMutations(hub)`; `internal/fabricengine#NewMutations` joins `Uses:`.
- Card 10: `reconcileHubWide` passes a local `NewMutations` rooted at the board dir's parent; `internal/fabricengine#NewMutations` joins `Uses:`.

No test is added: neither finding is a coverage gap, and card 13's `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict` already pins the recording behavior of `PushRecorded` the new callers reach.

`lyx loom validate-plan` reports `ok: true`, with only the informational `create-new-unit` rows.

## Disputed

None.

## Deferred

None.
