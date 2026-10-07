---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/03-prime-records-only-push.md
    summary: Card 3 resolves the prime from the anchor path and so breaks four existing tests whose code worktree lacks the anchor subdirectory, and the card names none of them
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/03-prime-records-only-push.md
    summary: Card 3 leaves comments and Short strings that say push, sync and commit push both sides outside its docs line, and coalesce.go is not in its Edit list at all
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/02-rollback-created-warp-branch.md
    summary: Card 2 deletes TestBranchOwnership_RefusalHoldsAtOtherDeletionSites but does not name the doc comment in livestate_verbs_test.go that cites it by name
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/02-rollback-created-warp-branch.md
    summary: Card 2 adds a third token-carrying ownership kind but its docs list does not name the doc.go paragraph that describes exactly two
---

### [BLOCKING] Card 3 breaks four existing tests that the card does not fix

**Location:** `_lyx/plan/03-prime-records-only-push.md` (Intent bullets 1–3; Coverage list)

**Issue:**
Card 3 has `Fabric.Commit`, `PushPairAnchored` and the `sync` verb call `PrimeName(l)`, and Card 3's own Coverage section states that `PrimeName` "fails when `git worktree list` fails in `l.AnchorPath()`".
`l.AnchorPath()` is `<worktree>/<AnchorRel>`, so any `Commit` call whose resolved `AnchorRel` names a subdirectory that does not exist in the code worktree now fails with `resolve main worktree: ... chdir ...: no such file or directory`.
Card 3 uses that exact condition to build its failure row, yet two existing tests build the same condition and expect success:

- `internal/fabricengine/commit_integration_test.go` `TestCommit_NestedRelPath_ClassifiesWeftFileUnderRelPath` records anchor `wts/some-task` in the board marker but creates that directory only on the records side (`<weft>/wts/some-task/_lyx`), never in the plain code fixture.
  The file is in Card 3's Edit list, but the card lists only `TestCommit_InvokesPushRecorder` as changed.
- `internal/webstercli/sync_integration_test.go` `TestFabricSync_CommitsAtEveryRelPathDepth` rows `one segment`, `two segments` and `three segments` use `newPairFixtureAt`, which creates the anchor subdirectory only under the records repo, and `fabricSync` runs `Fabric.Commit`.
  The file is in the Edit lists of Cards 4–6 but not Card 3's.

I confirmed this on the real substrate: in a scratch copy of the tree I added a `PrimeName(l)` call to `Fabric.Commit` directly after its `lyxcwd.ResolveWorktree`, as Card 3 prescribes, and ran `go test -tags integration` over the affected packages.
Exactly these four tests fail, all with `resolve main worktree: list git worktrees in "<code worktree>/<anchor>": chdir ...: no such file or directory`.
No other test in fabricengine, fabriccli, webstercli, configcli, loomcli, battencli, hubgeom, landingshed, boardengine, preflight, shedcli or orchcli changed outcome.
Card 3 would therefore fail its own gate, and `## verify:` (`go test ./...` and `go test -tags integration ./...`) would fail on it.

**Fix:**
Add `internal/webstercli/sync_integration_test.go` to Card 3's Edit list.
State in the card that `newPairFixtureAt` creates `<code>/<relPath>` in the code worktree, and that `TestCommit_NestedRelPath_ClassifiesWeftFileUnderRelPath` creates `<warpPath>/wts/some-task` before calling `Commit`.
Leave the prime resolution on `PrimeName(l)` as the decision record prescribes.

### [LOW] Card 3 leaves both-sides statements untouched

**Location:** `_lyx/plan/03-prime-records-only-push.md` (Docs line); `internal/fabricengine/coalesce.go:80`

**Issue:**
Card 3's Docs line names the `push` and `sync` long help, `doc.go`, `PATTERN.md` and `docs/overview.md`.
Statements it also falsifies are not named:

- `internal/fabricengine/coalesce.go` says the detached push child "always supplies both paths" and cites `Fabric.Commit`'s `spawnDetachedPushFn(f.warpPath, f.weftPath)` call.
  In the prime the child now gets only the records path.
  `coalesce.go` is not in Card 3's Edit list, and Card 7 touches it only for the `side=` detail.
- `internal/fabriccli/weft_verbs.go` has a file header saying "push and sync both push the warp side and the weft side", and `Short` strings "commit weft changes and push both sides" and "commit weft changes and async-push both sides".
  The file is in Card 3's Edit list, but only the long help is named.
- `internal/fabricengine/commit.go`'s `Commit` doc says it "fires async both-sides push".
  The file is in Card 3's Edit list, but the doc is not named.

**Fix:**
Add `internal/fabricengine/coalesce.go` to Card 3's Edit list.
Extend the Docs line to cover the `coalesce.go` sentence, the `weft_verbs.go` header and `Short` strings, and the `commit.go` `Commit` doc.

### [LOW] Card 2 leaves a dangling reference to the test it deletes

**Location:** `_lyx/plan/02-rollback-created-warp-branch.md` (the `livestate_verbs_test.go` bullet); `internal/fabricengine/livestate_verbs_test.go:412`

**Issue:**
Card 2 deletes `TestBranchOwnership_RefusalHoldsAtOtherDeletionSites`.
The doc comment of `addCase` in `livestate_verbs_test.go` names it ("the same injection TestBranchOwnership_RefusalHoldsAtOtherDeletionSites already uses").
The card's bullet for that file covers only the flipped `Add` expectation and the empty-prefix comment, so the reference to a deleted test survives.

**Fix:**
Extend the `livestate_verbs_test.go` bullet to say the `addCase` doc comment drops the reference and cites the surviving test that uses the same broken-origin injection.

### [LOW] Card 2's docs list misses the token-carrying ownership paragraph

**Location:** `_lyx/plan/02-rollback-created-warp-branch.md` (Docs paragraph); `internal/fabricengine/doc.go:918`

**Issue:**
`doc.go` has a paragraph headed "Why the two token-carrying ownership kinds exist" that covers `ownedFreshlyCreatedPath` and `ownedFreshlyCreatedWorktree`.
Card 2 adds a created-branch proof and a `branchOwnershipKind` that accepts it, which is a third token-carrying kind, and the paragraph's "two" and its account of what backs the proofs become stale.
Card 2 edits `doc.go` but names only the sentence about the rollback's local branches.

**Fix:**
Name that paragraph in Card 2's docs obligations and say it covers the branch proof, including that the proof type's unforgeability rests on the same `destructiveguard_test.go` composite-literal ban.
Derive the count rather than restating "two".

## Focus verdicts

1. **Card 6 and the old refusal phrase.**
   Not a defect.
   A repository-wide grep, `tools/` and `contracts/` included, finds three places that assert or instruct the exact wording: `internal/battencli/lifecycle_integration_test.go` ("weft sibling"), `internal/fabriccli/cleanhub_integration_test.go` ("weft sibling of a pair") and `tools/sandbox/SANDBOX-FABRIC-SUITE.md`.
   Card 6 targets all three, and `fabric_test.go` asserts only `ErrNotAWarpWorktree`.
   Every other hit is descriptive prose or a code comment, not the refusal text.
   Card 6's `WEFT_SKIP_` list also matches the repository grep, apart from the `lyxcwd` exemption entries that Card 10 deletes.
2. **Card 3 and tests that push from the prime.**
   The two fabriccli steps and the `weftskippush` rows are handled as the card describes, and `internal/fabriccli` passes in the scratch simulation.
   A CLI-level task-pair case keeps `push` pushing both sides: the converted `PushPushesBothSides` step and the pair `sync` row.
   The remaining defect is the anchor-path breakage in F1.
   The loom per-transition tests use a pair fixture and are unaffected.
3. **Cards 4 and 5 renames.**
   Not a defect.
   `WeftBranchName`, `WeftRepoRoot`, `CommitWeftPaths` and `WarpLyxLink` are declared in `branchname.go`, `worktreelist.go`, `commitweftpaths.go` and `junction.go`, each a target of Card 5 alone, so `Rename` is correct.
   `WeftWorktree`, `WeftWorktreePath`, `Fabric.ResetPairWarp` and the three `Hub` methods are declared in files that other cards target whole, so `Edit` is correct.
   `containment-file-overlap` compares a member glyph with other cards' file targets only, so a same-card `Rename` beside a whole-file `Edit` of its declaring file is fine.
   Edit-list completeness holds: a reference-aware `gopls` rename of the result fields and a compile-based check of the `Fabric` handle fields touch only files in Cards 7 and 8, and Card 4 covers every file spelling a `Hub` name.
4. **Cards 1 and 2 and statements of the old rule.**
   Every statement of the old rollback rule that I found is covered by Card 2 (`add.go` header and `rollbackAdd` doc, the step 5 log line, `doc.go`'s rollback sentence, `battencli/wire.go`, `landingshed/publish.go`, `docs/overview.md`, and the tests that assert the left-behind branch).
   The gaps are F3 and F4.
   Card 1's push-retry statements sit in `doc.go`'s `Add` section, which Card 1 edits.
5. **Card 3 and the test budget.**
   Not a defect.
   Card 3 adds only rows and steps, so no top-level test function, and `internal/fabricengine`'s row (622) equals its current count.
   Cards 1 and 2 add top-level functions and raise the row themselves.
