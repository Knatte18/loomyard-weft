# Fixer report — discussion, round 4

## Changes

- **F1** (BLOCKING), fixed as proposed, option (a).
  D9 gains a paragraph: `Fabric.Pull`'s warp arm reads its unpushed test after its own fetch through the tracking ref's reflog, as D4's rewound test does, so a rewritten branch the reconcile already fetched takes the anchor walk and reset instead of `ErrWarpDivergedUnpushed`.
  It also refuses a rewound task branch before the reset, naming `ResetPairCode`'s way forward, so Q28 and the Acceptance criterion on a rewind fetched by `lyx fabric pull` still hold; the prime's warp branch keeps today's reset.
  D9's Bound, D4's rewritten bullet and the Scope bullet on `lyx fabric pull` now agree with it.
  New Acceptance criterion: `lyx fabric pull` after a moving reconcile recovers a rewritten pair, refuses a rewound task branch, and refuses a local commit no reflog value reaches.
- **F2** (MEDIUM), fixed as proposed.
  D6 item 1: the hub workspace regeneration takes one try-acquire of the hub workspace lock and is skipped with its existing warning when held; the local mode skips the store housekeeping's foreground gc; the two `runLockWait` waits are the only waits, summed over a hub's ended pairs.
  D6's snapshot sentence: it builds commits through a temporary index and holds the weft write lock by its own try-acquire, reporting busy when held.
  D4's Bound and D6's Bound list the same waits. New Acceptance criterion for a held hub workspace lock and no gc.
- **F3** (MEDIUM), fixed as proposed (first option).
  D8 wake: the wake reconcile retries the ls-remote pair every 10 seconds within 2 minutes, each attempt under a 20-second deadline, all injectable; still unreachable, it moves nothing, the owned entry is lifted, and the notice names `unreachable` and `lyx cleanup --apply`.
  The Auto-mode assumptions bound what a resumed driver can do in that window. New Acceptance criterion for wake with an unreachable origin.
- **F4** (LOW), fixed as proposed.
  The rewound/rewritten Acceptance bullet is split; the rewound bullet pins precedence over ahead before and after the fetch; a new bullet pins the diverged boundary case.
- Support log: ledger entries Q30–Q32 for the round's picks; Q28 marked as widened by Q30.

## Self-check

- fix-consistency, MEDIUM: the wake retry contradicted "exactly two ls-remote calls" in D1, D8's wake Bound and the Acceptance criterion. Fixed: D1 and the wake Bound name the retry, the criterion is scoped to a reachable origin.
- fix-consistency, LOW: sibling wake notices named plain `lyx cleanup`, which only judges. Fixed: both now name `lyx cleanup --apply`.
- fix-consistency, LOW: Q28 stale next to Q30. Fixed: Q28 marked as widened by Q30.
- fix-way-forward, MEDIUM: a snapshot refused busy after the pause lift left no marker for a re-run. Fixed: the pair pause entry is lifted only once the snapshot tag is written.
- fix-way-forward, MEDIUM: `lyx orch start` does not reconcile against a live orch strand. Fixed: the unreachable-wake notice names only `lyx cleanup --apply`.
- fix-way-forward, LOW: that reconcile reports resumed pairs live and does not move them. Fixed: the notice says so and names their run-scope reconcile at Merge-In or Finalize.
- fix-proposal, MEDIUM: one ls-remote attempt can run up to `remoteDeadline` (10 minutes), past the 2-minute bound. Fixed: each attempt gets a 20-second deadline.
- fix-proposal, MEDIUM: the rewound refusal reached the prime and Land's pull, where `ResetPairCode` does not apply. Fixed: the refusal covers a pair's task branch only; the prime keeps today's reset.
- fix-claims, MEDIUM: "pushes only fast-forwards" ignored `ResetPairCode`'s leased update. Fixed: the sentence names the lease and what it refuses.
- fix-claims, LOW: an ordinary Remove does not always run gc. Fixed: reworded to leave the gc to the next ordinary Remove's housekeeping.

## Disputed

None.

## Deferred

None.
