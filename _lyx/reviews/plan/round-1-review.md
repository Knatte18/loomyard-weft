---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/05-retire-pane-era-anchors.md:18
    summary: Card 5 deletes render.AnchorTop and render.AnchorBelowParent while four surviving test members still reference them and only card 2 lists them, with card 2 keeping their anchors, so card 5 does not compile
    fix: In card 5's Edit list, after the `internal/reedengine#TestStrandWindowTarget` bullet, add `internal/reedengine/render#TestDisplayMarshalsWithoutRetiredKeys`, `internal/reedengine#goodAttachStrands`, `internal/reedengine#TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` and `internal/reedengine#setupAttachGeometryFixture`. In card 5's Tests paragraph, add a sentence saying these four move their displays to `own-window`, and that `TestDisplayMarshalsWithoutRetiredKeys` round-trips an `own-window` display with `KeepViewer` set, since a `top` value now decodes as `own-window` and would no longer round-trip equal. The overlay gate admits this fix.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-retire-shell-window-stack.md:159
    summary: No card rewrites the render/policy.go file header, which names the top-anchored and below-parent stack and the layout mechanics files, so the plan's own third verify grep fails on policy.go
    fix: In card 2's Intent, after the sentence that begins render keeps only the closed display vocabulary, add that the `internal/reedengine/render/policy.go` file header is rewritten to describe only the display check `ValidateDisplay` makes, dropping the placed-cell filtering, stack ordering, cycle repair, the below-parent and top wording and the policy-versus-mechanics note. Card 2 already edits `render#ValidateDisplay` in that file, so no target changes.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/08-sweep-docs-and-sandbox.md:11
    summary: The file headers of internal/reedcli/smoke_test.go and internal/reedcli/smoke_lifecycle_test.go still claim layout survival under stacked below-parent adds; no card rewrites them, and the third verify grep exempts every reedcli test file, so the acceptance criterion's no-below-parent sweep fails unnoticed
    fix: Add `internal/reedcli/smoke_test.go` and `internal/reedcli/smoke_lifecycle_test.go` to card 8's Prosa list after `internal/reedengine/watchdog_test.go`. Add an Intent sentence saying both file headers drop the layout-survival-under-stacked-below-parent-adds scenario from their inventories. The overlay gate admits this fix.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/02-retire-shell-window-stack.md:27
    summary: Card 2 deletes both test functions in render/policy_test.go as members but not the file, leaving a stale header and unused slices and testing imports that fail compilation
    fix: In card 2's Delete list, replace the two bullets `internal/reedengine/render#TestPartitionByAnchor` and `internal/reedengine/render#TestBreakCyclesTerminatesAndKeepsEveryStrand` with the single file bullet `internal/reedengine/render/policy_test.go`, and name that file beside the other render test files the Tests paragraph says go whole. The overlay gate admits this fix.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/05-retire-pane-era-anchors.md:44
    summary: Card 5 lists internal/reedengine#TestWouldFormCycle as an Edit target, but that test names no anchor and builds strands with a zero Display, so the card's Intent changes nothing in it
    fix: Remove the `internal/reedengine#TestWouldFormCycle` bullet from card 5's Edit list. The overlay gate admits this fix.
---

### [BLOCKING] Card 5 deletes the anchor constants under four test members it does not edit

**Location:** _lyx/plan/05-retire-pane-era-anchors.md:18

**Issue:** Card 5 deletes `render#AnchorTop` and `render#AnchorBelowParent`.
Four members that survive card 5 still reference them:
`render/types_test.go:28` and `:45` in `TestDisplayMarshalsWithoutRetiredKeys` (`AnchorBelowParent`, and `AnchorTop` in its round-trip `want`),
`attach_test.go:80` in `goodAttachStrands`, `attach_test.go:242` in `TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` (`AnchorTop`),
and `attachgeometry_integration_test.go:139` and `:147` in `setupAttachGeometryFixture`.
Only card 2 lists them.
The gate's `caller-uncovered` check accepts a caller listed on an earlier card, but card 2's Intent changes only their `Size` and layout assertions, and for its fixtures it says "the displays keep their anchors for card 5".
Card 5 lists none of the four.
The `render` and `reedengine` test binaries therefore stop compiling at card 5's commit, against the rule that a card builds on its own.
The round-trip half of `TestDisplayMarshalsWithoutRetiredKeys` also needs a new value, not just a new spelling: once `Anchor.UnmarshalJSON` maps `"top"` to `own-window`, a `top` display no longer round-trips equal.

### [MEDIUM] render/policy.go's file header is left describing the stack

**Location:** _lyx/plan/02-retire-shell-window-stack.md:159

**Issue:** After card 2, `policy.go` holds only `ValidateDisplay`.
Its header still reads "filtering strands down to the placed cells (top-anchored first, then the below-parent stack) … ordering the stack by insertion, and repairing a corrupt cyclic parent table", and it points at `layout.go`/`checksum.go`, which card 2 deletes.
No card names this header: card 2 and card 5 edit only the `ValidateDisplay` member, and card 8's Prosa covers `types.go` but not `policy.go`.
`policy.go` is not in the third `## verify:` grep's exception list.
Running that grep against the current tree lists `internal/reedengine/render/policy.go`, so the plan as written fails its own closing gate.

### [MEDIUM] Two reedcli test-file headers keep the below-parent claim, and the verify grep's exemption hides them

**Location:** _lyx/plan/08-sweep-docs-and-sandbox.md:11

**Issue:** `internal/reedcli/smoke_test.go:5` and `internal/reedcli/smoke_lifecycle_test.go:3` are file headers listing "layout survival under stacked below-parent adds" among the scenarios.
Card 5 edits `listPaneLines` and `TestSmokeLifecycle`, but those member spans do not include a file header, and no card's Prosa names either file.
The acceptance criterion excepts only the CLI refusal, the engine refusal and remap, retired-key fixtures, sandbox retirement notes and `docs/benchmarks/`.
The third verify grep instead exempts every `internal/reedcli/*_test.go`, so these two leftovers pass verify while breaking D8's sweep and the criterion.
The wide exemption exists because the new refusal tests' files are deliberately unpinned.
The fix therefore covers the two concrete leftovers instead of narrowing the grep.

### [LOW] policy_test.go is emptied but not deleted

**Location:** _lyx/plan/02-retire-shell-window-stack.md:27

**Issue:** `render/policy_test.go` holds exactly two functions, `TestPartitionByAnchor` and `TestBreakCyclesTerminatesAndKeepsEveryStrand`, and imports `slices` and `testing`.
Card 2 deletes both as members, which leaves a file with a header naming the retired stack ordering and two unused imports, which fail compilation.
Card 1 handles the same situation for `watchloop_test.go` by naming the emptied file's removal; card 2 should delete this file whole the same way.

### [LOW] Card 5 lists a test its Intent does not change

**Location:** _lyx/plan/05-retire-pane-era-anchors.md:44

**Issue:** `TestWouldFormCycle` (`strand_test.go:172-198`) builds three strands with only `GUID` and `Parent` and calls `wouldFormCycle`.
It names no anchor and does not rely on `lyx reed add`'s default.
The Intent's claim that "every other test listed used `below-parent` or `top` only as a convenient anchor, or relied on `lyx reed add`'s old default" is false for it.

## Rejected

- Card 5's Edit list (focus 1): every other listed test member either references a retired constant in its span, or swaps `split-window` for `new-window` (the `TestLaunchStrandLocked_*` tests, `launchFake` and `TestMultiplexerContract`'s split-path comment).
  The rest reach `lyx reed add`'s old default through `addStrand`/`addStrandIn` and read panes via `listPaneLines` (`TestSmokeCrashRecovery`, `TestSmokeStateRecovery`, `TestSmokeSelvage`, `TestSmokeRemoveLastStrandThenAddRunsTheNewCommand`).
  The unlisted `TestSmokeColdStart` counts two panes through `listPaneLines` and passes once card 5 makes that listing session-wide.
- Refusal reachability (focus 2): `validateDisplay`'s retired-anchor branch is reached by a Go-built `render.Anchor("below-parent")`, which no JSON decode touches, and card 5 places the check at the head of `AddStrandUnless` and `ReplaceStrand`, ahead of the session pre-flight and the old-pane kill (`replace.go:46-61`).
  `TestWriteOps_RefuseARetiredAnchor` reaches it on all four ops.
  The CLI's `overview` and retired-value refusal runs before `c.eng` is touched, and `TestAddCmd_RefusesAnAnchorOutsideTheVocabulary` reaches it with a nil engine.
- Attack surface and CLI shape (focus 3): cards 5 and 6 only narrow the accepted `--anchor` values, and say so.
  The decode remap is bounded to the two retired spellings, keeping every other value as written.
  Neither card adds a command or changes `Use`, audience or `Short`, so `PATTERN-cli-cobra`'s audience bullet is not engaged.
- Card 7 (focus 4): its Intent carries each D3 sentence (move at the head of every reconciling op, dead pane moved before the dead-pane kill, re-list after a move, heal before move in up and resume, the last-pane exception, and the `@lyx_strands` fallback that never reads the current window).
  No `Edit` glyph on it changes a signature: `strandWindowTarget`, `launchStrandLocked`, `reconcileApplyPersistLocked`, `Resume` and `openOwnWindowLocked` keep theirs.
  Its `Create` helpers all serve the one move surface and are bundled in the one card.
- Card order (focus 5): no card targets a symbol a later card creates, and none edits a symbol an earlier card deleted; card 7's edit of `launchStrandLocked` follows card 5's.
  F1 is the one ordering-adjacent defect.
- Fidelity (focus 6): D4 sits on cards 2 and 5, D5 on card 4 (`RetiredKeys` with `CategoryNone`, `width`/`height` kept), D6 on cards 1 and 2 (`watchdog` key kept, comment rewritten), D7 on card 3 (explicit `unbind-key -n M-z`, shell click no longer unzooms), and D8 on card 8 plus the per-card test sweeps.
  Card 3's `alt+z` drop from `internal/vscode`'s forwarded keys follows from D7's binding retirement.
- Verify coverage (focus 7): `./...` under the untagged and integration tiers runs every targeted package, including `internal/vscode`, `internal/pairteardown`, `internal/standalonegeom`, `internal/configreg` and `internal/configsync`.
  The tmux-tagged tests these cards touch are compiled by `go vet -tags tmux`, which the rubric admits.
  The section follows the template's order.
  `tools/` holds no Go package this plan targets.
  Apart from the gaps raised in F2 and F3, the grep gates match the current tree.
- Test economy (focus 8): the new tests cover behavior with no existing test.
  `TestPinBootOptionsLocked_ClearsTheWindowResizedHook` replaces the deleted boot test, and `TestListReconcilePanesLocked_MovesShellWindowStrands` pins the new move.
  `TestAddCmd_RefusesAnAnchorOutsideTheVocabulary` is the faster-tier pin `TestStrandNamingAndList` cannot give.
  Card 8 asserts no prose.
- Card 8 (focus 9): its Prosa group holds file paths only, and no card writes this task's board entry.
- Card 5's tolerant-records claim holds: `internal/state`'s `keepUnknownInList` pairs an old element only when it decodes and re-encodes equal to the new one (`keep.go:154`), so the remap at decode keeps the undeclared `size` and `contentRows` keys.
