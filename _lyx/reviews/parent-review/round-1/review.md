---
verdict: APPROVED
---

## Scope check against board entry `darn-verify-fixes`

The design covers each open item of the entry's 2026-10-22 re-scope and adds no work another entry owns.

- Item 1, `TwoWorktreesBootedConcurrently` (#714): closed with no change, on `98b171b05` (2026-10-18, an ancestor of this branch), which moved the capability probe's `list-commands` to a private `-L` socket and whose message names this exact test and failure as its cause.
  The entry asks to make the test deterministic "or retry or isolate the tmux server start it races on"; that commit isolates it, so the goal point is met by landed work, not left out.
  The record states openly that this departs from the re-scope's open list (Scope, Q10), so the operator can overrule it.
- Item 2: the record finds that a darn session can run the tier through `lyx gate test --tier 2`, which is the entry's "nothing is left here" case.
  D6's one pinning row in the shell guard's allow table is a test of that finding, not new behaviour.
- Item 3: D3 reroutes Darn-Write to Merge-In when the parent has moved since this run's failure, so the parent is merged before the verify re-runs, under `PATTERN-parent-lock`.
- Item 4: D1 and D2 tell an owned failure from a not-owned one through the impacted set and give each its own target and way forward.
- Item 5: D4 keeps an exhausted gate's findings file under the shed run directory until the row's next passing call.
- Item 6: D5 finds the cause, the blanket `lyxbin` join on any `cmd/lyx` dependency, and narrows it to a change to the `cmd/lyx` main package.
- Out of scope as the entry says: the work the re-scope lists as landed, and the loom recipe's routing, which the entry does not name.

No decision contradicts another board entry.
D3 reads tips with no fetch, which follows `fabric-remote-async`.
`gate-test-repeats` is about which tiers repeat, not the impacted set's selection, so D5 does not duplicate it.

## Overlap with runs in flight

`lyx loom peers` lists no landed parent commits; `own_row` is 3.

- `machine-handover` (row 3, tie): its D7 adds an owner check to the landing producer's Finalize, and this run's D1 and D2 change the verify failure target and way forward at Merge-In and Publish in `internal/landingshed`.
  Different behaviour in the same package and its docs; no decision conflicts and no shared PATTERN entry changes.
  Landing order: a tie, both at row 3.
  The record's Notes for the plan writer already expect merge conflicts there.
- `quarry-csharp` (row 3, tie): changes `internal/planparser` and `internal/planglyph`; this run uses `internal/planglyph` only as the example diff in D5's acceptance criterion and changes neither package.
  No overlap.
- `decision-add-notifies`, `research-in-discussion`, `shuttle-open-agent-defaults` (row 0, unknown order): no decision record and no changed paths yet; their board entries concern `lyx loom decision add` notices, Discussion stencils, and `lyx reed add`/`shuttle open` defaults.
  None changes verify routing, Darn-Write's entry, gate findings or the impacted set.
  No overlap.
