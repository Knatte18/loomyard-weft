---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/06-neutral-env-and-refusal.md:34
    summary: Card 6 claims no other test asserts the old refusal wording, but internal/fabriccli/cleanhub_integration_test.go:542 asserts "weft sibling of a pair" and is not a Card 6 target, so the integration tier fails at Card 6
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/03-prime-records-only-push.md:37
    summary: Card 3 never states the fate of the prime step PushPushesBothSides in hubmutation_integration_test.go, which asserts the prime's code side is pushed and fails under D3
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-rollback-created-warp-branch.md:43
    summary: Cards 1 and 2 leave statements that D1 and D2 falsify or make incomplete in internal/fabricengine/doc.go, internal/battencli/wire.go, internal/landingshed/publish.go and docs/overview.md
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/00-overview.md:33
    summary: Cards 4 and 5 type quarry-indexed method and function renames as Edit over about a hundred files, on a blanket claim that holds for only some of the declaring files
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-prime-records-only-push.md:39
    summary: Card 3's Coverage adds prime, pair and resolution-failure cases to internal/fabricengine tests without naming the budget row or saying they are rows of existing top-level tests, and that row has no headroom
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/03-prime-records-only-push.md:39
    summary: Card 3's "prime-name resolution failure" rows name no way to make PrimeName fail after the Location has resolved
  - id: F7
    severity: NIT
    class: scope
    location: _lyx/plan/05-fabricengine-neutral-exports.md:104
    summary: Card 5 lists internal/ideengine/spawn_hub_integration_test.go, which spells only the result-struct fields Card 7 renames
  - id: F8
    severity: NIT
    class: scope
    location: _lyx/plan/10-delete-vocab-exemption.md:13
    summary: Card 10 does not name the doc comment of bareVocabularyTokenIn, whose second sentence describes the stripping it removes
---

### [BLOCKING] Card 6 misses a test that asserts the reworded refusal

**Location:** `_lyx/plan/06-neutral-env-and-refusal.md:34`.
**Issue:** Card 6 rewords `RequireWarpWorktree`'s refusal from "weft sibling" to "records sibling" and says "a grep for the old phrase finds no other assertion" beyond `fabric_test.go` and `battencli`'s `lifecycle_integration_test.go`.
The grep finds another: `internal/fabriccli/cleanhub_integration_test.go:542` calls `envelope.RequireErr(t, output, "weft sibling of a pair")`, and `internal/fabricengine/fabric.go:138` is the only place that text is produced.
The file is not among Card 6's Edit targets, so `go test -tags integration ./...` fails at Card 6, and the plan template's rule 4 ("a card that changes a user-visible message greps the repository for the old text, and every test asserting it joins that card's targets") is broken.
The card also names `fabric_test.go` as asserting the wording, but that file asserts only `errors.Is(err, ErrNotAWarpWorktree)`.
`tools/sandbox/SANDBOX-FABRIC-SUITE.md:542` instructs the sandbox agent to confirm a refusal "naming the weft sibling", the same text.
**Fix:** Add `internal/fabriccli/cleanhub_integration_test.go` to Card 6's Edit list, with its assertion moved to "records sibling of a pair".
Drop `fabric_test.go` from the sentence naming wording assertions (it stays a target for `FABRIC_SKIP_*`).
Add `tools/sandbox/SANDBOX-FABRIC-SUITE.md` as a Prosa target, reworded to "records sibling".

### [BLOCKING] Card 3 leaves an existing prime test that pins the old behaviour

**Location:** `_lyx/plan/03-prime-records-only-push.md:37`.
**Issue:** `internal/fabriccli/hubmutation_integration_test.go` carries two prime steps that run `lyx fabric push` from `h.PrimeWorktree()`.
Card 3 reworks only `PushReportsADivergedWarpSideAndStillPushesWeft`.
`PushPushesBothSides` (line 74) commits on both prime sides, pushes from the prime and asserts `h.WarpBare`'s HEAD equals the local code HEAD.
Under D3 the prime pushes the records side only, so that assertion fails and the card's own verify goes red.
Reworking both steps into prime cases would also cover one behaviour twice (`PATTERN-test-economy`), and the plan has no CLI-level case that a task pair's `push` still pushes both sides (an acceptance criterion).
**Fix:** State in Card 3's Coverage that `PushPushesBothSides` becomes the task-pair case: the step adds a pair, runs `push` from the pair's code worktree and asserts both bares reach their local HEADs.
`PushReportsADivergedWarpSideAndStillPushesWeft` stays the single prime case, as written.

### [MEDIUM] Docs and messages that D1 and D2 falsify are not in any card

**Location:** `_lyx/plan/02-rollback-created-warp-branch.md:43`, `_lyx/plan/01-add-push-retry.md:33`.
**Issue:** The decision record's scope puts `internal/fabricengine/doc.go` first in the docs landing with the code, and D2 ends "a rolled-back create keeps the branch it made".
Four statements say so or describe the old rule, and no card edits their files:
- `internal/fabricengine/doc.go:514`: "A rollback deletes only the local branches this `Add` created ... and never restores the deleted remote branch" is false once the rollback deletes the warp branch on origin.
- `internal/battencli/wire.go:223` and `:232`: the comment and the user-visible refusal both say "a rolled-back create keeps the branch it made".
- `internal/landingshed/publish.go:103`: the comment justifies a Stuck verdict by "Add's rollback keeps the branch it made".
- `docs/overview.md:473`: "a rolled-back create keeps the branch it made".

D1 adds a retrying push helper to `Add`, and Card 1 touches no doc (`CLAUDE.md`: docs land in the same commit as new behaviour).
**Fix:** Add `internal/fabricengine/doc.go` to Card 2 (the rollback sentence states the origin exception and its bound) and to Card 1 (one sentence on the bounded retry of a bare `(failed)` push).
Add `internal/battencli/wire.go` and `internal/landingshed/publish.go` to Card 2's Edit list, rewording the comment and message so that only a torn-down pair, or a create rolled back by an older binary, leaves a branch.
Add `docs/overview.md` to Card 2's Prosa list with the same rewording.
`internal/battencli/wire_test.go` and `lifecycle_integration_test.go` assert none of the removed clause.

### [MEDIUM] Quarry-indexed renames are typed Edit on a claim that is only partly true

**Location:** `_lyx/plan/00-overview.md:33`.
**Issue:** Shared Decisions says Cards 4, 5, 7 and 8 type their renames `Edit` because "the renamed symbols sit in files that other cards target whole" and the gate blocks the overlap.
`lyx quarry glyphs` indexes `Hub.PrimeWeft`, `Hub.PairWarpWorktree`, `Hub.PairWeftSibling` and every function Card 5 renames; it does not index struct fields (`Hub.WarpBare`, `PairStatus.WarpWorktree`, `Fabric.warp`), JSON tags or string literals, so Cards 7 and 8 are correctly `Edit`.
For Cards 4 and 5 the claim holds only where another card targets the declaring file whole: `weftwiring.go` (Cards 1, 7), `fabric.go` (Cards 6, 8) and `destroy.go` (Cards 2, 8).
`WeftRepoRoot` (`worktreelist.go`), `CommitWeftPaths` (`commitweftpaths.go`), `WarpLyxLink` (`junction.go`), `WeftBranchName` (`branchname.go`) and the three hubforge methods have no such overlap.
Typed `Edit`, they forgo the Rename grammar (`old glyph -> plan:` handle pairs, the doc-comment opening fix, the binary build check) and make Cards 4 and 5 hand-edit about a hundred caller files that an AST rename updates itself.
**Fix:** Type the non-overlapping renames `Rename`, with the old glyph copied from `lyx quarry glyphs` and a `plan:<unit>#<new name>` handle as the new side, beside an `Edit` group for what quarry does not index.
Callers a Rename updates leave the Edit list.
Reword the Shared Decision to name the declaring files that really overlap, and keep those renames `Edit`.

### [MEDIUM] Card 3's new tests have no budget plan

**Location:** `_lyx/plan/03-prime-records-only-push.md:39`.
**Issue:** `cmd/lyx/testdata/test-budget.yaml` pins `internal/fabricengine` at exactly its top-level test count, and Cards 1 and 2 each raise the row to their own count.
Card 3 adds a prime case and a resolution-failure case to `pushanchored_integration_test.go`, and a pair row and a resolution-failure case to `commit_integration_test.go`, in a package with no headroom.
It does not target the budget file or say the cases are subtests of existing top-level tests, and the Shared Decision requires a card that adds top-level functions to raise the row.
`internal/fabriccli` is also at its budget; its `hubmutation` and `weftskippush` cases are steps and `t.Run` rows of existing tests, so that package is safe.
**Fix:** State in Card 3 that the fabricengine cases are `t.Run` rows inside `TestCommit_InvokesPushRecorder` and `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`, adding no top-level function.

### [LOW] The prime-resolution failure rows do not say how a test reaches them

**Location:** `_lyx/plan/03-prime-records-only-push.md:39`.
**Issue:** `PrimeName` fails only when `git worktree list` fails in `l.AnchorPath()`, after `lyxcwd.ResolveWorktree` has already succeeded.
The code path is reachable, but the rows name no mechanism: for `PushPairAnchored` a hand-built `Location` whose `AnchorRel` names a directory absent from the worktree, for `Fabric.Commit` a recorded anchor naming the same.
**Fix:** Name that mechanism in the two rows, so the writer of the tests does not have to find it.

### [NIT] Card 5 lists a file Card 7 owns

**Location:** `_lyx/plan/05-fabricengine-neutral-exports.md:104`.
**Issue:** `internal/ideengine/spawn_hub_integration_test.go` spells only `e.WarpWorktree` and `e.WeftWorktree`, the result fields Card 7 renames, and no function Card 5 renames; Card 5's own Intent says those fields are not its part.
**Fix:** Remove the file from Card 5's Edit list; Card 7 already carries it.

### [NIT] Card 10 omits one comment the deletion falsifies

**Location:** `_lyx/plan/10-delete-vocab-exemption.md:13`.
**Issue:** The doc comment of `bareVocabularyTokenIn` ends "Both patterns only strip spellings that themselves contain weft or warp", which stops being true when the exemption pattern goes; Card 10 names the comments of `bareVocabularyToken` and `TestEnforcement_FabricVocabulary` only.
**Fix:** Add `bareVocabularyTokenIn`'s doc comment to the list of comments Card 10 rewrites.

## Focus outcomes

- Budget file: Cards 1 and 2 target it and raise the row to their own count, with Card 2 counting the test it deletes; Cards 4 to 10 add no top-level test; Card 3 is the gap (F5).
- Rename typing: Cards 7 and 8 are correctly `Edit` (fields, tags, strings); Cards 4 and 5 are partly mistyped (F4).
- Edit footprints: grepping the tree for every old spelling finds Cards 4, 5, 7 and 8 complete apart from F7's over-listing; Card 6 misses one assertion and one sandbox line (F1); the D1 and D2 behaviour changes miss four statements (F3).
- Existing tests and Coverage: Card 2's reworked and deleted tests exist as described and its rows are reachable; Card 3 misses `PushPushesBothSides` (F2); Card 9's moved rows already exist in `TestRefScanner_Matches` except `lyx.exe weft push`, as the card says.
- The plan passes `lyx loom validate-plan` as it stands, so none of these findings is a mechanical-gate duplicate.
