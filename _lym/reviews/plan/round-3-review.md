---
verdict: APPROVED
prior:
  - id: F16
    status: resolved
    answer: "Card 10 no longer lists internal/loomcli#intentionallyNil under Edit and its TeardownDeps bullet keeps only the nilLegal addition. envkit.walkNil skips nilLegal[path] before it reads a field, so NilSeams never reports Teardown.Push or Teardown.Level, and neither loomcli's nor battencli's TestWire_EverySeamFilled sees them. Neither intentionallyNil map lists them, so the second loop has no entry to fail on. Each seam sits in exactly one list. envkit's TestFullEnv_FillsEverySeam and TestNilSeams_SkipsNilLegalPaths walk the same function and still pass. Card 10 also adds TeardownPush and TeardownLevel rows to battencli's TestWire_BuildsLazilyForANonexistentTaskWorktree, so a wire that leaves them nil is still caught."
  - id: F17
    status: resolved
    answer: "Card 1 now creates gitrepo.Repo.DeleteRemoteTrackingRef, which removes only the tracking ref that the remote's fetch refspecs map the branch's origin ref to, in one hold of the remote-refs lock, and touches no config. It sits in gitrepo as PATTERN-gitrepo-client-boundary requires, and internal/gitrepo/doc.go is under Edit, since its remote-refs lock line lists the methods run under the lock. The branch keeps branch.<branch>.remote and .merge, so card 2's RunCommitsAhead and TipsOnOrigin take their upstream-set, tracking-ref-absent branch and read level and held, which ends card 10's Level loop and the dobby restart. UpstreamSHA now wraps ErrRefNotFound for that state, so checkOrigin goes on to look the branch up on origin, finds it gone with its entry ended, and skips it with nil; the next push no longer takes the ErrNoUpstream path that recreates a branch origin deleted."
---

No findings this round.

## Settled by reading the code

- `internal/testkit/envkit/envkit.go` `walkNil` tests `nilLegal[path]` before it reads the field.
  A `nilLegal` path is never in `NilSeams`' result.
  `internal/loomcli/wiring_seams_test.go` and `internal/battencli/wire_seams_test.go` both fail a listed `intentionallyNil` key that `NilSeams` does not report.
  Card 10's single placement in `nilLegal` satisfies both loops and `FullEnv`.
  The loomcli map still lists `Teardown.Shutdown` and `Teardown.Remove`, which `nilLegal` does not skip, so the cards leave those two entries as they are.
- `gitrepo.Repo.DropRemoteTracking` unsets `branch.<branch>.remote` and `.merge`, so card 1's reason for not using it holds.
  `UpstreamSHA` returns a plain resolve error for a set upstream with an absent ref, and `checkOrigin` returns it as it is today, so card 1's change to `UpstreamSHA` and `checkOrigin` is needed for the next push.
  No other `UpstreamSHA` caller tests for `ErrRefNotFound`, and the `ErrRefNotFound` checks in `fabricengine` read `RefSHA` results.
- `HasUnpushed` folds a non-zero git exit into true, so a records branch whose tracking ref the delete removed still reads unpushed.
  `PushPairAnchored` then reaches `recordsSideNotLanded`, which returns the `ErrEntryEnded` error on the first and every later push.
- `checkOrigin` runs from `pushClassified`, which no `HoldRemoteRefs` holder calls, so the lock `DeleteRemoteTrackingRef` takes is not nested.
- The plan's tests reach the delete through the pair push, in rows of tests that already exist in `internal/fabricengine/pushanchored_integration_test.go`, and the card plans no gitrepo test of its own.

## Rejected

- The card 1 paragraph that opens "It also gains rows in the mode" follows the `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` paragraph, so "It" can read as that test.
  Its rows are `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` rows by their content, and an implementer places them in either test without a failing gate.
  Not BLOCKING, and an incremental round raises only BLOCKING findings.
- A branch whose tracking ref was pruned while origin holds it again would reach `holdBackRewound`, whose `@{u}` read may fail.
  The path needs an ended entry's branch to reappear on origin and no card's behavior changes for it, so no finding.
