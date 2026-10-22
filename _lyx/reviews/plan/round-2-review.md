---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/04-board-owner-writes.md:55
    summary: TestSetOwners names a held push lock as the input for the deferred-push refusal, but no such input reaches that branch, and the replay-loss re-read has no reaching input either
    fix: >-
      In card 4, give `OwnerRequest` one more field, a hook run once between the setter's commit and its first push, nil in production
      (state it in the same sentence as `Sleep`, and say it is the test's only way to race a second clone, since `PullThenCommitWritten` pulls immediately before the commit).
      Then rewrite the two `TestSetOwners` rows over that hook.
      Replay-loss row: the hook pushes a different `owner_machine` for the slug from a second clone, so the first push is rejected, the retry replays, origin's value stands, and `SetOwners` reports the slug in `Lost` with origin's owner from its post-push re-read.
      Deferred-push row: the hook pushes a conflicting commit from a second clone and takes the board write lock with `lock.TryAcquireWriteLock` on `fabricengine.BoardWriteLockFile`; the first push is rejected and `retryBoltPush` reports a deferral because that try-lock fails.
      A `Sleep` stub releases the lock on its first call for the retried-and-lands case, and keeps it for the still-ahead case, which refuses naming `lyx board sync`.
      Delete the words "a push lock held across the retry" from the row, since holding the push lock reaches nothing (see the issue).
      The change adds no target, so no `validate-plan --overlay` run is needed.
---

### [BLOCKING] TestSetOwners cannot reach the push-deferral refusal or the replay-loss re-read as written

**Location:** `_lyx/plan/04-board-owner-writes.md:48` and `:55`

**Issue:**
Card 4 prescribes that a push deferred by a held lock is retried once after `Sleep`, and that a branch still ahead of its upstream then refuses naming `lyx board sync`.
Its `TestSetOwners` row for that refusal is "a push lock held across the retry refusing naming `lyx board sync`, with a no-op `Sleep`".
That input does not reach the branch, checked against the code the card builds on:

- `SetOwners` pushes through `Bolt.PushRecorded`, which calls `pushClassified` (`internal/fabricengine/pushclassified.go`).
  Its first push is `PushRebaseFree`, which is lock-free, so a held push lock does not affect it.
- A deferral is reported only by `retryBoltPush` (`pushclassified.go:542`), and only after the first push was rejected.
  `retryBoltPush` try-acquires the board *write* lock and reports a deferral when that is held, or when `ErrReplayDirty` fires.
  It then takes the push lock with the blocking `lock.AcquireWriteLock`, so a test that holds the push lock across a rejected push hangs instead of refusing.
- A test cannot hold the board write lock across the call either, because `SetOwners` first calls `PullThenCommitWritten`, which takes that lock with the blocking `lock.AcquireWriteLock`.
- The first push is rejected only if origin moved between the setter's pull and its push, and `PullThenCommitWritten` pulls immediately before the commit.
  Nothing in the card's inputs makes that happen.

The same gap hides the post-push re-read that D3 and D5 step 3 require.
The row "a second clone's competing write losing to origin's, reported in `Lost` with origin's owner" passes through the pre-write check (a second clone's push is pulled first, the slug reads as owned elsewhere, and it is skipped without `Takeover`).
The replay-keeps-origin's-value path and the re-read that reports it are never run.
That path is the one the callers' `Lost` refusals in cards 9 and 24 rest on, and those cards test it only against a fake `Set`.

The `Sleep` field already follows `PATTERN-test-speed`, so the repair is an input that reaches the branch, not a change to the retry.
This is not a re-raise of round 1's F3: that finding was about the row waiting a real second, which the fixer's `Sleep` seam closed, and the fixer report disputes nothing.
The round-1 fixes I checked in the cards (F1, F2, F4 to F11) are present and have not regressed.

### Rejected

- The fast-forward-only rule is not applied to `retryWeftPush` (card 6's Bound).
  The guard would need an owner read of origin's Bolt outside `ReconcileOrigin`, and the Constraint allows that read only in the reconcile, or from the local Bolt.
  The local Bolt on the old machine does not yet show the adoption, so a local read cannot guard it.
  The card states the gap openly, so I raised no finding against the plan.
- Card 24 rewords `ErrBranchExists` instead of passing on fabric's message, which D5 step 2 names as the stop text.
  `ErrBranchExists` is a typed error whose doc invites a caller to word its own refusal, and fabric's way forward (`git branch -D`) drops unpushed commits with no archive, so the reword is a safe departure.
- Card 25's short `PATTERN.md` line keeps the exception ("bar a Finalize past its parent-side merge") and the `ReconcileOrigin` read source.
  It drops the local-Bolt reads and the unowned-entry reading.
  The line is 357 bytes against the 360-character cap, so adding them does not fit, and the Added-after-Discussion decision puts the full declared text verbatim in `pattern/PATTERN-owner-machine.md`, which the card does.
- Card 25 leaves the choice of which other `PATTERN.md` lines to trim to the implementer, with the cuts listed in the commit body.
  I judged this workable and raised no finding, since the card states the constraint (no clause, package, symbol, operand, exception or `(test)` marker removed) and `lyx loom lint-pattern` enforces the size.
- Card 3 and card 5 read owner state only from the local Bolt (`ParseOwners` over `board.json`, `Owners.Effective`), card 6 reads origin's Bolt tip only inside `ReconcileOrigin`, and cards 10, 14, 16 and 24 take origin verdicts from a run-scope or hub-scope reconcile.
  No card reads origin's Bolt outside `ReconcileOrigin`.
  The `Takeover` guard in `SetOwners` reads the Bolt it just pulled, which the Constraint's local-Bolt rule covers.
- Card 12 keeps `ownerDue`, `ownerSleep` and `ownerCovered` on `innerRunProducer`, runs `ownerStep` after `checkChild` and before it returns an end on a status it read, and keeps `ownerDue` for the next `Call`'s first check when the end carried no status.
  That settles a post-wake owner check across a `Call` boundary and across a state change arriving on the same check as the sleep.
- Card 9 handles the resume prime claim as the focus asks: an unreachable reconcile makes no claim and keeps the Warn and resume, and a failed claim on a reachable reconcile warns and goes on to the owner verdict.
  This matches the decision record's rule that every other check reads the local Bolt when origin is unreachable.
- Cards 1 and 24 name their packages in the background files and the overview that list such packages: `pattern/PATTERN-leaf-packages.md` and `docs/overview.md` for `internal/machinename`, and `pattern/PATTERN-cli-cobra.md` and `docs/overview.md` for `internal/adoptcli`.
  No other `pattern/` file lists leaf or CLI packages.
- The focus directive numbers cards as the round-1 plan did, so I matched each item to the current card by topic: owner pause and InnerRun are card 12, batten's resume claim is card 9, and `adoptcli` is card 24.
- `_lyx/plan/lens-review.md` is not a card and is not in the Card Index, so I judged no finding against it.
