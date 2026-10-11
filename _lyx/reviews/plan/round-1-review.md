---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-fabric-unwire-apply.md:43
    summary: the sandbox caller list misses F11's dirty-worktree matrix, which runs bare unwire, and names F6 for the alternation that lives in F15
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-fabric-unwire-apply.md:34
    summary: card 6 changes Unwire's observable behaviour but leaves internal/fabricengine/doc.go's Unwire paragraph, which says it removes every junction, untouched
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-fabric-unwire-apply.md:41
    summary: the new unwire_integration_test.go top-level test re-asserts junction state the extended engine tests pin and belongs as a step in TestRunCLI_HubMutationScenario
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-reconcile-floor-migrate.md:36
    summary: the value-shape rows (at floor, null, env marker, abc) are pinned twice, once in the new floor_test.go table and again as TestReconcileAll rows
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/05-reconcile-floor-migrate.md:34
    summary: shuttleengine and orchengine gain an exported MigrateConfig but card 5 leaves their doc.go files untouched
---

### [MEDIUM] Sandbox caller list misses F11 and mislabels F15

**Location:** `_lyx/plan/06-fabric-unwire-apply.md:43`

**Issue:** D6 requires every caller lyx ships to pass `--apply`, and the acceptance criteria repeat it.
The card lists the `tools/sandbox` sites as "S6's goal, durability note and watch line; F5's goal, watch steps and second idempotent run; F6's alternation".
In `tools/sandbox/SANDBOX-FABRIC-SUITE.md`, F6 (line 189) is "Rebased-warp pull recovery" and never mentions unwire.
The alternation of `lyx fabric unwire` and `lyx fabric reconcile` is in F15 (line 356).
The card also misses F11, "Dirty-worktree matrix across the mutating verbs" (line 282).
That step runs `unwire` against a dirty worktree to check that no verb discards uncommitted work.
After card 6, a bare `unwire` there is a dry run, so the matrix's unwire cell passes without exercising the destructive path at all.
F11 spells the verb as bare `unwire`, not `lyx fabric unwire`, so the card's catch-all phrase "every `lyx fabric unwire` the suites instruct the agent to run" does not reach it either.
The operating-model paragraph of `SANDBOX-CORE-SUITE.md` (line 42) also says S6 reverses its scaffolding "with `lyx fabric unwire`", outside the S6 sites the card names.

**Fix:** Replace "F6's alternation" with "F15's alternation".
Add F11's verb list, so its matrix runs `unwire --apply` and says so.
Add the CORE suite operating-model sentence (line 42) to the enumerated sites.

### [MEDIUM] fabricengine's doc.go Unwire paragraph left stale

**Location:** `_lyx/plan/06-fabric-unwire-apply.md:34`

**Issue:** `internal/fabricengine/doc.go` (around line 464) describes `Unwire` as the per-worktree full deactivation that "removes every fabric junction actually present on disk … and their warp `.git/info/exclude` entries".
After card 6, `Unwire(cwd, false)` removes nothing and only reports.
The decision record's Constraints carry CLAUDE.md "Docs land in the same commit", and D6's scope names package `doc.go` files.
Card 7 updates `internal/boardcli/doc.go` for the same change on the board verbs, but card 6 has no `doc.go` target.

**Fix:** Add `internal/fabricengine/doc.go` to card 6's `**Edit:**` list.
State in the Intent that the `Unwire` paragraph names the `apply` parameter, and that without it nothing is removed and the result reports what would be.

### [MEDIUM] New fabriccli top-level test where the mutation scenario fits

**Location:** `_lyx/plan/06-fabric-unwire-apply.md:41`

**Issue:** Card 6 creates `internal/fabriccli/unwire_integration_test.go` with one new top-level test and raises the package's test budget to make room for it.
Most of what that test asserts is already covered by card 6's own extension of `TestUnwire_RemovesOnDiskJunctionsIncludingStale`: the dry run lists the junctions, leaves them on disk and records no mutation, and the applied run removes them.
That leaves only the CLI plumbing to cover: the `--apply` flag reaches the engine, and `"applied"` lands in the envelope.
`TestRunCLI_HubMutationScenario` in `internal/fabriccli/hubmutation_integration_test.go` exists for exactly this.
It drives the fabric CLI's mutating verbs over one `hubforge.CopyHub` hub as ordered steps, asserting each verb's envelope and exit code.
The plan template allows a `**Create:**` test file only for a surface no existing test file covers.
`PATTERN-test-economy` makes a new test function cover a behavior no existing test covers, and otherwise the case is added to an existing test.

**Fix:** Drop the `**Create:**` of `unwire_integration_test.go` and the `internal/fabriccli` test-budget raise.
Add `internal/fabriccli/hubmutation_integration_test.go` under `**Edit:**`.
Plan an unwire step in `TestRunCLI_HubMutationScenario` that asserts only the envelope.
On a pair the step adds and later removes or re-wires, it runs bare `unwire` and expects exit 0, `"applied": false` and empty `mutations`, then `unwire --apply` and expects `"applied": true`.
Place the step before the steps that damage the hub.

### [MEDIUM] Value-shape cases pinned twice in card 5

**Location:** `_lyx/plan/05-reconcile-floor-migrate.md:36`

**Issue:** Card 5 pins the floor rewrite's value shapes in two places.
The new `floor_test.go` table over `RaiseIntFloor` covers below, at, above, absent, null, env marker, non-integer, comments preserved and unparseable (line 33).
`TestReconcileAll` then gains rows for null, at-floor, `${env:...}` and `abc` as well (line 36).
Those four behaviors are decided only inside `RaiseIntFloor`, so each would be covered by two tests.
`PATTERN-test-economy`: "no two tests cover the same behavior".

**Fix:** Keep the shape table in `floor_test.go`.
Trim the new `TestReconcileAll` rows to the wiring each module adds.
Keep one below-floor row per module (`shuttle`, `orch`, `loom`): it checks a dry run reports one `Migrated` line and writes nothing, and `--apply` writes the key at the floor inside the template-shaped merge.
Drop the null, at-floor, env-marker and `abc` rows there.
The other admissible fix drops `floor_test.go` and its budget raise and keeps every shape in `TestReconcileAll`.

### [LOW] Engine doc.go files not updated for the new MigrateConfig hooks

**Location:** `_lyx/plan/05-reconcile-floor-migrate.md:34`

**Issue:** Card 5 adds an exported `MigrateConfig` to `internal/shuttleengine` and `internal/orchengine`, and `loomengine` has no `doc.go`.
`internal/batcher/doc.go` documents its own `MigrateConfig`, which this card models the hooks on.
Neither `internal/shuttleengine/doc.go` nor `internal/orchengine/doc.go` is a target, although D5's scope lists package `doc.go` files among the docs the change touches.

**Fix:** Add `internal/shuttleengine/doc.go` and `internal/orchengine/doc.go` to card 5's `**Edit:**` list.
Each gets one sentence saying that `MigrateConfig` raises the module's below-floor key to its floor for reconcile.

## Focus directive verdicts

- **Cards 1 and 2 overlap:** not a defect.
  Card 1 changes only the darn stencil's Step 1 `board get` line, the landing describe stencil's "The board entry, via …" bullet, element 0 of `TestDarnSpec`'s first-spawn `wantInPrompt` and the `wordingClaims` darn row.
  Card 2 changes the darn stencil's Step 4 spec path, the landing stencil's parenthetical spec name and element 2 of the same `wantInPrompt` (`internal/darnengine/spec_test.go:57`).
  The two cards touch different lines, and card 1 depends on no later card.
- **Attack surface (cards 6, 7, 9):** each Intent states its bound.
  Cards 6 and 7 say `--apply` restores today's behaviour and the dry run reads state only.
  Card 6 keeps bare `unwire` out of `FabricReaderVerbs`, and card 7 keeps the `retire-legacy` refusal in both modes.
  Card 9 names the window the checks leave open (R2).
- **New top-level tests (cards 5, 6, 9):** card 6 is F3, and card 5's duplication is F4.
  Card 9's new test covers concurrent-writer detection, which no existing test covers.
  `TestWriteCarryOver_RefusesWithoutWriting` asserts the record is unchanged afterwards, which does not hold when another writer changed it, so no finding.
- **Re-sign arrows and member glyphs (cards 5, 9):** card 5 re-signs no existing member, and its new functions are `Create` heads.
  Card 9 carries arrows on `writeCarryOver` and `replaceFile`, and `WriteCarryOver`'s signature does not change.
  Neither card lists a bare Go file where its Intent names a symbol.
- **Fidelity:** D1–D8, the constraints and A1–A5 are each carried by a card, with the gaps in F1 and F2.
  R1–R4 need no card, and the carry-over entry in Open risks is not part of the answer key.
  Two pieces of scope go beyond the record's wording but are not findings, because each is a supporting detail of its decision.
  `RaiseIntFloor` in `yamlengine` is the shared rewrite for D5's three hooks.
  Card 4 exports `TerminalOpenedFile` so battencli tests can plant the marker without re-spelling its suffix.
- **Granularity (cards 2, 8):** card 2's embed `Create` cannot be separated from the rename and registry entry, so the card is one reviewable unit.
  Card 8 bundles the annotation, the renderer threading, the board annotation and the live-tree guard for one rule, so it is one reviewable unit too.
- **verify:** `go test ./...` and `go test -tags integration ./...` cover every package a card targets.
  `tmux` and `llm` are compiled only, which the rubric allows, and the section follows the template's order.
