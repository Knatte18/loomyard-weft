---
dispositions:
  - id: F6
    disposition: fixed-differently
    reason: "The code and test changes are made as proposed. The proposed edit of pattern/PATTERN-single-repo-hub.md is not made: the parent ruled in creel message 2317 that PATTERN stays unedited and the departure is recorded as a Pattern-Departure trailer, which commit 704c7aea6 carries."
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
---

# Round 2 fixer-report

All three findings hold against the code and are fixed.
Commits: `704c7aea6` (F6 and F8, one group through `doc.go`), `ba00dc2e3` (F7),
and `e9d677cb6`, which extends scenario F23 of `tools/sandbox/SANDBOX-FABRIC-SUITE.md`.
A later commit repairs the round's one gate finding, a wrapped comment break at `internal/fabricengine/doc.go:533`.
The line after it opened `so origin's ref is read`, which the comment lint does not read as a clause after a comma;
it now opens `so the ref on origin is read`.
That commit changes no code, and `lym loom lint-comments --base c25e17ac2` reports no finding over the round's commits.
Nothing is pushed.

## Changes

- **F6, fixed differently** (`internal/fabricengine/destroy.go#pushCloneSeed`).
  A records push for which `seedPushOutcomeUnknown` holds now reads origin's `refs/lym/heads/<records-branch>` before any teardown.
  `withdrawUnansweredBoard` is generalised to `withdrawUnansweredSeedPush(seed, branch, pushedTip, withdraw, cause)`, and both pushes go through it.
  A ref at the seed's tip is withdrawn: `restoreSeededRecordsBranch` for the records branch, the new `deleteSeededBoard` for the board.
  The hub is then removed.
  A failed read, a failed withdrawal, or a ref origin still holds as the push was leased on it keeps the hub.
  The keep error names the refs origin may hold and the `ls-remote` and `push origin --delete` commands.
  "As the push was leased on it" is absence for the board.
  For the records branch it is the tip the clone's check observed, so a leftover branch the pending push may still replace keeps the hub as well.
  A ref at any other tip is another clone's and is left alone, as before.
  The keep error's wording changed with the generalisation:
  `origin does not hold the pushed <board|records branch> yet` and `list what origin holds with`.
  - **Tests.** `TestCloneSingleRepo_BoardPushKilledAfterItLandedLeavesNothing` is replaced by the table test `TestCloneSingleRepo_SeedPushKilledBeforeOriginsAnswer` (tier 2, Linux only).
    It keeps the old row as `BoardLanded` and adds three.
    `RecordsBranchLanded`: a `post-receive` kills the client once the records ref is updated; it asserts no hub and an empty `refs/lym/`.
    `BoardNotApplied`: a `pre-receive` kills the client on the board ref and exits non-zero;
    it asserts the hub is kept, the error names both refs, and origin still holds the records branch.
    `RecordsBranchNotApplied`: the same on the records ref; it asserts the hub is kept, the error names the records ref, and `refs/lym/` is empty.
    The review asked for two tests; the four cases are rows of one table, and the last row is an addition.
  - **Not made as proposed.** The review's `fix` key asks for an edit of `pattern/PATTERN-single-repo-hub.md`, section The clone.
    The parent's creel message 2317 says to record a `Pattern-Departure` trailer and not to edit PATTERN.
    Commit `704c7aea6` carries the trailer against "A failed seed removes what it pushed and leaves nothing behind".
    The pattern file is unchanged, so it does not name the new test.
  - The package doc (`doc.go`) and the doc comments of `pushCloneSeed` and `PushCloneBoard` (`clone.go`) state the rule for both refs.
- **F7** (`internal/fabricengine/cleanup.go#primaryWeftBranch`).
  One way forward is built once and used by every prime-key refusal.
  It renders the value as `RecordsBranchName("<clone-time-code-branch>")`, so no literal suffix is left in the function.
  It says to name the prime pair's own records branch, the one of the code branch the hub was cloned on and never of the branch the prime pair is on now.
  It also says that fabric cannot tell a wrong records branch from the right one.
  After the `BranchExists` check, a key `WeftWarpSlug` does not accept is refused as "not a records branch name".
  `TestCleanup_RefusesWhenPrimaryWeftBranchIsUndeterminable` gains the row `OneRepositoryKeyNamesTheBoardBranch`, run with the prime pair on `alt`.
  The three checkouts onto `alt` in that test file now share the helper `checkoutPrimePairOnAlt`.
  A key naming another pair's existing records branch, such as `alt-weft`, still passes, as the review states it must: the key is the only record.
- **F8** (`internal/fabricengine/recordslayout.go#recordsStoreClaim`).
  The one `git config --get-regexp` listing now also matches every `branch.<name>.merge`.
  A value that starts with `RecordsNamespaceRoot` is a claim, named as `the upstream setting <key> <value>`.
  `TestOriginNamespaceAt`'s faulted-store table gains the row `OnlyABranchMergeSettingLeft`: the code fetch refspec, no push refspec, no prime key, `branch.main.merge refs/lym/heads/main`.
  The parent limited this finding to `recordslayout.go`; the one test row sits in that file's own test, `recordslayout_integration_test.go`, and `doc.go` gains half a sentence.
- **Sandbox suite.** F23 gains a client killed after the records ref landed, a client killed in a `pre-receive` that declines the board (the kept hub and its way forward),
  a store whose one remaining mark is `branch.main.merge`, and a prime key naming the board's branch.
- `docs/overview.md` and `PATTERN.md` carry no statement these changes move.

## Verification

- `lym gate test --tier 0 --tier 1 --tier 2 ./internal/fabricengine ./tools/sandbox` on the final tree:
  tier 0 1025 passed, tier 1 1582 passed, tier 2 5 passed, 0 failed.
- Sabotage, each fix switched off in turn and restored afterwards; `git status` is clean.
  - Records push handling off: `RecordsBranchLanded` failed at `origin's records namespace = [refs/lym/heads/main-weft]; want []`,
    and `RecordsBranchNotApplied` failed at `the container holds []; want the one kept hub`.
  - The still-leased case answering nil instead of the keep error: `BoardNotApplied` failed at the kept-hub assertion and at `origin's records namespace = []; want [refs/lym/heads/main-weft]`.
  - The `WeftWarpSlug` refusal off: `OneRepositoryKeyNamesTheBoardBranch` failed at `Cleanup() = nil error`.
  - The merge-setting claim off: `OnlyABranchMergeSettingLeft` failed with `originNamespaceAt(...) = ({Heads: Tags:}, <nil>); want an error`.
- Determinism of the tier-2 test: `-count=60` passed 300 of 300 while twelve loops of `git rev-list --all --objects` loaded the machine;
  the load average rose from 18 to 42 during the run, and the run took 98.5 s against 8.7 s for `-count=40` unloaded.
  Each repeat runs its four rows in parallel.
  No gate key runs tier 2 in side-by-side copies, so the load came from outside the gate.
  The hooks kill the client strictly before or strictly after origin's ref update, and lym awaits git's exit, so the test waits on nothing.
- Live, with `.dev-bin/lym` rebuilt from `e9d677cb6` by `lym harden sandbox new`:
  - F7, prime pair moved by hand to `alt` and `alt-weft`: a key of `main` refused as "not a records branch name";
    an unset key and `nosuch-weft` refused with the new way forward; `main-weft` listed `main-weft` protected.
  - F8, the store left with only `branch.main.merge`: `fabric cleanup --remote` and `board sync` both refused, naming the upstream setting and the fetch refspecs to restore;
    origin kept `refs/heads/main`, `refs/lym/heads/main` and `refs/lym/heads/main-weft` unmoved.
  - F6, three seeds through `.dev-bin/lym harden sandbox new --single-repo` under a hook template.
    Client killed after the records ref landed: `the seed push of the records branch failed ... way forward: run the same clone again`,
    and the hook log shows the ref applied and then deleted at the pushed tip.
    Client killed in a `pre-receive` declining the records ref: the keep error naming `refs/lym/heads/main-weft`.
    The same on the board ref: the keep error naming both refs.
  - `lym harden sandbox clear` afterwards left `hubs.json` and its lock, `lym harden sandbox list` is empty, and no process names the sandbox root.

## Disputed

None.

## Deferred

None.

## Rulings

None: no `decision`-class finding is in this review.
The parent's creel message 2317 directed the F6 pattern handling and the F8 scope, both recorded above.
