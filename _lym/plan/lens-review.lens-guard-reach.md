```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: I traced each guard, refusal and new test row the twelve cards add.

**Findings**

1. BLOCKING, card 12 (`movesPair` rewrite) against `TestBoltReplayers`: card 12 makes the `ErrReplayerMissing` refusal of `ReconcileOrigin` unreachable, and no card lists the test that asserts it.
   - The row "With no replayer registered the ended-set read and the owner read fail with ErrReplayerMissing" (`internal/fabricengine/boltreplayers_integration_test.go:100-113`) builds `newMovesPair` and passes an `OwnerOf` seam. It expects `ReconcileOrigin` to wrap `ErrReplayerMissing`.
   - Card 12 rewrites `movesPair`'s template to write no run status. `recordedPairs` then returns no slug.
   - `readOriginOwners` then returns early on `len(slugs) == 0` (`internal/fabricengine/originreconcile.go:730`).
   - `readOriginEnded` gets no candidates (`originreconcilemoves.go:617`), so neither ledger read runs. The row fails.
   - `internal/fabricengine_test#TestBoltReplayers` is on no card.

2. BLOCKING, card 7, the new row `AnOriginTipMovedAfterTheReadLosesTheLease`: the stated way to move origin cannot reach the lease refusal.
   - The card has a pre-receive hook let another clone's push land "between the reset's read of origin's tip and the leased reset".
   - No push to origin runs between `resetUnnamedTip`'s `UpstreamSHA` read and its `updateRemoteBranch` (`internal/fabricengine/pushclassified.go:435-464`), so nothing can fire that hook in between.
   - A hook on the leased push itself that moves the ref fails server-side. It never reaches the client-side "stale info" match. `UpdateRemoteBranchLeased` says such a hook rejection "never [returns] the sentinel" (`internal/gitrepo/push.go:396-397`, `:418-421`).
   - So `resetUnnamedTip`'s `errors.Is(err, gitrepo.ErrLeaseRejected)` branch (`pushclassified.go:465`) is not reached. The reset returns its wrapped error instead, and that flows into the retry loop.
   - D4's tier1 criterion for an action refused at execution therefore has no reaching test.

3. MEDIUM, card 12, the deleted `OwnerOf`-error row: the claimed holder does not reach the branch.
   - Card 12 deletes the reconcile's `OwnerOf`-error row as "held by `TestClassifyEntries` and `readOriginOwners`' error return in the kept reads".
   - `TestClassifyEntries` is a core test and never calls `readOriginOwners`.
   - Under the Shared Decision "fabric tests of this concern record no run", no kept fabric test has a recorded slug, so `readOriginOwners` returns before the seam call (`originreconcile.go:730`, `:745-748`).
   - The seam-error return goes untested.

4. MEDIUM, card 11, the new dirty-at-execution row: it does not reach the guard D4's criterion names.
   - The row has the pair's code checkout turn dirty inside the verdict hook. It expects the fast-forward to be refused at execution.
   - That refusal is `ffWorktree`'s own check in `originreconcilemoves.go:176-192`, which runs a `MergeFFOnly`. It is not `destroy.go`'s check.
   - Today's code also checks dirtiness only at execution, so the row pins nothing a snapshot could make stale.
   - D4's "refused by `internal/fabricengine/destroy.go`'s checks at execution" is reached by no row. Card 7's lease row was the other candidate, and finding 2 shows it unreachable.

5. LOW, card 9, the refusal rule: `OriginJudge` refuses a moving run scope whose run-locks seam is unset, though that run never takes the lock.
   - The rule refuses when `SeamRunLocks` is unsupplied for any scope that is not judge-only.
   - A run scope never takes the prime lock: `movePrime` skips it when `m.scope.Run != ""` (`originreconcilemoves.go:360`).
   - D13 has a verb refuse only when "its run would use one", so the refusal here guards a state the run never meets.

**Per guard, reachable or not**

| Card | Guard or row | Reachable |
|---|---|---|
| 1 | `RefuseUnsupplied` rows | yes, core rows |
| 3 | `FieldSeams` row of `TestSnapshotReader_ReadsAHub` | yes |
| 4 | `OriginRunStatus` refusal via `TestInstallRequiredSeams` | yes |
| 7 | `UnseededBoardRefusal` | yes |
| 7 | lease-lost row | no (finding 2) |
| 8 | records-landing verdicts | yes |
| 9 | `TestReconcileOrigin_RefusesAnUnsuppliedSeam` | yes |
| 9 | run-locks refusal for a run scope | over-broad (finding 5) |
| 10 | teardown seam-call rows | yes |
| 11 | live-pair probe row | yes |
| 11 | `TestReconcileOrigin_PrimeLockHeld` | yes |
| 11 | dirty row | reaches `ffWorktree`, not `destroy.go` (finding 4) |
| 12 | `ErrReplayerMissing` row of `TestBoltReplayers` | no (finding 1) |
| 12 | `OwnerOf` error return | no (finding 3) |

hubforge's one-time install reaches every hub-building path, so the refusals stay unreached in tests that build through it:
- `CopyHub`, `CopyRegistered` and `SharedHub` reach `copyTemplate`, or build their templates through `cloneHub` at `template.go:224`.
- `NewHub`, `NewOneRepoHub` and `CloneSecond` reach `cloneHub`.

**Tool calls**

- Listed hubforge's functions, to locate the hub-building paths card 3 hooks; my context lacked hubforge's file layout.
- Grepped `CopyHub`, `copyTemplate` and `SharedHub` definitions, to confirm every export reaches `cloneHub` or `copyTemplate`; my context lacked `template.go`.
- Read `template.go:426-510` and `gitrepo/push.go`'s lease detection, to judge the card 7 lease row; my context lacked both.
- Read `UpdateRemoteBranchLeased` (`push.go:392-430`), to confirm a hook rejection never maps to `ErrLeaseRejected`; my context lacked it.
- Listed the package and hubforge use of each test that runs a reconcile or an origin read, to check whether any meets the refusal unintended; my context lacked them.
- Read `boltreplayers_integration_test.go:96-116`, to check whether card 12's `movesPair` change starves the `ErrReplayerMissing` row; my context lacked that test body.
