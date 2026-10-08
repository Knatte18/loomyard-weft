---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/fabricengine/add.go:387
    summary: two new Go comments break a line at a comma that joins a compound object, not independent clauses, which PATTERN-comment-line-breaks forbids
  - id: F2
    severity: MEDIUM
    class: consistency
    location: internal/fabricengine/pushanchored.go:132
    summary: PushPairAnchored doc says a failing side is named warp or weft, but the error now names the side code or records
  - id: F3
    severity: LOW
    class: scope
    location: internal/fabricengine/doc.go:4
    summary: package doc still names the Fabric handle's unexported fields warp and weft, which Card 8 renamed to code and records
  - id: F4
    severity: LOW
    class: scope
    location: internal/fabricengine/branchname_test.go:11
    summary: old exported function names survive in test names and a helper whose docs name the new function (TestWeftBranchName, TestCommitWeftPaths_*, mustWeftRepoRoot)
  - id: F5
    severity: LOW
    class: consistency
    location: cmd/lyx/testdata/test-budget.yaml:29
    summary: the internal/fabricengine budget row sits one above the package's top-level test count, where Cards 1 and 2 say to raise it to the count
  - id: F6
    severity: LOW
    class: scope
    location: internal/fabricengine/destroy.go:746
    summary: the new gate behaviours (ownedCreatedBranch refusing a foreign name or empty proof, dirtyPushedByThisCall refusing an empty lease or a local delete) have no test
---

Range: `main` is `product.parent`, and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (ten commits, one per card).
Run against HEAD: `go build ./...`, `go vet` under the default, `integration`, `tmux` and `llm` tags, `go test ./...`, `go test -tags integration` over the packages the diff touches, and `lyx loom lint-comments --base dc30e1a98` all pass.
Review-time checks on the focus items:

- Card 1 `pushretry.go` is correct on every point the focus names.
  The classifier reads only the `! [remote rejected] <ref> -> <ref> (failed)` line whose ref is the pushed branch.
  Every other failure returns the first error unwrapped.
  The push is never forced, both push sites use `t.push`, and `pushWeftBranch` is the only other caller.
  The only `branch_pushed` entry is recorded after the push succeeds.
- Card 2: `createdBranchToken` is minted only in `createGitWorktree`, and the guard bans `createdBranchToken{` outside `destroy.go`.
  The gate order is unchanged.
  `rollbackAdd` deletes the origin branch only for `warpOriginAbsent && pushAttempted`, with `leaseSHA` set to the tip read before the push.
  Every statement the card names (`battencli`, `landingshed`, `docs/overview.md`, `doc.go`, `PATTERN-fabric-git.md`) is updated.
  The reworked tests flip the old assertions.
- Card 3: `IsPrimeWorktree` is `PrimeName(l) == l.WorktreeName`.
  It is resolved before the lock in `PushPairAnchored`, before any lock in `Commit`, and before the commit in the `sync` verb.
  A resolution failure returns before anything commits or pushes.
- Cards 4 to 10: a grep over every tracked file outside `docs/benchmarks/`, `docs/research/` and `_lyx` finds no `WEFT_SKIP`, `warp_worktree`, `weft_worktree`, `warp_branch_*`, `side=warp`, `side=weft` or `vocabExempt`.
  It also finds no old `hubforge` accessor, no `weftgit.go`/`warpprobe.go`/`warpbinding.go` reference, and no `warp.ResetHard(`/`weft.ResetHard(` token.
  `RequireWarpWorktree` says "records sibling".

### [BLOCKING] Two new comments break a sentence at a comma that is not an independent-clause boundary

**Location:** `internal/fabricengine/add.go:387-388` (the `rollbackAdd` doc) and `internal/fabricengine/destroy.go:1410-1411` (the `createGitWorktree` doc).
**Issue:** `PATTERN-comment-line-breaks` allows a break inside a sentence only after a semicolon or after a comma before a coordinating conjunction whose clause has its own subject and verb, and "a break at any other comma (a list item, a compound predicate …) is a violation".
In both comments the second line continues a compound object:
- `rollbackAdd` doc: "warp carries the proof that this Add created the warp branch, which deletes the local branch … from origin," / "and what step 11 did about origin."
- `createGitWorktree` doc: "returns the createdToken proving the gate itself added it there," / "and the createdBranchToken proving the `-b` of buildArgs created createdBranch."

`lyx loom lint-comments` passes both because it lets every comma-plus-conjunction break through; the PATTERN names these as review findings.
The `add.go` sentence is also hard to read: "which deletes the local branch" has `warp` as its subject, but the proof does not delete anything.
**Fix:** In `createGitWorktree`, join the two lines into one line, or end the first at "added it there;" if the sentence is recast with a verb in the second part.
In `rollbackAdd`, recast the sentence so the break falls at a sentence end, for example "warp carries the proof that this Add created the warp branch, which allows deleting the local branch whether it was forked or adopted from origin." followed by a new sentence "It also records what step 11 did about origin."

### [MEDIUM] `PushPairAnchored`'s doc names the sides by their old spelling

**Location:** `internal/fabricengine/pushanchored.go:132`
**Issue:** The doc reads "A failing side's error is wrapped as `fabricengine: push <side> side at <path>: %w` with the side named warp or weft".
Card 7 makes the error read `push code side at <path>` or `push records side at <path>`, and the code does this (`pushSide("code", …)`, `pushSide("records", …)`).
The doc therefore states the wrong values for a contract callers match on.
`pushanchored.go` is in Card 7's Edit list and the same commit touched the function body, so the doc was reachable; fabricengine is in the owner set, so the vocabulary scan does not catch it.
**Fix:** Say "with the side named code or records".

### [LOW] Package doc still names the `Fabric` handle fields `warp` and `weft`

**Location:** `internal/fabricengine/doc.go:4-5`
**Issue:** The opening paragraph says the `Fabric` handle "holds unexported `warp *gitrepo.Repo` and `weft *gitrepo.Repo` fields".
Card 8 renamed them to `code` and `records` (`fabric.go:55-56`) and lists `doc.go` among its edits, but this sentence was left unchanged.
**Fix:** Name the fields `code` and `records`.

### [LOW] Old function names survive in test names, a test helper and their docs

**Location:** `internal/fabricengine/branchname_test.go:11-39` (`TestWeftBranchName`, `TestWeftBranchName_RoundTripsWithWeftWarpSlug`), `internal/fabricengine/commitweftpaths_test.go:59-83` (`TestCommitWeftPaths_SkipGit`, `TestCommitWeftPaths_EmptyPaths`), `internal/fabricengine/add_rollback_adopt_test.go:34-38` (`mustWeftRepoRoot`), `internal/fabricengine/resetpairwarp_integration_test.go` (`TestResetPairWarp_*`), `internal/fabricengine/warpjunction_test.go:24` (`TestWarpLyxLinkMethods`).
**Issue:** Card 5 requires a grep pass confirming that no old function name survives in code, comments or docs, and keeps only file names (`commitweftpaths.go`, `resetpairwarp_integration_test.go`).
The AST rename does not touch test names, so these still spell `WeftBranchName`, `CommitWeftPaths`, `WeftRepoRoot`, `ResetPairWarp` and `WarpLyxLink`.
Several now contradict their own docs: `commitweftpaths_test.go:59` says "`TestCommitWeftPaths_SkipGit` asserts `CommitRecordsPaths` returns …", and `mustWeftRepoRoot`'s doc says it "resolves `fabricengine.RecordsRepoRoot(l)`".
**Fix:** Rename the tests and the helper after the new function names (`TestRecordsBranchName`, `TestCommitRecordsPaths_*`, `mustRecordsRepoRoot`, `TestResetPairCode_*`, `TestCodeLyxLinkMethods`) and update the `//testtiming:keep` and doc lines that cite them.
The file names stay, as the card says.

### [LOW] The `internal/fabricengine` budget row is one above the package's test count

**Location:** `cmd/lyx/testdata/test-budget.yaml:29`
**Issue:** The base commit's row (622) equalled the package's top-level test count.
Card 1 adds three top-level tests (two in `pushretry_test.go`, one in `add_pushretry_integration_test.go`) and raises the row to 626; Card 2 nets +1 and raises it to 627.
`PATTERN-test-speed` counts excluding `TestMain` give 625 after Card 1 and 626 now, so a test can be added with no visible raise.
Both cards say to raise the row "to the package's top-level test count after the card".
**Fix:** Set the row to 626, the count at HEAD.

### [LOW] The new gate kinds' own refusals are not tested

**Location:** `internal/fabricengine/destroy.go:746` (`branchOwnershipCreatedBranch` in `resolveBranchOwnership`), `destroy.go:971` and `destroy.go:1112` (`branchDirtinessPushedByThisCall` in `checkBranchDirtiness` and `checkRemoteBranchRequest`)
**Issue:** The diff adds three refusals no test reaches: `ownedCreatedBranch` refusing a branch other than the proof's or an empty proof; `dirtyPushedByThisCall` refused for a remote delete with an empty `leaseSHA`; and the same kind refused for a local delete.
The integration cases in `add_rollback_branch_integration_test.go` only reach the accepting path and git's own lease refusal.
Sibling kinds have this coverage: `TestDeleteArchivedWeftBranch_EmptyLeaseRefused` in `destroyremote_integration_test.go` pins the empty-lease refusal, and `TestBranchOwnership_ManagedBranchKind` pins a kind's ownership refusals.
The chokepoint's value is that these refusals hold, and they are what keeps the new proof from deleting a branch it does not cover.
**Fix:** Add rows to `TestBranchOwnership_ManagedBranchKind`'s style of table and to the executor-level empty-lease test (through the existing `…ForTest` helpers) for the three refusals, as rows of existing tests rather than new top-level functions.
