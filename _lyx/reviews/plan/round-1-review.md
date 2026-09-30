---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/02-fabric-add-drops-run-records.md
    summary: Card 2 puts the no-checkout fork and the drop inside createWeftWorktree, which Reconcile's createDormantWeftForRawWarp also calls; that caller never commits, so it would get a weft worktree with uncommitted run-record deletions, and the changed return type breaks reconcile.go, which the card does not target
  - id: F2
    severity: LOW
    location: _lyx/plan/02-fabric-add-drops-run-records.md
    summary: The git sequence leaves every index entry with no stat data (checkout-index without -u, and a final read-tree HEAD without -m), unlike the full checkout it replaces
  - id: F3
    severity: LOW
    location: _lyx/plan/03-battenshed-prime-lock-wait.md
    summary: Card 3's Intent edits the deps.go file header, which none of its targets cover, and the header needs no change anyway
---

### [BLOCKING] Card 2 changes a shared helper that Reconcile also calls

**Location:** `_lyx/plan/02-fabric-add-drops-run-records.md`, Mechanism steps 1–3; the code is `internal/fabricengine/weftwiring.go:117` (`createWeftWorktree`) and `internal/fabricengine/reconcile.go:541` (`createDormantWeftForRawWarp`).

**Issue:**
Card 2 adds the no-checkout fork, the drop helper and a new "root was tracked" return value to `createWeftWorktree`.
`createWeftWorktree` has a second caller, `createDormantWeftForRawWarp` in `reconcile.go`, which creates a dormant weft for a raw warp worktree and makes no commit.
With the card as written, three things go wrong on that path:
- The dormant weft worktree ends up with the parent's run records tracked but missing on disk, and nothing ever commits those deletions. The worktree stays dirty (`D _lyx/shed/...` under `git status`), so `dirtyScopeAll` treats it as dirty and a later non-force removal is refused.
- The decision record limits the drop to `Add` (Scope: "`fabricengine`'s `Add`"; Decision 1). Changing how Reconcile creates a weft is scope the decision record does not allow, and the ImpactSummary does not mention it.
- The changed signature stops `reconcile.go:541` compiling, and `reconcile.go` is not among the card's targets.

**Fix:**
Keep `createWeftWorktree` as it is for Reconcile, and give `Add` its own fork path: a `noCheckout`/drop switch that only `Add` sets, or a separate function that `Add` calls.
Either way, Reconcile's dormant weft forks exactly as it does today.
Also state in the card that Reconcile is left unchanged.

### [LOW] The no-checkout sequence leaves the index without stat data

**Location:** `_lyx/plan/02-fabric-add-drops-run-records.md`, Mechanism step 2.

**Issue:**
I ran the card's sequence in a scratch repo with git 2.53.
It writes file modes, symlinks and `eol` attribute conversion the same way a full checkout does, `StageAndCommit`'s `git add -- <root>` stages the deletions of the tracked-but-absent root, and the result is one commit.
The index state differs, though:
- `git checkout-index -a` without `-u` records no stat data in the index. On the path where nothing is tracked under the root, that is the index the pair is left with.
- A final `git read-tree HEAD` without `-m` throws away whatever stat data existed.

Until some porcelain command refreshes the index, `git diff-index HEAD` reports every file as modified (seven of seven in the scratch repo).
Nothing in production runs raw `diff-index` today, and the origin commit refreshes the index, so this is not a live bug.
It is still a difference from the "equivalent to today's checkout" property the card claims, and under `SkipGit` nothing refreshes the index at all.

**Fix:**
Use `git checkout-index -a -u` in both branches, and reset with `git read-tree -m HEAD` rather than `git read-tree HEAD`.
In the scratch repo this pair of changes leaves the index stat-clean everywhere except the dropped root, which shows as deleted.

### [LOW] Card 3's Intent edits a file header no target covers

**Location:** `_lyx/plan/03-battenshed-prime-lock-wait.md`, Intent ("Update both `Call` doc comments, `PrimeLock`'s doc comment (the new field), the `deps.go` header, and `doc.go` with the wait.").

**Issue:**
The card's `deps.go` targets are the symbol `internal/battenshed#PrimeLock` and no file target, so the `deps.go` header is outside what the card may change.
The two `Call` doc comments are fine, because a doc comment belongs to its symbol's target.
The `deps.go` header also needs no update: it lists the four seam types the file declares, and adding a `Sleep` field to `PrimeLock` leaves that list true.

**Fix:**
Remove "the `deps.go` header" from the doc-update sentence.

## Focus verdicts

- Card 2's git sequence and `CommitWeftPaths` handling: the sequence is correct and the deletions are staged in the single origin commit (confirmed empirically). The one gap is the stat data, raised as F2.
- Card 2 against the Constraints: the new raw git calls sit in `internal/fabricengine`, which is where the Fabric Git Invariant requires them. The commit still goes through `CommitWeftPaths` with a positive-only list. `git rm -r --cached` only touches the index; it is not on `cmd/lyx/destructiveguard_test.go`'s banned-token list, and none of the tokens on that list matches it. The `SkipGit` behaviour keeps the root off disk, which satisfies "no run records on disk", and it commits nothing, the same as the origin record today, so it does not conflict with Decision 5. Not a defect.
- Card 3's targets: the `deps.go` header is raised as F3. The `Call` doc comments are covered by their symbol targets.
- Card 3 against Decision 7: counting attempts waits at least `primeLockWaitBound` of poll sleeps before it gives up, so it honours "until a bound elapses", and "after waiting 10m0s" is a true lower bound. The existing `release on every exit path` subtests and `*_HappyPath` release assertions, together with the new contended-then-free cases, cover release in both producers. Not a defect.
- Card 4: it is the `battencli` package's pair of acceptance criteria, in a different package from card 2's `fabricengine` test, so it can be reviewed on its own. The fixture seams it relies on are all real: `wireForHub`, `seedEntryStatus`, `taskWorktreeLocation`, `shedrun.RecipeBatten`, `fabricengine.CommitAnchoredPaths` and the real `Env.InnerRun.ResolveStatus`/`ReadStatus`. Both tests would fail on today's code, because the child's `self` run directory is the inherited `_lyx/shed/B/`. Not a defect.
