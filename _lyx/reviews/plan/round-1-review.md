---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/17-review-roles-edit-directive.md, _lyx/plan/19-landing-friction-driver-edit-directive.md
    summary: The edit-directive cards do not enumerate the tests that hand-build a stencil or its values map, so card 17 breaks TestBouncer_MarkerCompleteness_BothTemplates and card 19's planned assertions fail against marker-less fixture stencils
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-hubreconcile.md (the pair call), _lyx/plan/05-fabric-walk-reads.md (CodeWorktree.Path)
    summary: The pair call matches opts.Pair by string equality against a path from git worktree list, which List does not normalize, so lyx batten run can fail with pair not in listing on Windows or a symlinked hub path
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/11-head-contains.md (warpforward_integration_test.go test)
    summary: Card 11's fabricengine.HeadContains test repeats the HEAD-sha and absent-sha rows card 13's StencilSource table runs through its only caller
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/08-orch-loom-start-reconcile.md (startreconcile_integration_test.go)
    summary: Card 8 creates a new loomcli test file for a surface smoke_prebootstrap_integration_test.go already drives, and its second hub repeats the unparseable loom.yaml row card 6 and card 8's orchcli step already cover
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/06-hubreconcile.md (plan:internal/configengine#ScratchDir)
    summary: configengine.ScratchDir is a public configengine member bundled into the hubreconcile card instead of its own card
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/03-configsync-file-error.md (internal/configsync#decodeLeafValues)
    summary: Dropping the path prefix from decodeLeafValues leaves its path parameter unused, and the card neither re-signs it nor keeps a use for it
---

### [MEDIUM] Edit-directive cards miss tests that hand-build stencil text or values

**Location:** `_lyx/plan/17-review-roles-edit-directive.md`, `_lyx/plan/19-landing-friction-driver-edit-directive.md`

**Issue:**
`stencil.Fill` (`internal/stencil/stencil.go`) refuses a template whose top-level marker has no value, and ignores a value whose marker the template lacks.
Two kinds of existing test therefore react to a new required marker, and neither card enumerates them.

- Card 17: `internal/shedadapters/bouncer_seed_test.go`'s `TestBouncer_MarkerCompleteness_BothTemplates` fills the real `stencils.BouncerTemplateSeed` and `stencils.BouncerTemplateJudge` from hand-built values maps that set `parentdirective.MarkerName` (lines 394 and 427).
  Once card 17 adds `{{.edit_directive}}` to both stencils, `stencil.Fill` returns "unfilled top-level marker(s): edit_directive" and the test fails.
  Card 17 lists the file but says only that it will "assert the same of the judge and seed prompts", and adds an `edit_directive` value to burler's `template_test.go` maps alone.
- Card 19: its Intent says `mergeresolve_test.go`, `describe_test.go` and `reflect_test.go` "each already render their prompt against seeded stencils".
  Each instead overwrites the seeded stencil with a hand-written fixture (`conflictStencilFixture`, `describeStencilFixture`, `reflectionStencilFixture`) that carries `{{.parent_directive}}` and no `{{.edit_directive}}`.
  The renderer fills the value, `Fill` drops it because the fixture has no marker, and the planned "contains the directive's `Edit or Write` sentence" assertion fails.
  Only `driverprompt_test.go` renders a seeded stencil.

The same gap recurs across two cards, so it is filed once against the method: the cards enumerated renderers and test files, not the tests whose stencil text or values are hand-built.

**Fix:**
Card 17 adds an `edit_directive` value to both hand-built maps in `TestBouncer_MarkerCompleteness_BothTemplates`.
Card 19 adds `{{.edit_directive}}` to the three fixture constants, so the assertion reaches the filled text, and corrects its "against seeded stencils" claim.
Before fixing, grep the targeted packages' tests for `stencil.Fill`, `parentdirective.MarkerName` and `{{.parent_directive}}` fixture strings, so each card names every test that hand-builds an opening stencil or its values.

### [MEDIUM] The pair call compares unnormalized worktree paths

**Location:** `_lyx/plan/06-hubreconcile.md` (the pair call paragraph), `_lyx/plan/05-fabric-walk-reads.md` (`CodeWorktree`, `CodeWorktrees`), `_lyx/plan/09-batten-run-reconcile.md` (second call)

**Issue:**
Card 9 passes `Options{Pair: fabricengine.WorktreePath(location, slug)}`, which is `filepath.Join(l.HubPath, slug)` (`internal/fabricengine/junction.go:24`).
Card 6 then "finds the `CodeWorktree` whose `Path` is `opts.Pair` in a fresh listing", and calls a miss "an error naming the path", which stops `lyx batten run`.
Card 5 builds that listing from `List`, and `parseWorktreePorcelain` stores the porcelain `worktree` path as git prints it, with no `filepath.FromSlash` or `Clean` (`internal/fabricengine/worktreelist.go`).
Every other `List` caller normalizes it (`PrimeName` and `matchParentBranch` run `filepath.FromSlash`).
Git prints forward slashes on Windows and its own canonical form of the path, so a byte comparison against a `filepath.Join` result can miss the very pair batten just created.
Card 5 never states which form `CodeWorktree.Path` takes.
`CLAUDE.md` makes Windows a supported platform, and the hubforge tests on Linux would not catch the miss.

**Fix:**
Card 5 states that `CodeWorktree.Path` is the normalized worktree root: `filepath.Clean(filepath.FromSlash(entry.Path))`, or the root `lyxcwd.ResolveWorktree` returns.
Card 6 normalizes `opts.Pair` the same way before comparing, or resolves the pair through `OpenCodeWorktree(opts.Pair)` and matches on the resolved root.
Card 5's `CodeWorktrees` test asserts the listed path equals `fabricengine.WorktreePath` for a pair.

### [LOW] Card 11's fabricengine HeadContains test repeats card 13's rows

**Location:** `_lyx/plan/11-head-contains.md`, the `warpforward_integration_test.go` bullet

**Issue:**
Card 11 adds a hubforge test of `fabricengine.HeadContains` asserting that the prime's HEAD sha is true and an absent sha is false.
Card 13's `StencilSource` table in the same file, on a hubforge prime, has rows "The prime's HEAD sha gives `BuildInHead`" and "An absent well-formed sha gives `BuildNotInHead`".
Those rows run through `HeadContains`, which is a one-line delegation whose only caller is `StencilSource`.
`PATTERN-test-economy` says a new test must cover a behavior no existing test covers, and calls a redundant test a finding.
The behavior is already covered by card 11's own `gitrepo` table, and card 13's `StencilSource` table covers it again at the public surface.

**Fix:**
Drop card 11's `warpforward_integration_test.go` test and that file from card 11's `**Edit:**`.
The `gitrepo` table pins the read, and card 13's `StencilSource` table pins the delegation.

### [LOW] Card 8's new loomcli test file and its duplicate error row

**Location:** `_lyx/plan/08-orch-loom-start-reconcile.md`, the `startreconcile_integration_test.go` bullet

**Issue:**
The plan template says a `**Create:**` test file appears only for a surface no existing test file covers.
`internal/loomcli/smoke_prebootstrap_integration_test.go` already drives `loomCLI.arm` on hubforge pairs, `TestLoomStatusAndPauseOnNeverBootstrappedPair` among them.
Its "status leaves the key, start removes it" rows belong there.
The file's second hub, with an unparseable pair `loom.yaml` returning the `*hubreconcile.WorktreeError` message for `resume`, repeats card 6's `ensure_integration_test.go` row (same file, same error type, same `lyx config loom` text).
It also repeats card 8's own `refusal_integration_test.go` step, which already asserts the start verb's envelope for that failure.
The behavior new to this card is the `resume` gate, and it needs no second hub.

**Fix:**
Fold the gate rows into `smoke_prebootstrap_integration_test.go`, and move the file from `**Create:**` to `**Edit:**`.
Pin `resume`'s gate on the same hub (a stale stamp that `resume` refreshes) instead of on a second hub with a broken file.

### [LOW] configengine.ScratchDir rides in the hubreconcile card

**Location:** `_lyx/plan/06-hubreconcile.md`, `plan:internal/configengine#ScratchDir`

**Issue:**
The rubric gives a public member of a module's surface, used across packages, its own card.
`configengine.ScratchDir` is an exported `internal/configengine` path accessor that `hubreconcile` calls.
Card 6 bundles it with a new package, so a review of configengine's path vocabulary has to read the hubreconcile card.
It sits beside `ConfigDir`, `ConfigFile` and `StagingFile`, and `StagingFile` already joins the same `<base>/.lyx/config` segments.

**Fix:**
Move `ScratchDir` into its own `Create` card ahead of card 6, with its doc comment saying it is the never-tracked mirror of `ConfigDir`.
Card 6 keeps it under `**Uses:**`, and its `StampPath`/`LockPath` assertions stay as they are.

### [LOW] decodeLeafValues keeps a parameter it no longer uses

**Location:** `_lyx/plan/03-configsync-file-error.md`, `internal/configsync#decodeLeafValues`

**Issue:**
`decodeLeafValues(data []byte, path string)` uses `path` only in its `parse %s: %w` error (`internal/configsync/configsync.go:119`).
Card 3 changes that error to `parse: %w` because the `*FileError` around it now names the file, so `path` goes unused.
The card lists the glyph with no re-sign arrow and says nothing about the parameter.

**Fix:**
Drop the parameter, writing the bullet as `` `internal/configsync#decodeLeafValues` -> `func decodeLeafValues(data []byte) (map[string]any, error)` ``.
Its two callers are both in `retireHubWideCopy`, which the card already lists.

## Focus items settled without a finding

- Fidelity: Decisions 1 to 14 and every Constraint are carried.
  The two loom-section refusal rows in card 8 are licensed by `PATTERN-refusal-way-forward`, which the Constraints list, since both refusals reach `lyx loom start`/`resume`.
  The carry-over entries license no scope, and the plan adds none for them.
- Attack surface: card 6 bounds the hub lock (`DefaultLockWait`, `*LockTimeoutError`, no stamp write) and the pair call (one pair's `_lyx/config`, same lock and mid-merge refusal).
  Card 9 restates the pair call's bound.
  Card 10 adds a `Stuck` exit ahead of `preflight.Check`, which bypasses nothing, and the walk's own bounds come from card 6.
- Card order: no card uses a later card's target, including 12 and 13 against 11 and 15 to 20 against 14.
  Import cycles: `hubgeom` to `hubreconcile` to `configreg` is acyclic, since `configreg`'s dependencies reach `preflight` but not `hubgeom` or `preflightshed`.
- Granularity: cards 15 to 19 are each one role group whose stencils and renderers change together, so each reviews on its own.
  Card 4 (`BoardLyxPresent`, now used across packages) and card 7 (a public `hubgeom` constructor) each stand alone.
- Member glyphs and re-sign arrows: present on every `Edit` card whose Intent names symbols, with the exception F6 names.
- Verify: `./...` covers every targeted package, `contracts/stencils` and `cmd/lyx` included, with the integration tier run and the tmux and llm tiers compiled, in the template's order.
