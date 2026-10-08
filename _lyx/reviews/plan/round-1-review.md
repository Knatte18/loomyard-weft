---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: cmd/lyx/testdata/test-budget.yaml (named by no card; cards 1, 2, 3, 4, 5, 7, 9, 10, 11, 15 add top-level tests)
    summary: No card edits the per-package test budget, so go test ./... in verify fails once the new test files land (PATTERN-test-speed budget)
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/06-retire-status-line-pipeline.md
    summary: Card 6 removes Geometry.RepoName but leaves three tmux-tier reedengine test files that still set it out of its targets, so go vet -tags tmux fails
  - id: F3
    severity: BLOCKING
    class: scope
    location: _lyx/plan/15-vscode-keybindings-block.md
    summary: Card 15 changes the ideengine.Spawn signature but omits the integration tests that call it, so go vet -tags integration fails
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/03-strand-pane-options.md
    summary: The attach re-assert is placed after the pane listing, which follows two early chain-suppression returns, so the claim that it runs even when the chain is suppressed is only partly true
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/13-webster-review-segments.md
    summary: ImpactSummary tally of nine spec literals in five packages is wrong (ten literals in four packages) and is a perishable count
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/13-webster-review-segments.md
    summary: Intent edits the comment blocks of the role constants, but internal/websterengine/roles.go, which declares MerriamStrandRole and RoleRecovery, is not a target
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/03-strand-pane-options.md
    summary: paneoptions_test.go argv table for setStrandPaneOptionsLocked repeats what the launch test in spawn_test.go already pins
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/08-title-row-geometry.md
    summary: reapply_test.go assertion that the watchdog re-apply runs the pins and no signal entry repeats the SkipFocus assertion planned for apply_test.go
  - id: F9
    severity: LOW
    class: design
    location: _lyx/plan/07-alt-key-and-mouse-bindings.md
    summary: The VIEW click introduces a new session option and a run-shell -C format-expanded command that the decision record does not name, and the Intent states no bound for it
  - id: F10
    severity: LOW
    class: decision
    location: _lyx/plan/10-spawn-order-revival.md
    summary: Pair start time is the modification time of a fabric-synced tracked seed.json, which a checkout or rewrite resets, and the Intent states no bound for that
---

### [BLOCKING] No card raises the per-package test budget

**Location:** `cmd/lyx/testdata/test-budget.yaml`, named in no card's targets; the cards that add top-level tests are 1, 2, 3, 4, 5, 7, 9, 10, 11 and 15.
**Issue:** `PATTERN-test-speed` (Test budget) pins, per package, the most top-level `func Test…` functions its `_test.go` files may hold, under every build tag.
`cmd/lyx/testbudget_test.go` fails when a package exceeds its row or has tests and no row, and it runs in the untagged tier, so the plan's own `go test ./...` fails on it.
The rows sit at the current counts: `internal/reedengine` holds 216 top-level tests against a row of 216, `internal/vscode` holds 9 against 9, and `internal/segmentcolor` has no row at all.
Card 1 creates that package with tests.
Cards 2, 3, 4, 5, 7, 9 and 10 add new test files to `internal/reedengine`, card 10 adds `spawnorder_test.go` to `internal/hubgeom`, card 11 adds `color_test.go` to `claudeengine`, and card 15 adds `keybindings_test.go` to `internal/vscode`.
Cards 5 and 6 delete tests in `reedengine`, so the net per package is not obvious, and the file's own header says a raise belongs in the commit that adds the tests.
A violation of a listed constraint is BLOCKING.
**Fix:** Add `cmd/lyx/testdata/test-budget.yaml` to the Edit targets of each card that changes a package's top-level test count, with the row recomputed from the cards up to and including that card (a new `internal/segmentcolor` row in card 1).
Card 6 may also drop the `internal/tokenvocab` row, which the gate tolerates but which is dead.

### [BLOCKING] Card 6 leaves tmux-tier tests that set the removed `Geometry.RepoName`

**Location:** `_lyx/plan/06-retire-status-line-pipeline.md`; uncovered sites are `internal/reedengine/ensuresession_integration_test.go:53`, `internal/reedengine/mouse_boot_integration_test.go:57` and `internal/reedengine/contract_integration_test.go:639` and `:758`.
**Issue:** Card 6 deletes `Geometry.RepoName` and says it updates "the test literals and assertions that set or read it", but its Edit list names only `lock_test.go`, the two geometry tellers' tests and `statusline` files.
The four literals above set `RepoName: "test-repo"` in `Geometry{…}` and sit in `//go:build tmux` files, so they compile only under `-tags tmux`.
`go vet -tags tmux ./...` in the plan's `## verify:` then fails, and the card does not build on its own.
**Fix:** Add the three files to card 6's Edit targets and drop the `RepoName` lines.

### [BLOCKING] Card 15 changes `ideengine.Spawn`'s signature without its integration callers

**Location:** `_lyx/plan/15-vscode-keybindings-block.md`; callers are `internal/ideengine/spawn_clean_integration_test.go` (lines 81, 123, 157, 200) and `internal/ideengine/spawn_hub_integration_test.go` (lines 67, 118, 140, 147, 158, 170, 181, 200, 220).
**Issue:** The Intent has `ideengine.Spawn` "return the keybindings result beside its error", and the ImpactSummary says its one caller is `idecli`.
Those two integration files call `err := Spawn(l, slug)` and `if err := Spawn(l, prime); err != nil`, the single-value form, and neither is a target.
Under `-tags integration` they stop compiling, so `go vet -tags integration ./...` and `go test -tags integration ./...` in verify fail.
The ImpactSummary's caller count is wrong for the same reason.
**Fix:** Either list both files as Edit targets and update their calls, or keep `Spawn`'s signature and carry the outcome another way (for example a package-level result seam like `CodeLauncher`), and correct the ImpactSummary.

### [LOW] Card 3's attach re-assert is not reached on two of the chain degrades

**Location:** `_lyx/plan/03-strand-pane-options.md`, Call sites and Tests.
**Issue:** The card calls `reassertStrandPaneOptionsLocked` "right after it lists the panes, … ahead of the chain guards, so a session or strand spawned by an older lyx gets them at the next attach even when the chain is suppressed", and its test asserts the re-assert "also on a suppressed chain".
In `AttachArgv` the pane listing comes after the `window-size` readback and the `#{status}` readback, and each of those returns `errAttachChainSuppressed` first.
On those two degrades the options are not re-asserted, which D2 ("on every bound, live strand pane in the attach pre-flight") does not allow.
**Fix:** Place the re-assert (with its own pane listing) before those readbacks, or state in the Intent that those two degrades skip it and narrow the test's claim; the first matches D2.

### [LOW] Card 13's ImpactSummary tally is wrong

**Location:** `_lyx/plan/13-webster-review-segments.md`, ImpactSummary.
**Issue:** It says "nine spec literals in five packages".
The Intent lists two Webster specs, two Burler specs, four Bouncer literals and two Treadle specs, which is ten literals, and the Edit list spans four packages (`websterengine`, `burlerengine`, `shedadapters`, `treadleengine`).
A literal count in a summary is also a perishable specific.
**Fix:** Replace the tally with a conclusion that names no count, for example "spec literals in the webster, burler, shedadapters and treadle packages gain one field each; no behavior change beyond the run's color".

### [LOW] Card 13 edits role-constant comments in a file it does not target

**Location:** `_lyx/plan/13-webster-review-segments.md`, Intent last paragraph before Tests.
**Issue:** "Where a role constant is declared, its comment block gains the segment beside it."
`MerriamStrandRole` and `RoleRecovery`, the roles of the Webster specs, are declared in `internal/websterengine/roles.go`, which is not among the card's targets.
The other roles' constants sit in files that are targets.
**Fix:** Add `internal/websterengine/roles.go` to the Edit targets, or drop the sentence for Webster.

### [LOW] Card 3's `setStrandPaneOptionsLocked` argv table repeats the launch test

**Location:** `_lyx/plan/03-strand-pane-options.md`, Tests.
**Issue:** `paneoptions_test.go` plans a table over `setStrandPaneOptionsLocked`'s argv for a colored strand, a strand with no segment and a strand whose segment has no key.
The same card extends `spawn_test.go`'s `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls`, which already pins the exact tmux calls a launch issues in order, and the helper is reached only through that launch path and the attach re-assert (`PATTERN-test-economy`: an unexported helper is tested through the surface that uses it).
**Fix:** Express the three variants as launch fixtures in the existing launch test (and one re-assert row in `attach_test.go`), and keep only the `strandLabel` table in `paneoptions_test.go`.

### [LOW] Card 8 asserts the watchdog re-apply's pins twice

**Location:** `_lyx/plan/08-title-row-geometry.md`, Tests.
**Issue:** `apply_test.go` is to assert that the `SkipFocus` path issues the pins and no `set-hook`, extending `TestApplyLayoutLockedOpts_SkipFocusSuppressesSelectPane`.
`reapply_test.go` is to assert that the watchdog re-apply runs the pins and never the signal entry.
`reapplyLayout` reaches the apply only through `applyLayoutLockedOpts(st, live, applyOpts{SkipFocus: true, …})`, so the second assertion exercises the same code and the same behavior.
**Fix:** Keep the `SkipFocus` assertion in `apply_test.go` and drop the `reapply_test.go` one, unless the card names what the re-apply adds that the apply path does not cover.

### [LOW] Card 7's VIEW click adds unlicensed state and an unbounded expansion

**Location:** `_lyx/plan/07-alt-key-and-mouse-bindings.md`, `MouseDown1Status` bullet and Bound paragraph.
**Issue:** To reach the strand window from a binding, the card has `markStrandWindowLocked` also write a new session option `@lyx_strand_window` and has the `view` branch target it "through a format-expanded command (`run-shell -C`)".
D3 licenses only the `@lyx_strands` window marker, and D9 says only "select the strand window and unzoom it".
The Bound paragraph covers the five keys and the left click but says nothing about this new routing edge: a stored option value is expanded into a tmux command at click time, and the card does not say that reed alone writes it, that it holds only a tmux window id, or what a stale or empty value does.
**Fix:** State in the Intent that the option holds a tmux-issued window id written by reed alone and that an empty or stale value selects nothing; or reach the strand window without new state, for example by expanding the `@lyx_strands` loop in the binding.

### [LOW] Card 10's start-time source is unstable and its bound is unstated

**Location:** `_lyx/plan/10-spawn-order-revival.md`, Teller, and `00-overview.md` Shared Decisions (Run start time).
**Issue:** D12 orders pairs by "their run's start time read from the pair's own records".
The plan substitutes the modification time of `seed.json`.
`seed.json` has no timestamp field, and `shedrun.RunDir` places it under `_lyx`, the durable, fabric-synced tree, so a checkout, rebase or re-materialized worktree resets its mtime and changes the order.
The Shared Decision records why a field is unavailable but not this instability, and the card's Intent does not say how far the ordering can be trusted.
**Fix:** State the bound in the Intent (order holds on the machine where the pair was created and seeded, and a re-materialized worktree sorts as a new run), or name a source that a checkout does not reset.

## Focus verdicts

- Cards 12 to 14 against D13 and D14: the seven segment constants in card 1, the template keys in card 2 and the Shared Decisions list are identical (`coordinator`, `discussion`, `plan`, `webster`, `review`, `describe`, `landing`).
  The assignments cover every row of D13's table, and every production `shuttleengine.Spec` literal with a fixed role is covered.
  The generic spawn sites `internal/shedrecipe/entries_singlellm.go` and `internal/shuttlecli/run.go` take their role from a caller, and D13 gives a strand with no segment no color, so leaving them out is not a defect.
  The orch exception in card 14 matches D14 and `PATTERN-orch-pane-single-writer`.
  The only defects found there are F5 and F6.
- Attack-surface question for cards 4, 7, 9 and 15: cards 4, 9 and 15 state what the new path can skip or let through and what bounds it, and card 7 does for the five keys, the quoting and the `#` doubling.
  The gap is the VIEW edge, F9.
- Card order: no card among 5, 7, 8 and 9 uses a symbol introduced by a later card, and card 11 uses nothing from cards 12 to 14 (its mention of card 14 is prose about `ColorByCaller`, which card 11 itself defines).
  Cards 9 and 14 each use an earlier card's output (cards 8 and 11), which is the right direction.
- `## verify:` runs `go test ./...` and `go test -tags integration ./...` over every targeted package, in the template's order, and compiles the `tmux` and `llm` tiers, which the rubric exempts.
  It is not a finding.
  F2 and F3 are about the files the cards omit, not the section.
- Test economy: cards 5 and 9 plan one real-tmux file each for behavior no fake can show, with their other assertions added to existing tests.
  The overlaps are F7 and F8.
