# fabric: retry refused pushes, prime pushes records only, neutral names

Three changes to fabric's pushes and exported names.

## What changes

- **Pair creation survives a transient push refusal.**
  `Add`'s warp and weft branch pushes retry a push whose stderr holds a `! [remote rejected] <ref> -> <ref> (failed)` line for the pushed ref, at most 4 attempts with exponential backoff and jitter (about 1s, 2s, 4s).
  Any other reason (hook declined, non-fast-forward) and any exec or auth error returns on the first attempt.
  The sleep and jitter are injectable, and each retry is logged with the attempt number and stderr line.
- **A failed `Add` leaves no warp branch it created.**
  `rollbackAdd` deletes the warp branch locally whenever this `Add` created it, and on origin only when the pre-flight probe found it absent, step 11 attempted its push, and the lease on the pushed commit holds.
  A branch adopted from origin, or found there at or behind `HEAD`, is deleted locally only.
  Under `SkipGit` or `SkipPush` the probe does not run, so only the local branch goes.
- **The prime no longer publishes unpushed `main` commits.**
  In the hub's prime, `lyx fabric push`, `lyx fabric sync`, `Fabric.Commit`'s spawned push (so `lyx fabric commit` and `lyx config`'s `sync`) push the records side only.
  `push` reports `code_push_skipped` when the code branch has unpushed commits, and `sync` records no code-side `push_spawned` entry.
  Task pairs push both sides as before.
  A prime-name resolution failure fails the call before anything is committed or pushed.
- **Fabric's vocabulary stays inside fabric.**
  Every `vocabExempt` spelling is renamed or moved, and `vocabExempt`, its entry types and the stripping step in `fabricVocabularyFailures` are deleted.
  `vocabScanAllowlist` gains nothing.

Closes #433

## Design decisions

- The retry classifier reads only the reason on the ref line, so `remote:` lines neither qualify nor disqualify a push.
  The retry lives in `pushretry.go`, separate from `pushRebaseFreeRetryingOnce`, because `Add` pushes `-u origin <branch>` through `gitexec`.
- Branch ownership for the rollback is proved by creation, not by naming scheme.
  `destroy.go` mints a proof when `git worktree add -b` succeeds, like `createdToken`, and the gate order stays containment, ownership, dirtiness, force.
  This narrows the rollback's "no origin branch is ever touched" rule to the one warp ref the same call created, pinned to the commit it pushed; `rollbackAdd`'s doc and `pattern/PATTERN-fabric-git.md` state the exception.
  When the probe found the branch absent, a real rejection at step 12 deletes the warp branch pushed seconds earlier.
- The prime is identified by `PrimeName` (new helper `IsPrimeWorktree`), the predicate batten's refusal already uses.
  The change removes a push and never adds one; landing's own push of the parent branch is untouched.
- Renames are hard cuts with no alias or fallback:

  | Old | New |
  | --- | --- |
  | `hubforge` `PairWarpWorktree`, `PairWeftSibling`, `PrimeWeft`, `WeftBare`/`WarpBare` | `PairCodeWorktree`, `PairRecordsSibling`, `PrimeRecords`, `RecordsBare`/`CodeBare` |
  | `fabricengine` `WeftWorktree`, `WeftBranchName`, `CommitWeftPaths`, `WarpLyxLink`, `ResetPairWarp` and siblings | `RecordsWorktree`, `RecordsBranchName`, `CommitRecordsPaths`, `CodeLyxLink`, `ResetPairCode` and siblings |
  | `WEFT_SKIP_GIT`, `WEFT_SKIP_PUSH` | `FABRIC_SKIP_GIT`, `FABRIC_SKIP_PUSH` |
  | Prune, Reconcile, Status `warp_worktree`; `remove` keys `warp_branch_*` | `code_worktree`; `code_branch_*` |
  | trace detail `side=warp`/`side=weft` | `side=code`/`side=records` |

  The `Fabric` handle fields became `code` and `records`, which keeps the destructive guard's `ResetHard` check side-neutral.
  `warpprobe.go`, `warpbinding.go` and `weftgit.go` became `recordsprobe.go`, `codebinding.go` and `recordsgit.go`.
- The retired spellings `lyx weft sync`, `lyx warp checkout` and `lyx.exe weft push` stay recognised by `RefScanner`, and their tests moved to `refscanner_test.go`.
  The `.weft` lock directory keeps its on-disk name; `internal/hubgeom` reaches it through `fabricengine.RecordsLockDirPath`.
- `contracts/stencils/shed/shed-template-driver.md`'s stranded-branch rule now reads `side=code`.
- Docs changed with the code: `internal/fabricengine/doc.go`, `add.go`'s header, `fabric push`/`sync` long help, `docs/overview.md`, `docs/shared-libs/lyxcwd.md`, `PATTERN.md` and the fabric vocabulary and git pattern files.

## To check

- The rename touches a wide set of files, tests and comments outside the main packages; skim for stray old spellings and for unrelated reformatting (two already-unformatted files were `gofmt`-ed).
- A run traced by an older binary carries `side=warp`, which the driver stencil no longer recognises; it escalates and an operator removes the branch.
- `batcher-v2` touches `tools/tokencount`, which resolves the records repo through fabric, so landing order may need a mechanical rename merge.
- The full `go test ./...` and the `integration`-tagged remote-backed cases were not run as part of this description; run them before landing.
- A bare `(failed)` is hard to provoke from a local bare remote; the classifier and retry loop are pinned with an injected push seam rather than a real remote.
