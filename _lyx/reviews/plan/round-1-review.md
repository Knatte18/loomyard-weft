---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/01-add-push-retry.md:8
    summary: Cards 1 and 2 add top-level test functions to internal/fabricengine, whose budget row in cmd/lyx/testdata/test-budget.yaml equals its current count, and no card targets that file, so the budget test fails under the card gate and verify
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-fabricengine-neutral-exports.md:3
    summary: Cards 4 and 5 rename existing functions and methods that quarry indexes as glyphs, yet they are typed Edit with an old/new table and a hundred-file enumeration instead of a Rename group
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-neutral-result-keys-and-trace.md:46
    summary: Card 7 says every refDetail and recordPushIfAdvanced call site passes code or records, but its Edit list omits spawn.go (two recordPushIfAdvanced calls with warp) and mutation.go (the side=warp|weft docs)
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-rollback-created-warp-branch.md:52
    summary: Card 2 targets add_rollback_adopt_test.go but names only a new adopted case, leaving the two existing tests there that pin the left-behind branch and the gate-refusal log line, and the flipped tests overlap its new file
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/02-rollback-created-warp-branch.md:51
    summary: The SkipPush coverage row describes a failing weft step, but under SkipPush the weft push returns early and nothing follows step 11, so the row cannot be built as written
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-prime-records-only-push.md:35
    summary: Card 3 Coverage pins only the prime arm of the sync verb and names no test for the lyx config route or for the sync verb in a task pair, though the acceptance criteria require both sides to still push there
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/04-hubforge-neutral-names.md:3
    summary: Edit lists of Cards 4 and 5 are computed from today's tree and omit the test files Cards 1 and 2 create, which need the bare-repo paths Hub.WarpBare and Hub.WeftBare that Card 4 renames
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/10-delete-vocab-exemption.md:16
    summary: Card 10 turns the exempt-identifier fixture row into an owner-set export fails row that repeats the existing non-exempt identifier fails row, where D6 removes fixtures that exercise stripping
  - id: F9
    severity: NIT
    class: scope
    location: _lyx/plan/00-overview.md:27
    summary: Three constraints of the decision record are named by no card or Shared Decision, namely comment line breaks, markdown link integrity and no denied recovery
---

### [BLOCKING] Test budget for `internal/fabricengine` is not raised

**Location:** `_lyx/plan/01-add-push-retry.md:8` and `_lyx/plan/02-rollback-created-warp-branch.md:6`.

**Issue:** `PATTERN-test-speed` pins a per-package budget in `cmd/lyx/testdata/test-budget.yaml`, counted over every build tag, and `TestTestBudget_PackagesStayWithinBudget` in `cmd/lyx` fails when a package exceeds its row.
The file says to raise a row in the commit that adds the tests that need it.
`internal/fabricengine` has 622 top-level tests, counted with the budget test's own rule (a parser pass over `*_test.go`, `TestMain` excluded), and its row is `internal/fabricengine: 622`, so there is no headroom.
Card 1 creates `pushretry_test.go` (a classifier table and a retry-loop table) and `add_pushretry_integration_test.go`, and Card 2 creates `add_rollback_branch_integration_test.go`.
Each adds at least one top-level test function.
No card targets `cmd/lyx/testdata/test-budget.yaml`, so Card 1's commit fails the whole-suite tier-1 gate and the plan's `go test ./...` in `## verify:`.
This breaches a listed constraint (`PATTERN-test-speed`).

**Fix:** Add `//cmd/lyx/testdata/test-budget.yaml` to the `Edit` list of Card 1 and of Card 2, and have each card's `Intent` raise the `internal/fabricengine` row by the number of top-level test functions that card adds (or fold the cases into existing tests per `PATTERN-test-economy` so the count does not grow).
Mention the file in each card's `ImpactSummary`.

### [MEDIUM] Symbol renames typed `Edit` instead of `Rename`

**Location:** `_lyx/plan/05-fabricengine-neutral-exports.md:3`, `_lyx/plan/04-hubforge-neutral-names.md:3`.

**Issue:** This settles the focus question once, as a question about the method.
The spec's type table gives `Rename` to "existing symbol(s), `old -> new` pairs" with an AST-aware script plus a grep verify, and gives `Edit` to a change of an existing symbol.
Cards 4 and 5 describe exactly the `Rename` operation ("an AST-aware rename over the whole module, every build tag included, never a text replace; ... a grep pass") but are typed `Edit` and carry a 94-entry and a 110-entry file enumeration "that spells an old name today".
`lyx quarry glyphs --text` indexes the renamed names as glyphs: `internal/hubforge#Hub.PrimeWeft`, `Hub.PairWarpWorktree`, `Hub.PairWeftSibling`, `internal/fabricengine#WeftWorktree`, `WeftWorktreePath`, `WeftBranchName`, `WeftRepoRoot`, `CommitWeftPaths`, `WarpLyxLink` and `Fabric.ResetPairWarp`.
So Card 5 is a pure `Rename` card, and Card 4 is a `Rename` of three methods plus two struct fields (`Hub.WarpBare`, `Hub.WeftBare`), which quarry does not index (the `type Hub` span covers them) and which therefore stay an `Edit`.
Typing them `Edit` drops the rename-specific mechanical check the type exists for, and it pins an enumerated consumer list that goes stale as soon as another branch lands (the record's own open risk names `batcher-v2` touching `tools/tokencount`).
The same reading leaves Cards 7 and 8 as they are: their targets are struct fields, JSON tags, env-var strings, trace details and handle fields, none of which quarry indexes, so `Edit` is the faithful type there.

**Fix:** Give Card 5 a `**Rename:**` group of seven pairs, `old glyph -> plan:<unit>#<new name>` (for example `internal/fabricengine#WeftWorktree` -> `plan:internal/fabricengine#RecordsWorktree`, `internal/fabricengine#Fabric.ResetPairWarp` -> `plan:internal/fabricengine#Fabric.ResetPairCode`), and drop the file-by-file `Edit` enumeration, keeping the `Prosa` group for `docs/shared-libs/lyxcwd.md`.
Give Card 4 a `Rename` group for the three methods and keep an `Edit` group only for the files that spell the two fields.
The plan already carries `## Rename mechanic`, so no overview change is needed; keep `ImpactSummary` on any card that retains an `Edit` group.

### [MEDIUM] Card 7's footprint misses `spawn.go` and `mutation.go`

**Location:** `_lyx/plan/07-neutral-result-keys-and-trace.md:46` (the `Edit` list at line 3).

**Issue:** The `Intent` says "Every `refDetail` and `recordPushIfAdvanced` call site passes the side as `code` or `records`".
`internal/fabricengine/spawn.go` calls `recordPushIfAdvanced(rec, repo, "warp", ...)` twice (`PushWarpAt`, `PushWarpRebaseFreeAt`), and `PushWarpRebaseFreeAt` is the production route of `Fabric.PushBranch`.
Neither `spawn.go` nor `mutation.go` is in Card 7's `Edit` list (the overview's other cards that touch `spawn.go` do so for env vars and file names only).
`mutation.go` carries the two `Kind` docs reading "`side=<warp|weft> repo=...`" and `refDetail`'s doc "side is "warp" or "weft"".
A fork following the declared footprint leaves `side=warp` emitted for a code-branch push while `branch_created` entries read `side=code`, which is the half-cut the driver stencil's stranded-branch rule (it accepts one `side` value) cannot read.

**Fix:** Add `internal/fabricengine/spawn.go` and `internal/fabricengine/mutation.go` to Card 7's `Edit` list, and name the two `Kind` docs and `refDetail`'s doc in the `Intent`.

### [MEDIUM] Card 2 leaves existing tests that pin the old rollback behaviour unaccounted

**Location:** `_lyx/plan/02-rollback-created-warp-branch.md:52`.

**Issue:** Card 2 lists `add_rollback_adopt_test.go` as an `Edit` target and its Coverage names one new case there.
That file also holds `TestAddRollback_WarpBranchLeftBehindUnderEmptyPrefix`, which asserts the bare-slug warp branch is left behind, and `TestAddRollback_RefusedWarpBranchDeletionLogsWarn`, which asserts the log line "rollbackAdd's warp-branch deletion was refused by the destructive gate" and the word `ownership`.
After D2 both fail, because the branch is deleted and no ownership refusal occurs for a branch this `Add` created.
The Coverage flips assertions only in `destructivegaps_integration_test.go` and `livestate_verbs_test.go`.
In `destructivegaps_integration_test.go` the flipped `TestBranchOwnership_RefusalHoldsAtOtherDeletionSites` also keeps a name and doc that describe a refusal which no longer happens.
The plan's new file then re-asserts what these flipped tests assert (the local warp branch is gone under the empty `branch_prefix`; a `SkipPush` rollback deletes only the local branch), which `PATTERN-test-economy` rules out for a new test.

**Fix:** State in Card 2 what happens to each of the three tests.
Flip `LeftBehindUnderEmptyPrefix` into the local-deletion-under-empty-prefix case and drop that half from the new file's first case.
Turn `RefusedWarpBranchDeletionLogsWarn` into the assertion of the lease-refused origin deletion log (the "moved on origin" case), so the logged refusal the acceptance criteria require is actually asserted.
Retitle or redirect `RefusalHoldsAtOtherDeletionSites`.

### [MEDIUM] The `SkipPush` coverage row cannot be built as written

**Location:** `_lyx/plan/02-rollback-created-warp-branch.md:51`.

**Issue:** The row reads "an `Add` under `SkipPush` whose weft step fails deletes only the local branch".
Under `SkipPush`, `pushWeftBranch` returns before pushing, the archived-leftover replacement needs the probe (which `SkipPush` skips), and `Add` returns right after step 12, so no weft step can fail.
The only post-creation failure under `SkipPush` that leaves origin untouched in a meaningful way is step 11's own push, and the acceptance criterion ("origin's branch is untouched") is only non-trivial when that push lands and then reports a failure.
The existing `TestBranchOwnership_RefusalHoldsAtOtherDeletionSites` already covers step 11 failing against a broken remote under `SkipPush`, where nothing lands on origin.

**Fix:** Reword the row to the case Card 1's seam makes possible: a step-11 runner that pushes for real and then reports an error, under `SkipPush`, leaves origin's branch and deletes the local one.
Fold the broken-remote variant into the existing test rather than a second case.

### [MEDIUM] Card 3 Coverage does not pin the pair arm of the `sync` verb or the `lyx config` route

**Location:** `_lyx/plan/03-prime-records-only-push.md:35`.

**Issue:** D3's route list is `PushPairAnchored`, `Fabric.Commit`, the `sync` verb and `lyx config`, and its acceptance criteria require that in a task pair `lyx fabric push`, `commit` and `sync` still push both sides.
The `sync` verb decides on its own (it calls `spawnPush` with both paths today) and the only existing CLI test of it, in `weftskippush_integration_test.go`, runs in the prime.
The Coverage names the prime case alone, so a change that makes `sync` skip the code side everywhere passes every named test; `PushPairAnchored` and `Fabric.Commit` do get a pair case.
`lyx config` reaches the change through `fabriccli.RunCLIIn(cwd, w, []string{"sync"})` (`internal/configcli/configcli.go:502`), but the `Intent` says nothing of that, and no test is named for the route.

**Fix:** Add a task-pair row to the `sync` scenario asserting two `push_spawned` entries, and state in the `Intent` that `lyx config`'s post-edit `sync` is the in-process `sync` verb, so the `sync` tests pin it and no separate `lyx config` test is added.

### [LOW] Edit lists of Cards 4 and 5 omit files that earlier cards create

**Location:** `_lyx/plan/04-hubforge-neutral-names.md:3`, `_lyx/plan/05-fabricengine-neutral-exports.md:3`.

**Issue:** Both lists are "every file that spells an old name today", and they are complete against the current tree.
Card 2's Coverage needs a pre-receive hook on the weft bare and on the warp bare and a check of origin, which only `Hub.WeftBare` and `Hub.WarpBare` expose, and Card 1's integration test checks branches "on origin" the same way.
The files those cards create (`add_rollback_branch_integration_test.go`, `add_pushretry_integration_test.go`) are therefore likely to spell names Card 4 renames, and they exist when Card 4 runs; they are in neither list.
The module-wide AST rename and the build catch a missed one, so this is footprint accuracy rather than a break.

**Fix:** Name those files in the affected `Edit` lists (their `path-missing` is satisfied by the earlier `Create` groups), or let the `Rename` groups of F2 carry the method renames and list only the field-bearing files.

### [LOW] Card 10 keeps a fixture row that repeats an existing one

**Location:** `_lyx/plan/10-delete-vocab-exemption.md:16`.

**Issue:** D6 removes "any `vocabscan_test.go` fixture that exercises stripping".
`TestFabricVocabularyFailures_Fixtures` has the row "exempt identifier alone passes" (`h.PairWarpWorktree`) and the row "non-exempt identifier fails" (`FooWeftBar`, line 38).
Once stripping is gone, converting the first into "owner-set export outside the owner set fails" makes a second row that pins the same behaviour as the second: a non-owner test file with a side-named identifier fails.
`PATTERN-test-economy` rules a row that covers nothing new out.

**Fix:** Delete the "exempt identifier alone passes" row and rename "non-exempt identifier fails" to a name that no longer refers to an exemption; keep the real-tree test as the coverage for the owner-set exports.

### [NIT] Three record constraints are carried by no card

**Location:** `_lyx/plan/00-overview.md:27`.

**Issue:** The `## Shared Decisions` section restates the test-tier constraints and the vocabulary, git and mutation-record constraints, but none of `PATTERN-comment-line-breaks`, `PATTERN-markdown-link-integrity` and `PATTERN-no-denied-recovery` appears anywhere in the plan.
The plan rewrites many Go comments and several docs, and adds user-visible messages (`code_push_skipped`, the rollback log lines).

**Fix:** Add one `## Shared Decisions` bullet naming the three, so the executing forks are told about them.
