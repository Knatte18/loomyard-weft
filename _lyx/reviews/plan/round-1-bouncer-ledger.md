---
round: 1
ledger:
  - key: attach-turn-boundary-seeding
    rounds: [1]
    status: resolved
  - key: gate-final-unbounded
    rounds: [1]
    status: resolved
  - key: wait-min-zero-near-off-switch
    rounds: [1]
    status: resolved
  - key: pending-memo-not-cleared
    rounds: [1]
    status: resolved
  - key: closure-restart-cap-throttle-notify-tests
    rounds: [1]
    status: resolved
  - key: overview-doc-edits-incomplete
    rounds: [1]
    status: resolved
  - key: wire-card-pathspec-bound
    rounds: [1]
    status: resolved
  - key: approve-review-file-cwd
    rounds: [1]
    status: resolved
---

Round 1 raised eight findings against the plan, all fixed before judging, so every entry is resolved.

The weight sat on card 1's gate contract.
After a driver restart, the turn-boundary state now rests on the replayed Done that `AttachGated` reads, with the mid-turn residual named and bounded.
`GateEntry.Final` is now bounded to `PassOnCap` at the shuttle contract, and the pending memo is cleared so poll ticks and `finalize` read fresh state.
Card 2 dropped `parent_review_wait_min: 0` as a second off switch, and card 5 gained restart tests over a second closure.
The rest were doc and bound gaps in cards 7 and 12.

Watch in later rounds: card 1 also edits the `wait.go` file comment and, conditionally, `completionsignal_enforcement_test.go`'s pinned sites, though neither appears in its `Edit` list.
That is not a finding, since both are in the package card 1 already targets.
