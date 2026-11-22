---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

# Round 1 fixer-report

Every finding is fixed as its `fix` key proposes.
Commits: `5a5f952a8` (F1, F4, F5, one group through `destroy.go`), `a628d0fc9` (F2), `8bd71e31f` (F3),
and `5a1bf5a39`, which adds scenario F23 to `tools/sandbox/SANDBOX-FABRIC-SUITE.md`.
A later commit repairs the round's gate findings:
comment lines in `cleanup.go`, `clone_singlerepo_integration_test.go`, `destroy.go`, `doc.go` and `recordslayout.go`
that wrapped mid-clause or reached 120 characters now break one sentence per line and stay under 120 characters.
That commit changes no code.
A second gate repair moves the tier-2 F1 test and its hook out of `clone_singlerepo_integration_test.go`.
They now live in `internal/fabricengine/clone_singlerepo_killedpush_integration_test.go`,
since `TestTierTags_OneTierPerTestFile` (`PATTERN-test-speed`) allows one tier per test file.
A third gate repair shortens that new file's header comment, whose two lines reached 120 characters, to three shorter sentences.
Nothing is pushed.

## Changes

- **F1** (`internal/fabricengine/destroy.go#pushCloneSeed`).
  When the board push fails, `seedPushOutcomeUnknown` checks whether it may have landed anyway:
  a killed or timed-out git (exit -1), or a git exit with no `[remote rejected]` or `! [rejected]` line.
  A lost lease, a rejection, a stall origin never answered, and a git that never ran count as not landed.
  When the outcome is unknown, `withdrawUnansweredBoard` reads `git ls-remote origin refs/lym/heads/main` first.
  A board at the seed's tip is deleted with `DeleteRemoteBranchLeased`, and only then is the records branch restored and the hub removed.
  A board at another tip belongs to another clone and is left alone.
  When the read fails, origin does not list the board, or the delete fails, the hub is kept.
  The error then names both refs, the `ls-remote` and `push --delete` commands, and the hub to remove before running the clone again.
  `checkSeedPush` now accepts a lease on the board's pushed tip, so the delete passes the destructive gate.
  New test `TestCloneSingleRepo_BoardPushKilledAfterItLandedLeavesNothing` (tier 2, Linux only).
  Origin's `post-receive` SIGKILLs the client `git push` once the board ref is updated, then the test asserts no hub and an empty `refs/lym/`.
- **F2** (`internal/fabricengine/recordslayout.go#originNamespaceAt`).
  When no fetch refspec maps `refs/lym/heads/`, one `git config --get-regexp` reads `remote.origin.push` and `lym.primeRecordsBranch`.
  If either one marks the store as a records store, the answer is an uncached error with the `--replace-all`/`--add` fetch refspec way forward.
  With origin removed, the way forward is `git remote add --no-tags origin <code-url>` plus both refspec pairs.
  `TestOriginNamespaceAt` gains rows for an unset fetch refspec, a `+refs/heads/*` refspec and a removed origin, each with the key set.
- **F3** (`internal/fabricengine/cleanup.go#primaryWeftBranch`).
  It now checks the key against the records store with `BranchExists`.
  A key naming a missing branch fails closed, naming the value and the same `git -C <store> config` way forward.
  `TestCleanup_RefusesWhenPrimaryWeftBranchIsUndeterminable` gains row `OneRepositoryKeyNamesNoBranch`, run after `Checkout(alt)`.
- **F4** (`internal/fabricengine/destroy.go#seedPushCause`).
  The existing `board` parameter is the review's `recordsLanded`: the board push runs only once the records push landed, so no new parameter was needed.
  A refused board push now says origin took the records branch but its hooks or rules declined `refs/lym/heads/main`, and to fix or lift that rule and run the same clone again.
  A refused records push keeps the host-refusal wording.
  The `RefusedBoard` row asserted the old wording; it now asserts the new one and that the host wording is absent.
- **F5** (`internal/fabricengine/destroy.go:1767`).
  The upstream loop sets `branch.<b>.remote` and `branch.<b>.merge` for both the board and the seeded records branch.
  `TestCloneSingleRepo_SeedsTheNamespace` asserts `<records>@{u}` resolves to `refs/remotes/origin/<records>`.
- The package doc's one-repository section (`internal/fabricengine/doc.go`) states the unanswered-board rule, both upstreams and the layout read's new error.
  `docs/overview.md` and `PATTERN.md` carry no statement these changes move.

## Verification

- `lym gate test --tier 0 --tier 1 --tier 2 ./internal/fabricengine ./cmd/lym`, run again after both gate repairs:
  tier 0 1379 passed, tier 1 1616 passed, tier 2 1 passed, 0 failed, `TestTierTags_OneTierPerTestFile` included.
- Sabotage: with each of the five fixes switched off, its new or corrected assertion failed at the intended line.
  The F1 test ended with origin holding only `refs/lym/heads/main`, P9's end state.
  All five fixes were restored afterwards.
- Determinism of the F1 test: 20 passes out of 20 with `-count=20`.
  Tier 1 ran the clone, cleanup and layout tests 20 times alongside it as load.
  The hook kills the client only after origin updated the ref, and git's exit is awaited, so the test needs no deadline.
- Live, on sandbox hubs built from the committed HEAD:
  F3 refused with the new message; F2 refused with the fetch refspec way forward and passed again once the refspecs were restored.
  For F5, a hub seeded by `.dev-bin/lym harden sandbox new --single-repo` showed `main-weft@{u}` = `refs/remotes/origin/main-weft` and `## main-weft...origin/main-weft [ahead 1]`.
  The installed `lym harden sandbox new` seeds in-process with the installed binary, so only the `.dev-bin` run tests the fix.
  `lym harden sandbox clear` afterwards left only `hubs.json` and its lock.
- `lym gate test --tier 0 --tier 1 ./tools/sandbox`: 106 passed.

## Notes

- The new F1 test is tier 2 because the fix rules place a test for a defect found live there.
  The F2, F3 and F5 rows go into the existing tier-1 tests their `fix` keys name.

## Disputed

None.

## Deferred

- **Gate finding: `harden-live` red with `[setup failed]` for `internal/fabriccli`, `internal/fabricengine` and `internal/gitrepo`.**
  None of the three packages ran a test.
  The run record `../_bolt/.lym/gate/runs/20261122T102348Z-tier0-3257666.json` shows the command `go test -json 'internal/fabriccli' 'internal/fabricengine' 'internal/gitrepo'`.
  Go answers `package internal/fabricengine is not in std` for each, so the targets lack a leading `./`.
  `hardenLiveArgs` (`internal/loomcli/hardenwiring.go:242`) appends the campaign's `hardener.packages` verbatim, and those are worktree-relative directories.
  The repair belongs either in that campaign config (`./internal/...`) or in `hardenLiveArgs`.
  Both are outside this round's write surface, the campaign's modules.
  The live call also runs the installed lym, so an edit here could not clear this round's live gate.
  I told the parent through creel (message 2280).
  The gate ran again at 11:24:45 (`../_bolt/.lym/gate/runs/20261122T102445Z-tier0-3263069.json`) with the same command and result.
  I then asked the parent to choose:
  rewrite `hardener.packages` with `./` prefixes, change `hardenLiveArgs` and rebuild the installed lym, or authorize that change in this worktree.
  The same packages pass through `lym gate test --tier 0 --tier 1 --tier 2 ./internal/fabricengine`.

## Rulings

None reached this round.
