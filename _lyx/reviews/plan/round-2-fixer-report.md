# Fixer report — plan, round 2

## Changes

- **F1** (BLOCKING, card 7), fixed as proposed.
  Dropped the Create line `plan:internal/creelcli#TestWorktreeTornDown` and the Tests bullet built on the false prunable premise.
  Added a `TestHubDelivery_SkipsAWorktreeThatNoLongerResolves` row for a gitfile whose `gitdir:` names a missing directory: `Strands` skips the pair and logs at Info.
  Added a bullet saying `worktreeTornDown` gets no test of its own, and which rows reach each of its marks and false answers.
- **F2** (BLOCKING, card 5), fixed as proposed, with the message reworded by the self-check (see below).
  `strandsElsewhereLocked`'s failed lookup now goes through `e.calleeFailures.Fail("pane placements", …)`, and a successful lookup calls `e.calleeFailures.Recover("pane placements", …)`, so the lookup logs Warn once per episode, Debug while it repeats and Info at recovery.
  The failing-lookup row of `TestStatus_ReportsOwnWindowStrandsFromTheSessionListing` now asserts that a second failing `Status` logs no second Warn.
- **F3** (BLOCKING, card 7), fixed as proposed, with the forget rule widened by the self-check (see below).
  Added `internal/creelcli#hubDelivery` to the Edit list, along with the rule that `hubDelivery` remembers skipped paths under a mutex.
  The first skip of a path logs at the decided level, and a repeated skip logs at Debug.
  The empty-`.git`-directory row now asserts that a second `Strands` call logs at Debug and not at Warn.
  The `doc.go` sentence also names the drop to Debug.
- **F4** (LOW, card 8), fixed as proposed.
  The ImpactSummary now says "every production caller of `ClearCycleRequest` is on this card".

`lyx loom validate-plan` reports `ok` after every change.

## Self-check

- fix-claims, LOW: card 5 widens `calleeFailures` to `Status` and reconcile, but the field's doc at `internal/reedengine/lock.go:47` names only the watch tick and the name-repair loop.
  Fixed as proposed: added `internal/reedengine#Engine` to card 5's Edit list and a sentence widening that doc.
- fix-proposal, LOW: card 7 forgot a remembered path only when it resolved again, so a later worktree at a pruned path would log its first skip at Debug.
  Fixed as proposed: a path the listing no longer holds is forgotten too.
- fix-proposal, LOW: card 5's Warn message said "clearing the bindings the session lacks", but `Engine.Status` reaches the same lookup and clears nothing.
  Fixed as proposed: the message now reads "reading every pane the session lacks as found nowhere".
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
