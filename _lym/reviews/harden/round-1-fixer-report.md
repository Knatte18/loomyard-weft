---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The guard change is as proposed. The test rows went into the guard's own test, internal/fabriccli/roleguard_test.go, not cli_refusals_test.go: the guard reads the strand name from the process environment, which a test sets with t.Setenv, and cli_refusals_test.go runs its rows in parallel with no strand name. The existing row that asserted a push with an arbitrary --weft-path passes was the wrong assertion and is replaced."
  - id: F2
    disposition: fixed-differently
    reason: "The namespace choice is made once, in a new archiveTagNamespaceAt, used by pushArchiveTag for every archive push (task discard, owner archive, parent-merge archive) and by remoteTagsUnder and readSideOwnerArchives for every origin read of those tags, instead of only the task discard. The orch brief asked to check every other archive push for the same code-store read. The proposed change to the origin-side check behind dirtyArchivedOnRemote was not made: that check reads no origin ref, only a non-empty tag and lease (internal/fabricengine/destroy.go:1159-1166)."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed-differently
    reason: "The refusal text is as proposed. Its assertion lives in internal/fabriccli/roleguard_test.go, where the task-session refusal is asserted, not in cli_refusals_test.go, which has no strand name."
---

# Round 1 fixer-report

Commits, each green under `lym gate test --upto 1` for the packages it touches:

- `f1ee38d98` fabriccli: fix F1, F7
- `5c76ebb51` fabricengine: fix F3, F4, F5
- `e20e7112e` fabricengine: fix F2, F6
- `94bf4a69c` tools/sandbox: fix F2, F4, F5, F6 (the fabric suite's F23 scenario)
- `da08accbf` fabricengine: fix gate findings (comment lines)
- `8a8b8debb` fabricengine: fix gate findings (destructive guard and test tiers)

## Gate repair

The second gate run (verify) failed three `cmd/lym` guards, all on F6's code:
- `TestNoDestructiveBypass_FabricengineProductionSource`: `clearPendingTeardown`'s `os.Remove` sat in `remoteteardown.go`, which no guard entry covers.
  The pending-teardown record now lives in its own file, `internal/fabricengine/pendingteardown.go`.
  That file has its own entry in `destructiveGuardAllowlist` (`cmd/lym/destructiveguard_test.go`), giving the reason:
  the removal deletes fabric's own record beside the teardown lock, never operator content.
- `TestTierPurity_UntaggedTestsSpawnNothing` and `TestTierTags_OneTierPerTestFile`: `remoteteardown_test.go` mixed the tier-0 tests with a tier-1 test that runs git.
  The tier-1 `TestRunTeardownChild_ClearsThePendingRecordOnceItEnds` moved to `pendingteardown_integration_test.go`,
  and the tier-0 `TestPendingTeardowns` moved to `pendingteardown_test.go`.

Behaviour is unchanged.
`lym gate test --upto 1` over `cmd/lym`, `internal/fabricengine` and `internal/fabriccli` passes.

The first gate run refused five comment lines this round added.
Four reached 120 characters:
- `doc.go:525`
- `pull_integration_test.go:688` and `:690`
- `taskbranchdiscard_integration_test.go:158`

One was wrapped mid-sentence: `remoteteardown.go:260`.
Each is now split at a sentence boundary, one sentence per line.
A scan of every comment line added since `0e4782373` finds none of 120 characters or more.
The fabricengine gate passes again.
No code or test logic changed.

## Changes

- **F1** (fixed differently): `sandboxedRoleRefusal` (`internal/fabriccli/roleguard.go`) now lets a push with `--weft-path` or `--warp-path` through only when every path it names passes one of three tests.
  The path is the strand's own task worktree, or that worktree's records sibling (`isOwnPairPath`), or it lies under the strand's own sandbox root (`inOwnSandboxRoot`).
  Otherwise the push gets the task-session refusal.
  The worktree-identity check moved into `isOwnWorktree`, which both rules share.
  New rows in `TestSandboxedRoleGuard` cover the strand's own pair, its records sibling alone, and a pair under its own sandbox root, all passing.
  Another worktree's pair, a mixed own/other pair, the same slug in another hub, and paths outside any hub are all refused.
  The proposed change I did not make, and why, is in the frontmatter.
- **F2** (fixed differently): `archiveTagNamespaceAt` (`internal/fabricengine/archive.go`) answers the records namespace for a records store that reads it.
  It answers the same for a code store whose hub's records store reads it from the same origin URL; the records store is the records sibling of the code store's prime worktree.
  `pushArchiveTag`, `remoteTagsUnder` and `readSideOwnerArchives` use it, so archive pushes and the origin reads that check them use one namespace.
  A failed discard's manual tag push in `failedRemoteSteps` now names its destination.
  The package doc's one-repository section states the rule.
  New tests:
  - `TestDiscardTaskBranch_OneRepositoryHubArchivesUnderTheRecordsNamespace`, over hubforge's one-repository builder: origin holds `refs/lym/tags/archive/<slug>/<sha12>` and nothing under `refs/tags/`.
  - `TestFailedRemoteSteps_DiscardTagPushNamesTheHubsArchiveNamespace`, which pins the way forward's destination in both layouts.
- **F3**: `unpushedCommits` (`internal/fabricengine/pushclassified.go`) falls back to `refs/remotes/origin/<branch>` on `gitrepo.ErrNoUpstream`, as `failRejection` does.
  The not-fast-forward row for a branch with no upstream now asserts that the record's commits include the local commit.
- **F4**: `pushClassified` reads the store's origin layout before `checkOrigin` on the code and records sides.
  A layout refusal is recorded as an other-class failed push, and the record's error carries the restore commands.
  New row `APairPushFromARecordsStoreWithNoNamespacedFetchRefspecRecordsTheRestoreCommands` in `TestPushFailureClasses`, over hubforge's one-repository builder.
  It asserts the error names `git -C <store> config --replace-all remote.origin.fetch`, the record exists, and one mail is sent.
- **F5**: `pullWeftArm` (`internal/fabricengine/pull.go`) reads the layout first and returns its refusal as the pull's error.
  New test `TestPull_RecordsStoreWithNoNamespacedFetchRefspecFails`: origin's records branch is ahead and the refspecs are unset.
  The pull fails naming the restore command, `WeftPulled` is false, and the records branch stays put.
- **F4, F5 test seam**: `ForgetOriginNamespaceForTest` (`export_test.go`) drops the per-process cache of one store's layout.
  This stands in for the fresh process the CLI runs each verb in, after the test rewrites the store's config.
- **F6**: `childHold.finish` writes a pending-teardown record, `_bolt/.lym/teardown/<slug>.pending.json`, before it hands the job to the child.
  `RunTeardownChild` removes it once the job has run, or when it receives an empty input.
  `PendingTeardowns` lists every record whose teardown lock no process holds exclusive, with the re-run that finishes it.
  It probes the lock shared, so it never holds back a push.
  `lym fabric status` and `lym fabric prune` print the list as `pending_teardowns`; the help text and the package doc say so.
  Tests:
  - `TestPendingTeardowns`: a record with no lock holder is listed with `run \`lym fabric remove <slug> --remote\` again`, or the discard re-run; a record whose lock is held is not listed.
  - `TestRunTeardownChild_ClearsThePendingRecordOnceItEnds`: a finished teardown leaves no record, both for an empty input and for a job that ran.
  - `TestChildHold_Finish` now asserts that the record is written when a job is handed over.
- **F7** (fixed differently, test placement only): the task-session refusal now adds "or run it in a hub made with \`lym harden sandbox new\` under <worktree>/.lym/harden-sandbox", built from `sandboxRootRel`.
- **Sandbox suite**: `tools/sandbox/SANDBOX-FABRIC-SUITE.md` scenario F23 now drives F2's discard tag, F4 and F5's missing refspec, and F6's killed teardown child.
  The sandbox coverage tests stay green.

## Verification

- Sabotage proofs: each fix was broken, and the new tests failed at their intended assertions.
  - F1: the guard's path check forced to pass; four refused rows failed.
  - F3, F4, F5: each check disabled; the commits, error-class and `WeftPulled` assertions failed.
  - F2: namespace mapping disabled; the tag landed under `refs/tags/`, and both new tests failed.
  - F6: the record write, the child's removal of the record, and the lock probe each broken; `TestChildHold_Finish`, the clear test and `TestPendingTeardowns` failed.
- Determinism: every new test passed 20 times in parallel (`-count=20`, 300 passes, no failure).
- Live drive in a one-repository sandbox hub built by `lym harden sandbox new --single-repo` after the last source change:
  - Probe 3: `fabric push --weft-path /nonexistent/weft --warp-path /nonexistent/warp` from the worktree is refused.
  - Probe 7: `batten cancel t4 --remove` leaves only `refs/lym/tags/archive/t4/*` on origin.
  - Probe 12: the push records an other-class failure whose error names the restore commands, `fabric status` lists it, and `fabric pull` fails with the same refusal.
  - Probe 15: after `kill -9` of the teardown child, `fabric status` and `fabric prune` list t8 with `run \`lym fabric remove t8 --remote\` again`.
    That re-run deleted both branches on origin and emptied the list.
  - The sandbox was cleared afterwards, `lym harden sandbox list` shows no hubs, and the scratch scripts were removed.
- Deviation: the new regression tests require tier 1, not tier 2.
  They run over real git hubs built by hubforge, plus the in-process child body for F6.
  The killed-process scenario itself was driven live in the sandbox and is recorded in the suite's F23, not as a tier-2 test.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
