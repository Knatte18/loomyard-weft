---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-mount-shortcuts.md (Intent, the two guard-skip bullets and TestShortcuts_FileAndRootListing)
    summary: Card 5 makes the root.Long module check and the sandbox-coverage walk skip every root child whose GroupID is shortcutcli.GroupID, but states no bound on what that skip lets through, and nothing ties the group to the shortcut file, so a module command placed in that group would escape both module guards unnoticed.
    fix: In card 5's Intent, after the two skip bullets, state that the skip lets through exactly the root children in shortcutcli.GroupID and that the bound is the shortcut file, and add to TestShortcuts_FileAndRootListing's bullet list an assertion that the names of newRoot()'s children whose GroupID is shortcutcli.GroupID equal, as a set, the names shortcutcli.Load() returns, so a module command carrying that group fails the test.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/05-mount-shortcuts.md (Edit cmd/lyx#newRoot) and _lyx/plan/06-loom-status-eighth.md (Edit internal/loomcli#statusStrandShareDen)
    summary: Two cards edit a free-standing comment that lies outside every member span they target, so the glyph scope guard will report the edit as out of scope and the plan understates where each card writes.
    fix: In card 5's Edit list replace `cmd/lyx#newRoot` with `cmd/lyx/main.go`, which covers newRoot, the root Long and the file and package comments at main.go lines 1 to 12. In card 6's Edit list replace `internal/loomcli#statusStrandShareDen` with `internal/loomcli/bootstrap.go`, which covers the constant and the const-block comment at bootstrap.go line 25. Adding the file beside the member instead is refused by redundant-file-target. Both swaps together pass validate-plan --overlay with no new blocking finding.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/05-mount-shortcuts.md (Intent, newRoot bullet, root Long line)
    summary: Card 5 adds a root Long line that lists the shortcuts by name (attach, prime), a hand-kept list that the same card exempts from the root.Long names-every-module check, so it goes stale without warning on the next shortcut added to the file, which the decision record names as the way to add one.
    fix: In card 5's newRoot bullet, change the root Long line to one that names the file and the help group without listing shortcut names, for example "Shortcuts, declared in internal/shortcutcli's embedded file, are listed under their own heading in lyx --help."
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/05-mount-shortcuts.md (shortcutProblems bullet and TestShortcuts_FileAndRootListing synthetic rows)
    summary: shortcutProblems prescribes a branch for an argv that resolves to root itself, but no synthetic row reaches it, since the row with argv [nosuch, verb] fails in root.Find with an unknown-command error before that branch runs.
    fix: In TestShortcuts_FileAndRootListing's synthetic rows, add an argv made only of a global flag, [--verbose], which root.Find strips and resolves to root with no error, expecting the problem that names the root.
---

### [MEDIUM] Shortcut-group skips in the module guards carry no stated bound

**Location:** `_lyx/plan/05-mount-shortcuts.md`, the `TestCLITree_EveryCommand` and `TestSandboxCoverage_AllModulesCoveredOrExcluded` bullets and `TestShortcuts_FileAndRootListing`.

**Issue:** The rubric's attack-surface question requires a card that weakens a guard to state what it now lets through and what bounds it.
Card 5 gives the reason for each skip ("a shortcut is not a module", "nothing of its own for a sandbox scenario to exercise") but not the bound.
Both skips are keyed on `GroupID == shortcutcli.GroupID` alone.
`shortcutProblems` checks the file's shortcuts against the root, but nothing checks the root's shortcut group against the file: a module command that sets that `GroupID` would drop out of the `root.Long` check (`cmd/lyx/clitree_test.go:137`) and the sandbox-coverage walk (`cmd/lyx/sandbox_coverage_test.go:61`) with every test still green.
Card 4's own bound ("no hub, repo or user can add or change one") covers who can write the file, not which commands the guards skip.

### [LOW] Free-standing comment edits fall outside the targeted member spans

**Location:** `_lyx/plan/05-mount-shortcuts.md` (`cmd/lyx#newRoot`), `_lyx/plan/06-loom-status-eighth.md` (`internal/loomcli#statusStrandShareDen`).

**Issue:** `lyx quarry resolve` puts `cmd/lyx#newRoot` at `cmd/lyx/main.go:83-158`, while card 5 also rewrites "the two file comments at the top of `cmd/lyx/main.go`" at lines 1-12.
It puts `internal/loomcli#statusStrandShareDen` at `internal/loomcli/bootstrap.go:28` only, while card 6 rewrites "the doc comment of the const block holding `statusStrandShareDen`", which sits at line 25 on the `const (` declaration and not on the spec.
No member span covers either comment, so per the rubric the right target is the file's self glyph.
The same gap appears on two cards, so it is filed once, against the method.

### [LOW] Root Long line hand-lists the shortcuts its own check no longer guards

**Location:** `_lyx/plan/05-mount-shortcuts.md`, newRoot bullet: "Shortcuts, from internal/shortcutcli's embedded file: attach, prime."

**Issue:** D5 exempts shortcuts from the `root.Long` names-every-module check, and the decision record's Out section says the file is where more shortcuts are added by commit.
A Long line that spells out the current shortcut names is an enumerated list that nothing checks, so the first shortcut added to the file leaves `lyx --help`'s Long wrong while every test passes.
`lyx --help` already lists the shortcuts under their `Shortcuts:` group, so the names in the Long are redundant.
The decision record licenses naming the shortcuts in `docs/overview.md`, and that bullet is not affected.

### [LOW] shortcutProblems' resolves-to-root branch has no reaching row

**Location:** `_lyx/plan/05-mount-shortcuts.md`, `shortcutProblems` bullet ("resolve to an error, to `root` itself, …") and the synthetic rows.

**Issue:** `newRoot` sets no `Args`, so cobra's `Find` runs `legacyArgs` on the root, and an unknown first word such as `nosuch` returns an "unknown command" error.
So the `[nosuch, verb]` row reaches the error branch, and no row reaches the root-itself branch.
An argv made only of a flag the root declares, `[--verbose]`, is stripped by `Find`, leaves no arguments, and resolves to the root with a nil error, which is the input that branch exists for.

## Focus notes

- Card 3 arrows: `pasteSubmitArgv` and `fakeTmux.exec` carry correct re-sign arrows.
  `TmuxCmd.execHook` is a struct field, which takes no arrow, and listing `internal/reedengine#TmuxCmd` covers it.
  `TmuxCmd.run` and `TmuxCmd.output` change only their bodies (a nil `stdin` argument to the hook), so they need member glyphs and no arrow, and the card lists both.
  `installFakeTmuxOn` keeps its signature and its assignment line, so it needs no arrow; listing it does no harm and is not a defect.
- Card 4 granularity and test economy: not a defect.
  The private helpers (`parse`, `mount`, `routedWords`, `routeWords`, `runArgvs`) and the fakes serve the one public surface (`Load`, `Mount`, `Shortcut`, `GroupID`) and sit on the same card.
  `TestShortcut_Run` pins dispatch and word routing, and `TestMount_File` pins parse and mount, including the Warn and the `Short` built from the argvs, so neither duplicates the other.
- Attack surface: card 4 states its bound (same-binary lyx commands only, a full root per argv, an embedded file, `--` and trailing words go only to the last argv, and a local `--json` only after `--`).
  Card 5's guard skips do not state a bound (F1).
- Reachability: card 2's held/dead skip has named rows that fail on the old code.
  Card 4's `parse` shapes are reached by `TestMount_File`'s malformed, scalar, mapping and mixed-sequence rows.
  `routeWords` has no refusal: an unknown or bad-value word is forwarded, and the `--json=bogus` row pins that.
  Card 1's `PauseGeometry` failure branch is pre-empted by `hubgeom.ReedGeometry`, which reads the same `.git` entry first, as the card says; the branch follows D1's prescribed `PauseGeometry` path, so no finding.
  Card 5's root-itself branch: F4.
- ImpactSummaries of cards 1 and 2 are real conclusions.
  `newOverviewSource`'s only callers are `overview.go:380` and `overview_test.go:261`, and `advisorWork.outstanding` keeps its signature.
  Card 1's members match its Intent.
- Card order: card 5 uses only card 4's handles, card 1 updates every caller of `newOverviewSource`, and card 6 has no dependency.
- Verify: every package a card targets is run by both `go test` rows, `go vet -tags tmux` compiles `paste_integration_test.go`, the order matches the template, and `internal/statuspane` is not edited.
- Fidelity: D1 to D6 and every Constraint are carried.
  Card 5's docs and test-budget edits are licensed by the Scope docs line and the test-budget Constraint, and `cmd/lyx: 78 → 79` matches the current count of 78 top-level tests besides `TestMain`.
  Card 4's pre-run confirmations hold: `traceOnce`, `logger.Arm`, the gate swap, and `seedStencilsAt` writing only a subtree that is due.

## Rejected

- Card 1 docs/overview.md batten-bullet edit as unlicensed scope: it keeps an existing description of the overview's state cell accurate, which the repo's same-commit docs rule requires; not a defect.
