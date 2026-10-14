# Card 6 — Every card gate ends with the root module's guard tests

**Create:**
- `internal/websterengine#guardTests.step`
- `internal/websterengine#guardNotice`

**Edit:**
- `internal/websterengine#cardGateCommand` -> `func cardGateCommand(plan *planparser.Plan, cards []planparser.Card, index int, worktree string, guards guardTests) (steps []string, guardTooLong int)`
- `internal/websterengine#renderCardGates` -> `func renderCardGates(plan *planparser.Plan, cards []planparser.Card, planDirDisplay, worktree, stencilsDir string, guards guardTests) (string, error)`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine#RenderForkPrompt`
- `internal/websterengine#RenderRecoveryPrompt`
- `internal/websterengine#TestCardGateCommand`
- `internal/websterengine#TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`
- `internal/websterengine_test#TestBeginBatch_Refusals`
- `internal/websterengine_test#TestBeginBatch_PromptFile`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/websterengine#guardTests`
- `internal/websterengine#scanGuardTests`
- `internal/impactset#MaxCommandLength`
- `internal/planparser#RefCovers`
- `contracts/stencils#WebsterFragmentGuardScanError`
- `contracts/stencils#WebsterFragmentGuardTooLong`
- `contracts/stencils#WebsterFragmentGuardFix`
- `internal/websterengine#packageArg`
- `internal/websterengine#writeGuardFixture`
- `internal/websterengine_test#newBeginFixture`
- `internal/websterengine_test#newRecoverFixture`
- `internal/websterengine_test#writeWorktreeFile`

**Intent:**
Decisions D2 and D3: every card gate ends with one step running the root module's guard tests, and a scan the render cannot use becomes a prompt line instead.

`guardTests.step(plan, cards, index)`, in `internal/websterengine/guardstep.go`, returns the guard step of `cards[index]`, or "" when it renders none, and the step's length:
`lyx gate test --tags integration <guard packages> -- -run '^(<guard test names>)$'`, the packages spelled through `packageArg` in sorted directory order and the names sorted and each listed once, joined by `|`.
A guard directory drops out for `cards[index]` when the cards from `cards[0]` through `cards[index]` have emptied it, which is what keeps the step from naming a directory gone by that card's gate:
start from the directory's `goFiles`, and for each of those cards in order remove every file a cover of the card covers (`planparser.RefCovers` with the file's path as the target, the covers being the card's Rename old sides and Delete targets), then add each `.go` file the card's Create targets and Rename new sides name in that directory (`planparser.RefFile`);
the directory drops when no file is left.
A directory-level cover removes every file under it, a file-level one removes that file, and a member-level one removes none, so a card moving one file out of a guard package keeps the package for every card.
The step is "" when `scanErr` is set or no directory is left.

`cardGateCommand(plan, cards, index, worktree, guards)` returns the steps of `cards[index]`: its own `lyx gate test` steps over `gateScopeOf(plan, cards[index:index+1], worktree)` as today, then the guard step, then `lyx loom lint-comments` and `lyx loom lint-pattern`.
A guard step longer than `impactset.MaxCommandLength` is left out, and its length is returned as `guardTooLong`, 0 otherwise.
Update its doc comment and every call in `TestCardGateCommand` to the new signature, passing a zero `guardTests` where a row has no guard.

`renderCardGates(plan, cards, planDirDisplay, worktree, stencilsDir, guards)` renders the bullets as today, passing each card's index, then appends, after a blank line, `guardNotice`'s text when it is not empty.
`guardNotice` reads its fragments through `stencilstore` from `stencilsDir`, strips their banner as `readFragment` does and fills them with `stencil.Fill`:
when `scanErr` is set, `webster-fragment-guard-scan-error` with the error's text, followed, when `scanErr.File`'s directory is a `planparser.CardTargetDirs` directory of a card in `cards`, by `webster-fragment-guard-fix` with that file and the first such card's pointer, spelled as `renderCardGates` spells it;
when `tooLong` is above 0, the largest length any card's step had, `webster-fragment-guard-too-long` with that length and `impactset.MaxCommandLength`;
"" otherwise, so a render with no fallback reads no fragment.

`BeginBatch` and `recoverSpawn` call `scanGuardTests(deps.Geom.WorktreeRoot)` before their first mutation, and pass `deps.Geom.StencilsDir` and the result to `renderCardGates`:
in `BeginBatch` right before the reports-dir `os.MkdirAll`, after the entry refusals; in `recoverSpawn` right before its reports-dir `os.MkdirAll`, so before `archiveStaleReport` and before a prior strand is stopped.
A scan error is then a transient refusal that mutates nothing, as `refusal-spec.md`'s transient class requires: begin-batch's message ends `; way forward: transient, re-run \`lyx webster begin-batch <N>\` once the path reads`, recover-batch's `; way forward: transient, re-run \`lyx webster recover-batch <N>\` once the path reads`, each after the error naming the path.
A `GuardScanError` or an over-long step never refuses: the render goes on without a guard step for the affected cards.
recover-batch renders the same guard step as begin-batch, since both go through `cardGateCommand`.
The doc comments of `RenderForkPrompt` and `RenderRecoveryPrompt` say `cardGates` is the per-card gate step list followed by any guard notice.

Tests:

- `TestCardGateCommand` gains a `guard` subtest table over fixture trees written with `writeGuardFixture`, holding a root `go.mod`, scanned with `scanGuardTests`, each row naming the batch's cards and the index and asserting the exact step list and returned length:
  a card's steps end with one guard step naming every guard package and every guard name, after its own step and before the two lints, with guards in two packages and the root;
  in the drop rows `internal/g` holds exactly `g.go` and `g_test.go`, the latter with a marked test, and it is the only guard package;
  a three-card batch whose second card renames `internal/g/g.go#` and `internal/g/g_test.go#` out to `internal/h`: card 1's step names `./internal/g`, cards 2 and 3 render none;
  the same batch whose second card renames only `internal/g/g.go#` out: every card keeps `./internal/g`;
  a second card deleting the directory `internal/g`: cards 2 and 3 lose it;
  a second card renaming both files out and creating `internal/g/new.go#`: every card keeps `./internal/g`, the re-add branch;
  a guard file holding enough marked tests to push the step past `impactset.MaxCommandLength`: no guard step and the returned length above the limit.
  A zero `guardTests` renders no guard step, which every existing row reaches once it passes one.
- `TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach` calls the new signature with a zero `guardTests` and an empty stencils directory, and its expected text is unchanged.
- `TestBeginBatch_PromptFile` gains rows over `newBeginFixture`'s worktree with a root `go.mod` and `internal/g/g_test.go` written by `writeWorktreeFile`:
  a marked test renders the card's step list ending in the guard step before `lyx loom lint-comments`;
  a misplaced marker renders no `-run '^(` step and the prompt names `internal/g/g_test.go`, and the begin still succeeds;
  the same misplaced marker with the card targeting `internal/g/g.go#` and a `SourcePath` set renders the fix line: the card's pointer occurs after the first occurrence of `internal/g/g_test.go`, which the scan-error text carries, where without the fix line every occurrence of the pointer (the card list and the gate bullet) comes before it;
  an over-long step names `impactset.MaxCommandLength` in the prompt and renders no `-run '^(` step.
- `TestBeginBatch_Refusals` gains a row whose worktree holds a root `go.mod` and a directory chmodded `0o000`, its mode restored in `t.Cleanup` so `t.TempDir`'s removal succeeds, and skipped on Windows and as root as `rebaseline_test.go` does: the error names that path and `lyx webster begin-batch 1`, and no prompt file is written.
- `TestRecoverSpawnOrAttach` gains a row whose worktree holds a root `go.mod` and a marked test: `fx.Engine.LastPrompt` carries the same guard step line begin-batch renders; and a row whose worktree holds a root `go.mod` and an unreadable directory, restored and skipped the same way: the error names the path and `lyx webster recover-batch 1`, no strand spawns, and a stale report seeded in the reports dir is still in place.

Docs in this commit:
`internal/websterengine/doc.go`'s card gate paragraph adds the guard step before the two lints, the walk and its skipped directories, the drop rule, D3's prompt line for a scan error or an over-long step, the read-error refusal, and D4's instruction in the implementer body;
`contracts/specs/refusal-spec.md`'s webster section gains a transient row `guard walk unreadable`: begin-batch or recover-batch cannot read a path while walking the worktree for `//lyx:guard` tests; way forward: re-run the same verb once the path the message names reads;
`contracts/specs/loom-plan-spec.md`'s Tier 1 bullet says the card gate ends with the guard step over the root module's `//lyx:guard` tests before its two lints (it now names both lints), and that its line about a card breaking a package outside its targets holds for unmarked tests.

**ImpactSummary:** Every rendered card gate of a Go root module with marked tests gains one step, begin-batch and recover-batch can refuse on an unreadable worktree path, and both prompts can carry a guard-off line.

**Commit:** 6: Every card gate ends with the root module's guard tests
