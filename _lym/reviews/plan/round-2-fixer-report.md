---
dispositions:
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
---

# Round 2 fixer-report

This session resumed a halted fix. The target already held the earlier session's changes (report set aside as `round-2-fixer-report-20261127T080451Z.md`).
I checked each against the review and the code, kept those that hold, and corrected the one estimate that did not.

## Changes

- **F15** (fixed):
  - Kept from the earlier session:
    - `00-overview.md`, decision "the foundation's tests shrink": the halt-and-ask clause is cut, and card 15's commit body still reports the measured totals.
    - `15-snapshot-reader.md` names the further `internal/fabricengine` deletions, whole tests and parts of tests, each with the test that holds its assertion or why it guards nothing of its own.
  - Corrected this session: the earlier text put card 15's further deletions at about 820 lines.
    The spans measured at HEAD come to about 700: about 280 for whole tests and helpers, about 330 for subtests, about 90 for tails and loops.
    The overview now says about 700, and that the part ends about 180 lines down.
  - Added from the self-check: the overview makes the estimate of about 1,050 added lines a budget the cards' new tests stay within, so the margin is a constraint, not only a forecast.
- **F16** (fixed): the earlier session's changes kept, and the multi-store error widened from the self-check.
  - Card 15 walks the stores in the fixed order `StoreCode`, `StoreRecords`, `StoreBolt`, skipping stores the reader does not hold. It names each other field's `Store`: `FieldWorktrees` `StoreCode`, `FieldAnchors` and `FieldRecordsStore` `StoreRecords`, `FieldPendingTeardowns` the zero `Store`.
  - Card 14: a phase enters `Phases` only after its read succeeds. A re-read's `ReadFailure` records the phase the re-read ran under. `TestRunRule`'s transport row asserts `Phases`.
  - Beyond the proposal: a failing store does not stop the walk. `read` returns one `*fieldReadError` per failing store, joined with `errors.Join` in walk order.
    Card 14's `runRule` records one `ReadFailure` per joined error when each is a transport failure, and `TestRunRule`'s transport row fails two stores and wants two read failures in order.
- Items from the review's "Seen but not admissible" list, inside this surface:
  - Kept from the earlier session: card 15's `FieldLocalRefs` rows read from the linked worktree's root. Card 2's `TestStructuralDirs` overwrites element 0. Card 3 asserts `HubReservedNames()` against its literal members. Card 13's `TestBuilder` overwrites the built slices.
  - Added this session, card 6: a recorder method called on a record value becomes `fabriclayout.MutationsOf` over the entry lists.
    The sites are `ReconcileOrigin`, `reconcileEnded`, `pairteardown`'s `snapshotOutcome.applyTo`, `fabriccli`'s `runCleanupWithFlags` and `orchcli`'s `TestStart_ReconcilesTheHubFirst`, which joins card 6's Edit list.
    Without it card 6 does not build: the layout's `Mutations` has no `Append`, `Extend` or `Snapshot`.
  - Not taken: card 4's seed and sync subject rows of `TestSeedCommitSignature`.
- `lym loom validate-plan` passes.

## Self-check

- fix-consistency, LOW, card 6's ImpactSummary says every call site compiles unchanged: fixed as proposed.
- fix-affected-set, MEDIUM, card 6's bullet misses `fabriccli/fabric.go:894-899`'s `rec.Extend`: fixed as proposed.
- fix-affected-set, MEDIUM, `orchcli/start_test.go:1186` calls `Append` on a record value, outside card 6's Edit set: fixed as proposed. The bullet covers any recorder method, and the test joins card 6's Edit list.
- fix-proposal, MEDIUM, the margin of about 180 lines rests on estimates and nothing acts on an overrun: fixed with my own change. The overview makes 1,050 the added tests' budget.
  I named no reserve deletions this round.
- fix-proposal, MEDIUM, returning only the first failing store's error hides a second unreachable store: fixed as proposed, with `errors.Join` in card 15 and one `ReadFailure` per joined error in card 14.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
