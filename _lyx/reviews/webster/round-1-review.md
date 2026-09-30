---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/battenshed/create.go:44
    summary: New and changed comments across the diff break mid-sentence at a column or share one line between two sentences, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F2
    severity: LOW
    location: internal/fabricengine/add_runrecords_integration_test.go:66
    summary: Every drop test runs at AnchorRel ".", so nothing pins that forkrecords.go's root pathspec and CommitWeftPaths' anchor join name the same tree in a subpath-anchored hub
  - id: F3
    severity: NIT
    location: internal/battencli/recordleak_integration_test.go:86
    summary: The Seed-Child test discards the Worktree-Create step's outcome, so a stuck create would surface as a misattributed Seed-Child failure
---

Review range: `product.parent` is `main` (from `_lyx/shed/batten-records-leak-into-pairs/status.json`); reviewed `git diff $(git merge-base main HEAD)..HEAD`, commits `fc1db0884`..`6b0ae7335`.

Evidence gathered: `go build ./...`, `go vet` on the touched packages, `go test ./...` and `go test -tags integration ./...` all pass.
With `add.go` reverted to its pre-card-2 content in a scratch copy, both card-4 batten tests and `TestAdd_FromTaskPairDropsParentRunRecords` fail with the reported seed collision, so the new tests do pin the fix.

### [LOW] Changed comments violate the Comment Line-Break Convention

**Location:** `internal/battenshed/create.go:44-49` (Call doc), `internal/battenshed/teardown.go:78-82` (Call doc), `internal/battenshed/deps.go:44-49` (PrimeLock doc), `internal/battenshed/primelockwait.go:1-2` (file header), `internal/shedrun/paths.go:126-127` (RunsRootRel doc), `internal/fabricengine/add.go:71-73` (Add doc) and its step-8 and step-10c comments, `internal/fabricengine/forkrecords.go:19-20` (function doc), and the new test-file comments in `internal/battenshed/create_test.go` (`waitingPrimeLock`), `internal/battencli/recordleak_integration_test.go` (header and every doc comment) and `internal/fabricengine/add_runrecords_integration_test.go` (header).

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line with further breaks only at independent-clause boundaries, and governs new and changed comments.
The changed lines above break mid-phrase at a column (`which is not a\n// producer verdict`, `the run-records root: the join of\n// lyxdirs.LyxDirName`, `a bound spent is\n// Stuck naming`), and several put two sentences on one line (`producer verdict. A context cancelled during the wait ...`, `(see acquirePrimeLock). The lock file this package acquires ...`).

**Fix:** Rewrite each changed sentence onto its own line, breaking only at an independent-clause boundary; leave untouched legacy lines alone except where a changed sentence shares their line.

### [LOW] No subpath-anchored coverage for the run-records drop

**Location:** `internal/fabricengine/add_runrecords_integration_test.go:66` (and every other `hubforge.NewHub(t, ".")` in the file).

**Issue:** The drop joins `l.AnchorRel` in two independent places: `createWeftWorktreeDroppingRuns` builds the `ls-files`/`rm --cached` pathspec through `ScopedPathspec` plus `filepath.ToSlash`, and `CommitWeftPaths` joins it again for `git add`.
Both resolve relative to the weft worktree and agree by construction; I confirmed it by running the drop, no-records and adopt tests in a scratch copy against a `"backend"`-anchored hub, and all three pass.
But every committed test uses `"."`, where the join is a no-op, so a later change to either join could diverge them unnoticed: the drop would then either miss the records or hit `git add`'s pathspec error at step 10c.

**Fix:** Add a subpath-anchored case (the `"backend"` anchor `reconcile_empty_anchor_integration_test.go` already uses) that seeds run records under the anchor and asserts the pair's weft branch tracks none of them and has no run-records root on disk.

### [NIT] Seed-Child test ignores the Worktree-Create outcome

**Location:** `internal/battencli/recordleak_integration_test.go:86`

**Issue:** `TestBattenIntegration_SeedChild_IgnoresPrimesCommittedBattenRecords` checks only the first `Step`'s error.
If `Worktree-Create` ended `Stuck`, the second `Step` would re-run `Worktree-Create`, and the failure would read `Seed-Child outcome = "stuck"` with the create's reason, naming the wrong row.

**Fix:** Assert the first step's outcome is `Done`, naming `Worktree-Create` in the failure message.

## Focus directives settled without a finding

- **No-checkout fork equivalence.**
  In a scratch repo carrying `.gitattributes` (`text=auto`, an `eol=crlf` path, a smudge/clean filter), a symlink and an executable file, the exact sequence (`worktree add --no-checkout`, `read-tree HEAD`, `rm -r --cached -q`, `checkout-index -a -u`, `read-tree -m HEAD`) left `git status --porcelain` and `git diff-files` showing only the dropped run records as ` D`.
  The CRLF conversion, filter output, symlink target and exec bit match a full `worktree add` checkout byte for byte.
  The one thing the plain checkout did that this skips is the `post-checkout` hook, and fabric installs that hook into the warp repo only, so the weft fork loses nothing.
- **Root pathspec vs. CommitWeftPaths path.**
  Both are `AnchorRel` joined with `shedrun.RunsRootRel()`, relative to the weft worktree, so they name the same tree; on Windows the commit side passes native separators as `OriginRecordRel` already does.
  The coverage gap is F2.
- **Rollback after the drop.**
  `rollbackAdd` calls `removeWeftWorktree` with `force` true, so the tracked-but-absent entries do not trip the dirtiness check, and the path goes through `destroy.go`'s gate as before.
  `TestAdd_DropRollsBackFully` seeds records on `main-weft` before `Add`, and the portal blocker fails at `createPortal`, right after step 8, so it does exercise the tracked-root path.
- **Bound arithmetic and cancellation.**
  `maxSleeps` is 300: 301 `Acquire` attempts and 300 sleeps of 2 s, which is the 10-minute bound; the test asserts exactly 300 sleeps.
  Cancellation is checked after every contended attempt and after every sleep, and an `Acquire` error defers to `cancelErr` first, as the old code did.
  Keying `Stuck` on `release == nil` agrees with the helper's documented outcomes; only an `Acquire` that violates its own contract by returning `(nil, true, nil)` would be misread, and the old code would have panicked on that same return.
- **PrimeLock.Sleep blast radius.**
  Every construction outside `battenshed` (`battencli/run_test.go`, `battenrecipe`, `shedbuild`, `shedrecipe` fixtures) always acquires, so none reaches the real `waitOrCancel`; `wire.go`'s production closure leaves `Sleep` nil as intended.
  `fakePrimeLock` sets a no-op `Sleep`, so the pre-existing contention tests do not wait.
- **Fabric Vocabulary Invariant.**
  No added line in `battenshed`, `shedrun` or `battencli` names warp or weft.
- **Per-card mechanical checks.**
  The Edit groups' blast radius holds: every `Add` caller's tests, unit and integration, pass, and no other `PrimeLock` construction changes behaviour.
  The Create groups duplicate nothing: `shedrun` had no root constructor, and no other file in the package uses `--no-checkout` or `checkout-index` for a worktree fork.
