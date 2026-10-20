# Round 2 fixer report — plan

Every finding's premise was checked against the code at HEAD and held.
`lyx loom validate-plan` was clean before the round and is clean after it.

## Changes

- F1 (fixed as proposed): card 1 gives `FailedPush` a `Prime` flag and `codeDivergedWayForward` a `prime bool` parameter (re-signed glyph).
  A `Prime` code record's `deferred`, `transport`, `auth` and `not_fast_forward` lines name `git merge origin/<branch>` and `git push` in the prime, and never `lyx fabric push`.
  `TestFailedPushRecord` gains prime rows for `transport` and `not_fast_forward`.
  Card 15's diverged bullet passes `prime` for the prime's warp branch, and `TestReconcileOrigin_Judge` gains a diverged-prime row.
- F2 (fixed, partly differently): card 15 adds `BranchOutcome.WayForward` and lists one line per stopping state: rewound, rewritten, diverged, gone, dirty, busy, live and the D3 stop.
  `Summary` and card 16's moves fill it.
  Cards 20, 21, 22 and 23 take their line from it.
  `ReconcileOrigin` fills it after `judgeBranch` returns, not `judgeBranch` itself, because `judgeBranch`'s inputs carry neither the slug nor the prime flag.
  The live line differs from the proposed `lyx batten pause <slug>` then `lyx cleanup --apply`.
  `lyx batten pause` takes effect only at batten's next producer boundary, which for a task pair comes after the whole loom run (`internal/battencli/cli.go:194`).
  Until then `probeQuiet` keeps reading the pair as live (`internal/pairteardown/teardown.go`).
  The line now names the run's own Merge-In reconcile (card 22), and `lyx cleanup --apply` once the run has ended.
  Rows: gone in `TestReconcileOrigin_Judge`; dirty, live, busy write lock, the Bolt dirty D3 stop and the prime-weft conflict in `TestReconcileOrigin_Moves`.
- F3 (fixed as proposed): card 9 names the predicate.
  A code-side push counts as a task branch's exactly when `path` is not the prime's worktree, the prime resolved as `IsPrimeWorktree` resolves it.
  The same predicate sets `FailedPush.Prime`, and `internal/fabricengine#PrimeName` is added to Uses.
  Added row: the prime's code branch pushed after an origin rewind is pushed and records nothing.
- F4 (fixed as proposed): card 8 creates `gitrepo#PushBackoffDelays`, derived from `pushBaseBackoff` and `pushMaxAttempts`.
  Its persistent-refusal row in `TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound` asserts the delays.
  Card 9 uses it for the gone-path backoff.
- F5 (fixed as proposed): one rule.
  A transport failure of the gone-path ls-remote or the Bolt fetch is retried by the same backoff and records `transport` only once the backoff ends.
  The push call's own transport failure stays `transport` at once.
  The unreachable Bolt-fetch row is replaced by an ls-remote row with a restoring sleep and one with a non-restoring sleep.
- F6 (fixed as proposed): card 17 calls `Teardown` only for an entry that still has a local part of its pair.
  A local part is the worktrees, the branches or the launchers, the remnants `refuseWhenNothingRemains` counts.
  An entry with none is reported ended with no seam call.
  Card 13 bounds `removeSibling`'s remote branch in local mode, and limits the no-ls-remote claim to a pair with a local part.
  Rows were added to `TestReconcileOrigin_Ended` and `TestRemoveLocal`.
- F7 (fixed as proposed): card 7 creates `battencli#pullStopOutcome`, a direct type assertion, and `TestPullStopOutcome`, with rows for a lone stop, a joined stop and an unrelated error.
- F8 (fixed as proposed): card 9 specifies that `PushFailedError.Error()` ends in `; way forward: <line>`.
  `loomcli#pushFailureWayForward` returns `FailedPushWayForward` for a `*PushFailedError`, and `TestCommitRecordsVerb_ErrorEnvelopes` gains `gone`, `rewound` and `auth` rows.
  A `Fabric.PushBranch` rewound row asserts that the error text ends in the rewound line.
- F9 (fixed as proposed): card 20's scenario test gains the step with an origin URL that names a path holding no repository: `--apply` refuses and runs no segment, while the dry run prints the error and still runs the segments.
- F10 (fixed differently in form): `TeardownReport` embeds `MutationRecord`, following PATTERN-mutation-record's rule that every mutating result embeds it.
  It does not take the proposed field named `Mutations` of type `MutationRecord`, which would read `report.Mutations.Mutations`.
  Card 18's seam fills it from `Removal`'s record, and card 17 extends the report's record with it.
  Card 13's `SnapshotUncommitted` takes `rec *Mutations` (re-signed) and records the commit and tag.
  Card 14 extends the removal result's record with the snapshot.
  Assertions were added to `TestReconcileOrigin_Ended`, `TestRemoveLocal` and `TestTeardown_OriginSeams`.
- F11 (fixed as proposed): card 16 calls `finalizeEntered` once per candidate right after `readEndedSet`, before any move, and passes the filtered set to every replay and to `reconcileEnded`.
  It orders the prime weft's replay before the prime's warp move.
  Card 17 states that it reads the filtered set.
  The hub-scope row lives in `TestReconcileOrigin_Ended`.

## Self-check

- fix-consistency, MEDIUM: `judgeBranch` lacks the inputs for the gone and prime lines. Fixed as proposed: `ReconcileOrigin` fills `WayForward` after `judgeBranch`.
- fix-consistency, MEDIUM: card 13's no-ls-remote claim still false for a pair with nothing local. Fixed as proposed: the claim is bounded to a pair with a local part.
- fix-consistency, LOW: card 17's Uses still listed `finalizeEntered`. Fixed as proposed: removed.
- fix-consistency, LOW: card 15's gone line said "done" where card 1 says "done or removed". Fixed as proposed.
- fix-consistency, LOW (merged with fix-affected-set LOW and fix-proposal LOW): the way forward printed twice by commit-records and `writeHubRules`. Fixed as proposed: those callers print the `Unwrap`ped error, and the commit-records rows assert the line appears once.
- fix-claims, MEDIUM: the claim that every replay's own drop reads an unmoved parent was overstated for pair weft replays. Fixed as proposed: the sentence now covers Bolt and the prime weft only, and says a pair replay's drop only removes slugs from the filtered set.
- fix-claims, LOW: the local-part definition left out launchers. Fixed as proposed.
- fix-proposal, MEDIUM: the live line's `lyx batten pause` does not quiet a pair in time. Fixed as proposed: the line names the run's Merge-In reconcile.
- fix-proposal, MEDIUM: Finalize's trailing `push it by hand` contradicts a `rewound` hold-back. Fixed as proposed: card 9 edits `Finalize.pushParent` to drop the phrase for a `*PushFailedError`, with a row in `TestFinalize_MergeMarkPushOrderAndVerdict`.
- fix-proposal, LOW: a no-local-part ended entry recurs in every notice. Fixed as proposed: `Summary` and the wake notice omit it, and the report keeps it.

## Disputed

None.

## Deferred

None.
