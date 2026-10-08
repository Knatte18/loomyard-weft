---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/fabricengine/pushanchored_integration_test.go:250
    summary: the doc comment of the pair-walk push test lists its behaviors in order but omits the two prime subtests the diff added to it
  - id: F2
    severity: NIT
    class: consistency
    location: internal/fabricengine/pushanchored_integration_test.go:353
    summary: a test comment mixes the renamed and the old side word in one sentence, code side reported against the warp side
---

### [LOW] Pair-walk test doc does not mention the prime subtests

**Location:** `internal/fabricengine/pushanchored_integration_test.go:250`
**Issue:** The doc comment of `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` says the test "walks one pair through the behaviors of PushPairAnchored in order" and enumerates them.
Card 3 added two `t.Run` subtests to this test, `PrimePushesRecordsSideOnly` and `UnresolvablePrimePushesNothing`.
Each builds its own hub and never touches the pair, so the doc is now wrong about what the test walks, and a reader deciding whether the prime's records-only push is covered here cannot tell from the doc.
**Fix:** Extend the doc comment with the two prime behaviors (records-only push with `code_push_skipped`, and the unresolvable-prime refusal with nothing pushed), and say they run on their own hubs.

### [NIT] Mixed side words in one test comment

**Location:** `internal/fabricengine/pushanchored_integration_test.go:353`
**Issue:** The comment reads "A diverged code side: reported against the warp side, the records side still pushes."
The assertions below it were renamed to `push code side at …`, so the same sentence now names the one side twice, as `code` and as `warp`.
The `warp`/`weft` spellings are legal in the owner set, so this is wording, not a vocabulary breach.
**Fix:** Say "reported against the code side".

## Focus questions

1. **Comment line breaks (`add.go` `rollbackAdd` doc, `destroy.go` `createGitWorktree` doc, fix-touched comments).**
   Every break the diff creates in those docs falls at a sentence end, a semicolon or a comma before a conjunction whose clause has its own subject and verb.
   I read the F1 rewrap of both docs, the `createdBranchToken` and `dirtyPushedByThisCall` docs, and every added comment line ending in a comma across the whole range together with its continuation line.
   The comma-ending ones are independent clauses (`so`/`and` plus a subject), or breaks carried over unchanged from legacy wrapping (`doc.go` "being in the same call …,", the `warpforward.go` and `weftpaths_test.go` doc lines), which `PATTERN-comment-line-breaks` excludes.
   `lyx loom lint-comments --base dc30e1a98` reports no finding.
   Not a defect.
2. **Rename residue in `internal/fabricengine`.**
   No old function, field, env-var, JSON key, envelope key or side value survives: a repo-wide search for the old spellings, excluding `docs/benchmarks/` and `docs/research/`, finds none, and test names and helpers follow the renamed functions (`mustRecordsRepoRoot`, `TestRecordsBranchName`, `TestCommitRecordsPaths_*`, `TestResetPairCode_*`, `TestCodeLyxLinkMethods`).
   The `PushPairAnchored` doc and its error text agree on `code`/`records`, and `doc.go` names the code and records handle fields.
   The only leftover is the wording in F2.
3. **Test-budget row.**
   The `internal/fabricengine` row is 626.
   Counting top-level `func Test…` functions across the package's `_test.go` files, `TestMain` excluded, as `cmd/lyx/testbudget_test.go` counts them, gives 626.
   `go test ./cmd/lyx -run Budget` passes.
4. **Gate coverage.**
   `TestGate_ZeroValueDeclarationsAreRefusals` carries four new rows: `CreatedBranchOtherNameRefused` (foreign name), `CreatedBranchEmptyProofRefused` (empty proof), `RemoteBranchPushedByThisCallEmptyLeaseRefused` (empty `leaseSHA` on a remote delete) and `BranchRequestPushedByThisCallRefused` (local delete).
   Each asserts the specific check (`CheckOwnership` or `CheckDirtiness`), the rows reach the refusals without a git spawn, and they extend an existing top-level test, as `PATTERN-test-economy` asks.
   The accepted paths (created-branch deletion, leased remote deletion) are covered by `TestAddRollback_DeletesTheWarpBranchItCreated`.

## What I ran and read

- Range: no rework round exists under `_lyx/loom/rework/`, and `product.parent` is `main`, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
- `go build ./...`, `go vet` under the `integration`, `tmux` and `llm` tags, `go test ./...` and `go test -tags integration ./...` all pass.
- Per-card checks against the plan: Cards 1 and 2 touch exactly their listed files; Card 2's `createdBranchToken{` is banned in `cmd/lyx/destructiveguard_test.go`; Card 5's renamed symbols each open their doc with the new name; Card 8's three files moved by rename (git reports them as renames) with headers opening on the new name, and the guard keys and banned tokens follow; Card 9's lock-directory helper and refscanner rows are in place; Card 10 deleted `vocabExempt` and its stripping, and `TestEnforcement_FabricVocabulary` passes.
- Card 3 adds `IsPrimeWorktree` to `worktreelist.go`, a file its Edit list does not name.
  It is a three-line wrapper over the predicate the Shared Decisions name, used at the three prime sites, and inlining it would repeat the predicate, so I raise no finding.
