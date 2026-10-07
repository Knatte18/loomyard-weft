# fabric-pushes — decision record

## Goal

Fabric's pushes and exported names, three parts in one task:

- A pair creation survives a transiently refused push: a bare `(failed)` rejection is retried, and a rollback leaves no warp branch this `Add` created, locally or on origin (issue #433).
- The prime never publishes unpushed parent-branch (`main`) commits as a side effect of `lyx fabric push`, `lyx fabric sync` or the `sync` that `lyx config` runs.
- Fabric's vocabulary stays inside fabric's owner set: every spelling in `vocabExempt` is renamed or moved, and the exemption mechanism is deleted.

## Scope

In:

- `fabricengine.Add`'s two branch pushes (warp step 11, weft step 12) and `rollbackAdd`.
- The code-side push of `lyx fabric push` (`PushPairAnchored`) and every detached push child, `lyx fabric sync`'s own spawn and `Fabric.Commit`'s spawn after a landed commit included, when run in the prime.
- Every entry of `vocabExempt` in `internal/lyxcwd/enforcement_test.go`, the scan's stripping code, and every non-owner file that carries one of those spellings.
- Docs in the same commit: `internal/fabricengine/doc.go`, the `add.go` file header and `rollbackAdd` doc, `fabric push`/`sync` long help, `docs/overview.md`, `docs/shared-libs/lyxcwd.md`, `PATTERN.md` and `pattern/PATTERN-fabric-vocabulary.md`, `pattern/PATTERN-fabric-git.md`, `pattern/PATTERN-batten-bookend.md` where they name renamed identifiers or the changed behaviour.
- Close issue #433.

Out:

- Retrying `(failed)` in the anchored pushes (`PushAnchored`, `PushPairAnchored`, `CoalescePushBothAt`): a failure there is warned and the next transition pushes again, so nothing is destroyed.
- Branches left behind by binaries that predate this task; `ErrBranchExists`' existing remedy covers them.
- The on-disk `.weft` lock directory name.
- `docs/benchmarks/` and `docs/research/`: allowlisted measurement reports, never rewritten.
- A code-side push from an operator's own long-lived non-prime worktree (board note `orch-any-worktree`).

## Decisions

### D1 — A bare `(failed)` push rejection is transient and retried in Add

Decision: `Add`'s warp push (step 11) and weft push (step 12) go through one fabricengine helper that retries a push whose stderr holds a `! [remote rejected] <ref> -> <ref> (failed)` line for the pushed ref, the reason being exactly `failed`.
Other stderr lines, `remote:` lines included, neither qualify nor disqualify a push: the reason on the ref line alone decides, so `(pre-receive hook declined)` and every other reason are not retried.
It makes at most 4 attempts with exponential backoff and jitter (about 1s, 2s, 4s), with the sleep and jitter injectable for tests (`PATTERN-test-speed`).
Every other push failure (non-fast-forward, `pre-receive hook declined`, any rejection with another reason, an exec or auth error) returns on the first attempt, as today.
Each retry is logged through `internal/logger` with the attempt number and the stderr line.
The push stays a plain `git push -u origin <branch>` through `gitexec`, never forced.
Rationale: a bare `(failed)` reads as a transient server-side refusal that a retry clears, and the bound keeps a real outage from stalling `Add` for more than about 10 seconds.
Bound: the retry repeats only a non-forced push of one commit to one ref, which is a no-op when an earlier attempt had already landed; it runs only inside `Add`, and only up to the attempt count.

### D2 — rollbackAdd deletes the warp branch it created, locally and on origin

Decision: the warp-branch deletion in `rollbackAdd` proves ownership by creation in this call, not by `ownedManagedBranch`'s naming scheme.
`Add` holds a proof minted when its `git worktree add -b <warpBranch>` succeeds, minted in `destroy.go` like the worktree's `createdToken` (`PATTERN-fabric-destruction-chokepoint`); a new ownership check accepts that proof for exactly that branch name.
The gate order stays containment, ownership, dirtiness, force.
The local branch is deleted whenever this `Add` created it, in both the forked and the adopted-from-origin case, since an adopted local copy carries only origin's tip.
The origin branch is deleted only when both hold:

- the pre-flight probe ran and found no such branch on origin, so this `Add` alone can have created the remote ref; `Add` carries the probe's outcome (absent, present, or not run) to the rollback, only absent qualifies, and a branch the probe found is never deleted, whether it was adopted or was a leftover at or behind `HEAD` that step 11 pushes forward;
- this `Add` attempted step 11's push.

The remote deletion goes through `deleteRemoteBranch` with `leaseSHA` set to the commit step 11 pushed, so a branch someone moved since is refused, logged and left.
When step 11 never landed (its attempts were exhausted), the lease finds no ref at that commit and the deletion is refused and logged; a ref that landed despite the refusals is deleted.
Any refusal or failure is logged with `logger.Warn`, as today, and returned; only a deletion that happened is recorded in `rec`.
Rationale: the branch is fabric's own one-call artifact; leaving it blocks the resumed run with "branch already exists".
Bound: this weakens the rollback's "no origin branch is ever touched" rule only for the one warp ref the same `Add` call created, pinned to the commit it pushed; an adopted or pre-existing origin branch, any weft branch and any other ref stay untouched.
When the probe ran and found the branch absent, a real non-fast-forward or hook rejection at step 12 deletes the warp branch pushed seconds earlier.
Under `SkipGit` or `SkipPush` the probe does not run while step 11 still pushes, so the rollback there deletes the local branch only and leaves origin's.
`rollbackAdd`'s doc states this exception to its "no origin branch is ever touched" rule, and `pattern/PATTERN-fabric-git.md`'s teardown section gains one line stating it.

### D3 — The prime pushes its records side only

Decision: when the acting worktree is the hub's prime (`fabricengine.PrimeName(l) == l.WorktreeName`), fabric hands the code side to no push.
`lyx fabric push` pushes the records side only, and every detached push child fabric spawns gets an empty code-side path, so it pushes the records side only.
That covers the `sync` verb's own spawn and the spawn `Fabric.Commit` fires after any landed commit, so `lyx fabric commit`, `lyx fabric sync`, `lyx config`'s post-edit `sync` and every other `Commit` caller leave the code branch alone.
`PushPairAnchored` and the `sync` verb decide from the `Location` they already hold, and `Fabric.Commit` from the one it resolves from its warp path, so no route is handed a flag from outside.
In every other worktree, task pairs included, both sides push as today.
When the prime's code branch has unpushed commits, `push`'s envelope carries a `code_push_skipped` field naming the branch and saying it was left for the operator; `sync`'s envelope records no `push_spawned` entry for the code side.
A prime-name resolution failure fails the call before it commits or pushes anything: `push` and `sync` return the error, and `Fabric.Commit` returns it before taking any lock, like its other pre-flight refusals, so no commit lands and no push spawns.
Rationale: in the prime, `fabric push`/`sync` never make a code-side commit (`fab.Commit` commits the records side), so every unpushed parent-branch commit is one the call did not make; the hub publishes `main` only after its own full `go test ./...`.
This matches `PATTERN-push-both-sides`, whose per-transition prime push is already records-only.
Bound: the change removes a push, never adds one; it skips only the prime's code side, on every route above, and landing's own push of the parent branch is untouched.

### D4 — Neutral names: warp becomes "code", weft becomes "records"

Decision: every `vocabExempt` spelling that is an identifier, env var, result key or trace detail is renamed, using the code/records vocabulary fabric's prose already uses ("code side", "records side", "records worktree").
Mapping, with the Plan free to adjust a name that collides:

| Old | New |
| --- | --- |
| `hubforge.Hub.PairWarpWorktree` | `PairCodeWorktree` |
| `hubforge.Hub.PairWeftSibling` | `PairRecordsSibling` |
| `hubforge.Hub.WeftBare`, `WarpBare` | `RecordsBare`, `CodeBare` |
| `hubforge.Hub.PrimeWeft` | `PrimeRecords` |
| `fabricengine.WeftWorktree`, `WeftWorktreePath` | `RecordsWorktree`, `RecordsWorktreePath` |
| `fabricengine.WeftBranchName`, `WeftRepoRoot` | `RecordsBranchName`, `RecordsRepoRoot` |
| `fabricengine.CommitWeftPaths`, `WarpLyxLink` | `CommitRecordsPaths`, `CodeLyxLink` |
| `Fabric.ResetPairWarp` | `ResetPairCode` |
| result field `WarpWorktree` (Prune, Reconcile, Status) | `CodeWorktree` |
| `WEFT_SKIP_GIT`, `WEFT_SKIP_PUSH` | `FABRIC_SKIP_GIT`, `FABRIC_SKIP_PUSH` |
| `remove` result keys `warp_branch_deleted`, `warp_branch_kept_reason`, `remote_warp_branch_deleted`, `remote_warp_branch_kept_reason` | `code_branch_deleted`, `code_branch_kept_reason`, `remote_code_branch_deleted`, `remote_code_branch_kept_reason` |
| trace detail `side=warp`, `side=weft` | `side=code`, `side=records` |
| `RequireWarpWorktree`'s refusal wording "weft sibling" | "records sibling" |

The JSON tags on the renamed result fields follow the new names.
The renames are hard cuts: no alias, no fallback read of the old env var, key or trace detail.
`contracts/stencils/shed/shed-template-driver.md`'s stranded-branch rule reads `side=code`.
Bound: the hard cut of `WEFT_SKIP_GIT` and `WEFT_SKIP_PUSH` can only let git and push work run in a process that still sets the old name; the repository's setters are tests, renamed in the same commit, and the skip variables are a test and sandbox switch with no production consumer.
Rationale: the board entry asks for the vocabulary to stay inside fabric and the exemption set to shrink to nothing; there is no versioning (`CLAUDE.md`), so a compatibility layer has no consumer to serve.

### D5 — Spellings that are not names move or are reached through fabric

Decision:

- The file names `warpprobe.go`, `warpbinding.go`, `weftgit.go` in `internal/fabricengine` are renamed to neutral names (`git mv`), so the `cmd/lyx` guard tests that key on them carry neutral paths.
- The destructive guard's banned tokens `warp.ResetHard(` and `weft.ResetHard(` (`cmd/lyx/destructiveguard_test.go`) are replaced by a side-neutral form that still fails on a raw `gitrepo.Repo.ResetHard` call through a `Fabric` handle field in any fabricengine file but `destroy.go`; renaming the `Fabric` struct's `warp`/`weft` fields to `code`/`records` is one acceptable way.
- The retired spellings `lyx weft sync`, `lyx warp checkout`, `lyx.exe weft push` stay recognised by `RefScanner`; their cases move from `internal/websterengine/audit_test.go` into `internal/fabricengine/refscanner_test.go`, and webster's audit test keeps a current spelling for the hard-error classification.
- The `.weft` lock directory keeps its on-disk name; the tests outside the owner set that spell it (`internal/hubgeom`) reach the path through an exported fabricengine helper with a neutral name.

Rationale: each of these is either fabric-internal (a file name, a handle field) or data the owner set must keep (a retired spelling, an on-disk name); moving the spelling into the owner set keeps behaviour and on-disk state unchanged.

### D6 — The exemption mechanism is deleted

Decision: `vocabExempt`, `vocabExemptEntry`, `vocabExemptPattern` and the stripping step in `fabricVocabularyFailures` are removed, with any `vocabscan_test.go` fixture that exercises stripping.
`pattern/PATTERN-fabric-vocabulary.md` drops its `vocabExempt` bullet and states that a non-owner file carries no side-named spelling, owner-set exports included.
`vocabScanAllowlist` keeps its current entries and gains none.
Rationale: an empty closed list kept with its machinery is dead code and an invitation to refill it.

## Constraints

- `PATTERN-fabric-git`: all git runs through `internal/fabricengine`; the retry helper uses `gitexec` (`PATTERN-gitexec-checked-call`).
- `PATTERN-fabric-destruction-chokepoint`: the new branch-ownership proof and every deletion live in `destroy.go`; minting the proof follows the `createdToken{` guard.
- `PATTERN-push-both-sides`: D3 narrows the code-side push of `fabric push`/`sync` to non-prime worktrees; the PATTERN line is updated in the same commit to say the prime's `fabric push`/`sync` are records-only too.
- `PATTERN-fabric-vocabulary`: D4–D6 change its background file, see D6.
- `PATTERN-mutation-record`: the rollback's remote deletion records through `deleteRemoteBranch`'s existing success-only entry, and a push retry adds no entry of its own, so the one accepted push records `branch_pushed` as before.
  The pattern's `mutations` and `partial` keys are untouched; D3's `code_push_skipped` and D4's renamed result fields are verb-specific fields outside that set, and D4's `side=` rename changes a `detail` value inside the `mutations` entries, which the fixed key set does not constrain.
- `PATTERN-no-denied-recovery`: no new message names `git push --force`, `git reset --hard` or `rm -rf`.
- `PATTERN-spawn-observability`: each retry attempt is logged.
- `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-hubforge-fixtures`: retry sleeps are injectable; remote-backed tests are `integration`-tagged, build hubs through `hubforge`, and run under `gitkit.HermeticGitEnv()`.
- `PATTERN-comment-line-breaks`: every touched Go comment uses semantic line breaks.
- `PATTERN-markdown-link-integrity`: renamed files and identifiers referenced from `docs/` keep their links resolving.

## Auto-mode assumptions

- The task runs without an operator; every choice in this record is an auto-pick.
- A bare `(failed)` is transient; the board entry states it, and nothing in #433 contradicts it.
- The prime is identified by `PrimeName`, the predicate batten's refusal already uses.
- "code" and "records" are the neutral side names; fabric's prose already uses them.
- A hard cut of the env vars, result keys and trace detail is acceptable because nothing outside this repository consumes them live; the one in-repository reader of persisted data is the driver stencil's trace rule, see Open risks.

## Open risks

- `batcher-v2` is running in parallel and touches `tools/tokencount`, which resolves the records repo through fabric; the D4 renames may conflict with its branch at landing, resolved as a mechanical rename.
- A run whose failed step was traced by the old binary carries `side=warp`; a driver reading it after deploy does not recognise the stranded warp branch.
  The driver's stranded-branch rule escalates on a `side` it does not recognise, so an operator removes the branch, the same remedy as for the branches Out names that earlier binaries left behind.
- GitHub may answer a sustained outage with `(failed)` too; D1's bound caps the delay at about 10 seconds before `Add` fails and rolls back.

## Acceptance criteria

- An `Add` whose weft push is refused with a bare `(failed)` and then accepted succeeds, with the pair fully created.
- An `Add` whose weft push keeps failing with `(failed)` stops after the attempt bound, rolls back, and leaves under the default empty `branch_prefix` no warp branch locally or on origin; a re-add of the same slug then succeeds.
- A push whose stderr holds `remote:` lines and a bare `(failed)` line for the pushed ref is retried.
- A push refused for any other reason (hook declined, non-fast-forward) is attempted once, and so is one whose stderr holds `remote:` lines with a non-`failed` reason.
- An `Add` whose step 11 push keeps failing with `(failed)` stops after the attempt bound and rolls back; the origin deletion is refused and logged when nothing landed, and deletes the warp branch when the ref landed despite the refusals.
- A rollback whose warp branch was adopted from origin deletes only the local copy; origin's branch is untouched.
- A rollback of an `Add` whose step 11 pushed forward a pre-existing origin branch at or behind `HEAD` deletes only the local copy; origin's branch is untouched.
- A rollback whose warp branch moved on origin after step 11's push leaves the remote branch and logs the refusal.
- A rollback of an `Add` run under `SkipGit` or `SkipPush`, whose probe did not run, deletes only the local branch; origin's branch is untouched.
- In the prime with an unpushed local `main` commit, `lyx fabric push`, `lyx fabric commit`, `lyx fabric sync` and `lyx config --set` push the records side only and leave origin's `main` unchanged; `fabric push` reports `code_push_skipped`, and `sync`'s envelope has no code-side `push_spawned` entry.
- A prime-name resolution failure makes `lyx fabric push`, `sync` and `Fabric.Commit` fail before anything is committed or pushed, and `Commit` spawns no push.
- In a task pair, `lyx fabric push`, `commit` and `sync` still push both sides.
- `vocabExempt` and its stripping code are gone and `TestEnforcement_FabricVocabulary` passes; a non-owner file spelling a side-named identifier still fails the scan.
- The destructive guard still fails on a raw `ResetHard` through a `Fabric` handle field outside `destroy.go`.
- `RefScanner` still classifies each retired spelling as a fabric reference.
- `FABRIC_SKIP_GIT` and `FABRIC_SKIP_PUSH` take effect wherever the old variables did.
- `lyx fabric remove`'s envelope carries the renamed `code_branch_*` keys, and the driver stencil's stranded-branch rule matches the `side=code` trace detail.
- The Prune, Reconcile and Status results carry `code_worktree` where they carried `warp_worktree`, and `RequireWarpWorktree`'s refusal says "records sibling".
- `internal/hubgeom`'s tests reach the records worktree's lock directory through the neutral fabricengine helper and still assert it exists.
- `go test ./...` passes, with the `integration` tag for the remote-backed cases.

## Notes for the plan writer

- `add.go`'s file header, `rollbackAdd`'s doc and the logger line at its step 5 describe the left-behind branch as expected; they change with D2, as do `add_rollback_adopt_test.go`, `destructivegaps_integration_test.go` and `livestate_verbs_test.go`, which assert it.
- `pushRebaseFreeRetryingOnce` in `pushanchored.go` is the existing retry shape; D1's helper is separate because `Add` pushes with `-u origin <branch>` through `gitexec`, not through `gitrepo.PushRebaseFree`.
- A bare `(failed)` is hard to provoke from a local bare remote; the classifier and the retry loop can be pinned with an injected push seam, and the rollback with a hook-declined weft push.
- `rollbackAdd`'s callers discard its return, so its logging is the only visible trace of a refusal.
