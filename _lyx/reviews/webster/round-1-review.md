---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/fabricengine/add_rollback_adopt_test.go:317
    summary: Three broken-origin rollback tests still call Add with zero AddOptions, so the new pre-flight ls-remote fails before any mutation and they pass without exercising rollback
  - id: F2
    severity: LOW
    location: internal/fabricengine/remoteleftover.go:149
    summary: probeWarpLeftover builds its ErrRemoteLeftover without Slug, breaking the exported field's documented contract
  - id: F3
    severity: LOW
    location: internal/fabricengine/add_remoteleftover_integration_test.go:354
    summary: The lease-race test asserts only that origin's weft branch still exists, not that it holds the tip the hook pushed
  - id: F4
    severity: LOW
    location: internal/fabricengine/destroy.go:257
    summary: New and changed comments in destroy.go wrap mid-sentence at a fixed column, against the Comment Line-Break Convention
  - id: F5
    severity: LOW
    location: internal/fabricengine/remoteleftover.go:2
    summary: The remoteleftover.go file header wraps mid-sentence at a fixed column
  - id: F6
    severity: LOW
    location: internal/gitrepo/push.go:135
    summary: The DeleteRemoteBranchLeased doc comment wraps mid-sentence at a fixed column
  - id: F7
    severity: LOW
    location: internal/fabricengine/export_test.go:156
    summary: New test-file comments wrap mid-sentence at a fixed column (export_test.go, destroy_test.go, destroyremote_integration_test.go, deleteremotebranch_integration_test.go, addleftover_integration_test.go)
  - id: F8
    severity: NIT
    location: internal/fabricengine/add.go:155
    summary: The pre-flight probe comment breaks after the conjunction so, and the file-header addition breaks at a colon before a non-clause fragment
  - id: F9
    severity: NIT
    location: internal/fabricengine/mutation_record_integration_test.go:98
    summary: The three tests switched to SkipPush carry no comment saying why, although their comments still attribute the failure to a push SkipPush would seem to skip
  - id: F10
    severity: NIT
    location: internal/fabricengine/destroyremote_integration_test.go:8
    summary: The header still says building either case needs a real hub, but the file now has seven cases
---

Review range: `git diff $(git merge-base main HEAD)..HEAD` (merge base `676d9ec03`), from `product.parent: main` in `_lyx/shed/fabric-readd-weft-push/status.json`: Cards 1–5 plus `fd2c3d3ec`.

Substrate: `go build ./...` and `go vet -tags integration` over the three packages pass.
`go test -tags integration` passes for `internal/gitrepo` and `internal/fabricengine`.
`internal/fabriccli` fails `TestRunCLI_CleanupRemoteFailureExitsNonZero`; it fails identically on an export of the merge base, so it predates this diff and is not a finding here.
I checked the archive-probe fetch by hand against a bare repo holding an annotated `archive/x/abc` tag: `git fetch --no-tags --refmap= origin refs/heads/x-weft refs/tags/archive/x/abc` left `for-each-ref` unchanged and wrote both only to `FETCH_HEAD`, and `ls-remote` emits the `^{}` peeled line that `remoteArchiveTags` prefers.

Per-card mechanical checks: every `Create` symbol exists (`DeleteRemoteBranchLeased`, `ownedPairWeftBranch`, `dirtyArchivedOnRemote`, `ErrRemoteLeftover`, `weftLeftover`, `probeWeftLeftover`, `remoteleftover.go`, both new test files), and `DeleteRemoteBranchLeased` is pinned in `cmd/lyx/gitrepoboundary_test.go`.
The plan has no `Rename` or `Delete` group.
Card 3's `Edit` of `Topology.Add` fails its impact check: F1.

Focus verdicts not raised as findings:
step 12's refusal and failed-lease branches both roll back through `rollbackAdd`, and `deleteRemoteBranch`'s leased branch appends `remote_branch_deleted` only after `DeleteRemoteBranchLeased` returns nil, so the record orders it before the weft `branch_pushed` only on an observed deletion.
`isAncestorOrEqual` maps any `*gitexec.GitError` from `cat-file -e` to "not an ancestor", which is what Card 3 specifies, and fails in the refusing direction; a spawn failure is still returned.
`Error()`'s remedies match Card 3's list for all three kinds.
The `push.go` file-header edit changes two lines of a legacy fixed-column paragraph; rewrapping it would touch untouched lines, which the convention bars, so it is not a finding.

### [MEDIUM] Three rollback tests pass vacuously after the pre-flight

**Location:** `internal/fabricengine/add_rollback_adopt_test.go:317` (`TestAddRollback_UnwiresJunctionsOnPostWiringFailure`), `internal/fabricengine/add_rollback_adopt_test.go:376` (`TestAdd_GitFailureCarriesGitsOwnReason`), `internal/fabricengine/origin_integration_test.go:238` (`TestAddRollback_CreatedPathLeavesNoOriginRecord`)
**Issue:** Each test breaks the warp origin with `git remote set-url origin <missing>` and then calls `Add` with `AddOptions{}`, intending the step-11 push to fail after worktrees and junctions exist, so that `rollbackAdd` runs.
Card 3's pre-flight now runs `probeWarpLeftover` first, whose `ls-remote origin` fails on the broken URL before any mutation.
`Add` still returns an error, so every assertion that something is gone holds vacuously; nothing was ever created.
`TestAddRollback_UnwiresJunctionsOnPostWiringFailure`'s own doc comment says that without a post-wiring failure "this test would prove nothing".
The diff switched the three sibling tests to `SkipPush` (`mutation_record_integration_test.go`, `origin_integration_test.go`'s adopted-path test, `livestate_verbs_test.go`) but missed these three.
**Fix:** Pass `AddOptions{SkipPush: true}` in all three, so the probes are skipped and step 11's ungated warp push fails as each test intends, and say why in a comment.

### [LOW] probeWarpLeftover omits Slug from its refusal

**Location:** `internal/fabricengine/remoteleftover.go:149`
**Issue:** `ErrRemoteLeftover.Slug` is documented as "the slug Add was asked to create", but `probeWarpLeftover` never receives the slug and returns `&ErrRemoteLeftover{Branch: warpBranch, …}`, leaving `Slug` empty.
The warp `Error()` does not print it, but any caller reading the exported field gets an empty string on the warp kind only.
**Fix:** Give `probeWarpLeftover` a `slug` parameter, set `Slug` on its refusal, and assert it in `TestAdd_WarpLeftoverRefusedAtPreflight`.

### [LOW] Lease-race test does not assert origin kept the moved tip

**Location:** `internal/fabricengine/add_remoteleftover_integration_test.go:354`
**Issue:** Card 5 requires that "origin keeps the moved tip".
The test only checks that `weftBareTip` is non-empty, which a branch reset to any other commit would also satisfy.
**Fix:** Record the bare's tip inside the hook after `pushCommitToOrigin`, and assert the post-Add tip equals it.

### [LOW] destroy.go comments wrap at a fixed column

**Location:** `internal/fabricengine/destroy.go:257` (`leaseSHA`), `:371` (`warpBranch` field), `:387` (`ownedPairWeftBranch`), `:464` (`archiveTag`), `:469` (`dirtyArchivedOnRemote`), `:665` (`resolvePairWeftBranch`)
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line in new and changed comments, with further breaks only at an independent-clause boundary.
These comments break mid-clause at a fixed column (for example "succeeds only while / the remote branch", "is / accepted by", "the pre-flight proved / covers").
The `leaseSHA` comment also puts two sentences on one line.
**Fix:** Rewrap each comment one sentence per line, breaking only at an independent-clause boundary.

### [LOW] remoteleftover.go header wraps at a fixed column

**Location:** `internal/fabricengine/remoteleftover.go:2`
**Issue:** "proven replaceable or refused / before Add's first mutation" and "writes / FETCH_HEAD only" break mid-clause.
**Fix:** Rewrap one sentence per line, breaking at the `, so` and `;` clause boundaries.

### [LOW] DeleteRemoteBranchLeased doc wraps at a fixed column

**Location:** `internal/gitrepo/push.go:135`
**Issue:** "via / `git push …`, succeeding only while the / remote branch" breaks mid-clause in a new doc comment.
**Fix:** Rewrap one sentence per line.

### [LOW] New test-file comments wrap at a fixed column

**Location:** `internal/fabricengine/export_test.go:156` (`DeleteArchivedWeftBranchForTest`), `internal/fabricengine/destroy_test.go:659`, `internal/fabricengine/destroyremote_integration_test.go:84`, `:102`, `:170`, `internal/gitrepo/deleteremotebranch_integration_test.go:135`, `:166`, `:189`, `:229`, `:247`, `internal/fabriccli/addleftover_integration_test.go:3`, `:24`
**Issue:** Each new comment breaks mid-sentence at a fixed column, and the `destroy_test.go` comment puts two sentences on one line.
**Fix:** Rewrap each one sentence per line.

### [NIT] add.go comment breaks

**Location:** `internal/fabricengine/add.go:155`, `internal/fabricengine/add.go:9`
**Issue:** The in-function comment breaks after "so" rather than before it, leaving the conjunction dangling at the end of a line.
The file-header addition breaks at a colon before "proven replaceable or refused…", which is not an independent clause.
**Fix:** Break before ", so an unreplaceable one…"; join the header sentence onto one line.

### [NIT] SkipPush switches carry no explanation

**Location:** `internal/fabricengine/mutation_record_integration_test.go:98`, `internal/fabricengine/origin_integration_test.go:283`, `internal/fabricengine/livestate_verbs_test.go:505`
**Issue:** Each test's comment attributes the expected failure to the push, but the call now passes `SkipPush: true`.
A reader cannot tell that `SkipPush` is there only to skip the pre-flight probes, or that step 11's warp push ignores it.
**Fix:** Add a one-line comment at each call explaining this.

### [NIT] destroyremote header says "either case"

**Location:** `internal/fabricengine/destroyremote_integration_test.go:8`
**Issue:** "building either case needs a real hub" dates from when the file had two cases; it now has seven.
**Fix:** Say "every case".
