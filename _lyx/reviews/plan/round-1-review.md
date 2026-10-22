---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/03-lenses-lens-set.md (ImpactSummary, Edit list); _lyx/plan/06-lenses-remove.md (Edit list)
    summary: The new lens_set and remove fields leak into the burler.yaml fixtures that lensesFixture marshals, so the TestReconcile_MovesKeysAcrossModules rows asserting board:burler absent fail at card 3's gate, and card 3's ImpactSummary claims the opposite
    fix: "Card 3: add `internal/configsync#lensesFixture` under **Edit:**, and in Intent say the fixture leaves the new `LensSet` field out of the marshalled document (for example by marshalling a copy holding only `lenses` and `fans`), so a burler.yaml built from it holds only the two keys the burler-to-lenses moves take out. Rewrite the ImpactSummary's lensesFixture clause to say this. Card 6: add `internal/configsync#lensesFixture` under **Edit:** and extend the same rule to `Remove`. Checked with `lyx loom validate-plan --overlay`, no new finding."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/05-configsync-effective-template.md (Tests, TestReconcile_MovesKeysAcrossModules board row)
    summary: The new board row runs ReconcileHubWideAt without StripBase, so stripMovedUnits' result is only reported, and the row's excludes concurrency is satisfied by DropTemplateEqualNames; the row passes even if stripMovedUnits still judged against the base template
    fix: "Card 5: give the `board` row an assertion on the lenses Result's `Stripped` holding `lenses.concurrency`, or a new row field that runs the call with `StripBase: true`. State in Intent which of the two pins `stripMovedUnits`' judgment under the file's own `lens_set`."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/05-configsync-effective-template.md (Intent, the unknown-selector paragraph)
    summary: The claim that an unknown selector fails reconcile naming the file holding the value is false on two paths, and the SeedPrime path's way forward omits a step
    fix: "Card 5: rewrite the unknown-selector paragraph to cover every path that composes the template from a file other than the one reconciled. On `reconcileSparseFile` (a hub worktree or local file pruned against `TemplateUnder`), name the base layer's file. On `moveKeys`' `loadTarget` seed through `sparseSeed`, name the prime's copy, as the `SeedPrime` branch does. On both prime paths, say the way forward is to delete the `lens_set` line from the prime's copy and then set the base. Add a `TestReconcile_FailureNamesItsFile` row for the hub `ReconcileAll` path, with the bad value in the board file, wanting the board file's path."
  - id: F4
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/01-configengine-overlay-load.md (Intent, the refusal's bound sentence)
    summary: The refusal's stated bound omits that a base-layer write (setHubWide validates the board file alone) can turn an admitted equal worktree value, such as lens_set an empty string, into a refused one for every load of that worktree
    fix: "Card 1: extend the bound sentence. A base write that changes the selector is not checked against any worktree file, so a hub worktree file holding the old base value is refused at its next load, naming the deletion of its `lens_set` line."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/06-lenses-remove.md (Intent, ResolveFan refusal; ImpactSummary)
    summary: The emptied-fan way forward puts first a step (point the fan key at another fan) that cannot help a burler half resolving a fan name recorded at spawn time, and the ImpactSummary claims no consumer changes
    fix: "Card 6: reword the refusal's way forward so dropping a name from `remove` comes first, then pointing the loom or darn fan key at another fan or an empty string for a later run. In the ImpactSummary, say burler's `resolveRecordedFan` appends its own define-the-fan way forward to this refusal."
  - id: F6
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/03-lenses-lens-set.md (Create internal/lenses/testdata/go-lenses.yaml, golden assertion); _lyx/plan/04-lenses-language-layers.md (Tests)
    summary: The golden file duplicates sets/go.yaml's prose and only pins static prose; card 3's own per-layer check (each layer lens resolves to the layer's prose) covers the behaviour, and card 3's equality with the base pins the verbatim move at its own gate
    fix: "Card 3: drop the `internal/lenses/testdata/go-lenses.yaml` Create bullet, its Intent paragraph and the golden assertion. Keep the in-card check that the go layer's five lenses equal the base template's. Card 4: assert each of the five base lenses differs from `lensSets()[\"go\"]`'s prose instead of the golden file's. Checked with `lyx loom validate-plan --overlay`, no new finding."
  - id: F7
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/02-configengine-overlay-writes.md (Tests, TestEdit wantSawScaffold and the absent standalone base row)
    summary: wantSawScaffold compares Edit's scaffold with the BaseScaffold function Edit already calls, which is tautological; the emptied per-name maps are pinned by card 2's TestSet scaffold row, and Edit's BaseScaffold wiring by TestEdit's existing absent-base wantSaw rows
    fix: "Card 2: drop the `wantSawScaffold` field and the TestEdit row for an absent standalone base. If an Edit row is kept, assert the editor's first view with the existing `wantSaw` field against literal bytes holding `lenses: {}`."
  - id: F8
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/01-configengine-overlay-load.md (Tests, TestLoad_LayerStack local-file row)
    summary: "The row with a local file holding pick: x repeats the guard-key local refusal that TestValidateLocal's guard category row and lenses TestLoad's local-entry row already pin; its lacks-only-the-base-file check tests a state the worktree-only selector check cannot produce"
    fix: "Card 1: drop the local-file `pick: x` row from TestLoad_LayerStack, and the `refusalLacks` field if no other row uses it."
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/03-lenses-lens-set.md (Tests, emphasis-line check)
    summary: The exemption for a layer lens in a review-* or fix-* fan, read against the layer-resolved fans, exempts csharp's numerics (which csharp adds to review-code), so D2's rule that every layer lens keeps the emphasis posture goes unchecked for the one lens a layer adds
    fix: "Card 3: compute the exemption from the base template's review-* and fix-* fans only, so each lens a layer defines but the base lacks must end with the emphasis line. Card 4: say that numerics ends with it."
  - id: F10
    severity: NIT
    class: design
    origin: lens:way-forward
    location: _lyx/plan/01-configengine-overlay-load.md (Intent, selectorValue)
    summary: The non-scalar selector refusal names the key but no way forward, unlike every other refusal the plan adds
    fix: "Card 1: give the non-scalar `selectorValue` error the same way forward as the unknown-value error: set the selector to a layer name or an empty string with `--layer default --set` from the prime in a hub, or `--set` standalone. Add that substring to the `pick: [x]` row."
---

### [BLOCKING] New Config fields leak into the burler.yaml move fixtures

**Location:** `_lyx/plan/03-lenses-lens-set.md` (ImpactSummary, Edit list); `_lyx/plan/06-lenses-remove.md` (Edit list)

**Issue:** `lensesFixture` (`internal/configsync/configsync_test.go:41-53`) decodes the template into `lenses.Config` and marshals the whole struct.
After card 3 it emits `lens_set: ""`, and after card 6 also `remove: []`.
`TestReconcile_MovesKeysAcrossModules` writes `burlerToday` and `burlerOldTemplate` as `board:burler` (`configsync_test.go:1587-1594`, used by the rows at 1750-1800).
The moves take out only `lenses` and `fans` (`internal/configreg/keymove.go:24-25`).
`moveFile.write` deletes a source only when it is left with no keys, and a retired module's file has `keepHeader` false (`internal/configsync/movekeys.go:31-32`, `276-293`).
So `burler.yaml` survives holding `lens_set`, and the rows asserting `absent: board:burler` fail at card 3's gate.
Card 3's ImpactSummary says these rows keep their results.
That holds only where the fixture is a lenses file.
No card lists `lensesFixture` or a covering test edit before card 5.

### [MEDIUM] Card 5's move row does not pin stripMovedUnits' judgment

**Location:** `_lyx/plan/05-configsync-effective-template.md` (Tests)

**Issue:** The `board` call runs `ReconcileHubWideAt(..., HubWideOptions{Apply: !tt.dryRun, SeedDefaults: true})` with no `StripBase` (`configsync_test.go`, the test loop near line 1842).
`stripMovedUnits`' bytes are therefore only reported, never applied (`configsync.go:888-899`).
The row's `excludes concurrency:` is met by the following `DropTemplateEqualNames`, which card 5 also moves to the effective template (`configsync.go:916-918`).
A `stripMovedUnits` left on the base template would still pass.

### [LOW] Unknown-selector reconcile errors name the wrong file on two paths

**Location:** `_lyx/plan/05-configsync-effective-template.md` (Intent)

**Issue:**
- On a hub `ReconcileAll`, `reconcileSparseFile` prunes against `TemplateUnder`, and its `fail` names the worktree or local path (`configsync.go:530-533`), while the value sits in the board file.
  `overlayTemplate`'s message names no file.
- `moveKeys`' `loadTarget` seeds through `sparseSeed` from the prime (`movekeys.go:118-122`) and names the board target.
- On the `SeedPrime` path, following the hub way forward (set the base) leaves `lens_set: nope` in the prime's worktree file.
  That file is then refused by card 1's selector refusal, a step the first message never named.

### [LOW] The refusal's bound omits base-side writes

**Location:** `_lyx/plan/01-configengine-overlay-load.md` (Intent)

**Issue:** Card 2 guards only worktree-layer writes, and `setHubWide` validates `Dirs{Board}` alone (`internal/configcli/hubwide.go:349-364`).
Under the equal-value carve-out, a hub worktree file may hold `lens_set: ""`.
Setting the base to `go` then makes every load of that worktree refuse.
The load refusal still catches this and names the deletion step, so the effect is bounded, but the Intent's stated bound does not say so.

### [LOW] Emptied-fan way forward is wrong-first for burler's recorded fans

**Location:** `_lyx/plan/06-lenses-remove.md` (Intent, ImpactSummary)

**Issue:** `resolveRecordedFan` (`internal/burlerengine/profile.go:196-203`) resolves a fan name recorded at spawn time and appends its own way forward.
Re-pointing a loom or darn fan key does not change a recorded name, so the refusal's first option does not progress for a resumed round.
The ImpactSummary says no consumer changes.

### [LOW] Golden file duplicates the go layer's prose

**Location:** `_lyx/plan/03-lenses-lens-set.md`; `_lyx/plan/04-lenses-language-layers.md`

**Issue:** `testdata/go-lenses.yaml` is a second copy of `sets/go.yaml`'s prose.
After card 4 its only assertion is that two committed copies of static prose agree.
Card 3's own per-layer check already pins that each go lens resolves to the layer's prose, in `TestConfigTemplate_DecodesAndIsSelfConsistent`.
The verbatim move is a one-time property, pinned by card 3's equality with the base at card 3's own gate.

### [LOW] TestEdit's wantSawScaffold is circular

**Location:** `_lyx/plan/02-configengine-overlay-writes.md` (Tests)

**Issue:** `Edit` already scaffolds an absent base with `BaseScaffold(dirs, spec)` (`internal/configengine/edit.go`, `Edit`), so comparing the editor's view with `BaseScaffold`'s return proves nothing about its content.
Two existing tests already cover the rest:
- The emptied per-name maps are pinned by card 2's TestSet row for an absent standalone base, since `Set` scaffolds through the same `BaseScaffold`.
- The wiring is pinned by TestEdit's existing `wantSaw` rows for the hub base (`edit_test.go:63`) and the seed-only base (`edit_test.go:73`).

### [LOW] Local-file guard row repeats existing coverage

**Location:** `_lyx/plan/01-configengine-overlay-load.md` (Tests)

**Issue:** A local guard key is refused by TestValidateLocal's `guard category` row (`internal/configengine/layers_test.go:536`), and a refused local file fails the load in lenses TestLoad's `a local lenses entry fails the load naming the local file` (`internal/lenses/config_test.go:93`).
The row's `refusalLacks` check guards a state the design cannot produce: card 1's selector check reads only the worktree document.

### [LOW] Emphasis-line exemption lets numerics escape

**Location:** `_lyx/plan/03-lenses-lens-set.md` (Tests)

**Issue:** D2 adds `numerics` to csharp's `review-code`.
An exemption keyed on the review-* and fix-* fans of the loaded configuration treats `numerics` as a procedure lens, so its emphasis line is never checked.
D2 says every layer lens keeps the emphasis-never-exclusion posture, and the test's current `procedureLenses` set is built from the base fans (`internal/lenses/config_test.go:194-207`).

### [NIT] Non-scalar selector refusal names no way forward

**Location:** `_lyx/plan/01-configengine-overlay-load.md` (Intent)

**Issue:** `selectorValue`'s non-scalar error is the one new refusal with no next step.

## Rejected

- WarnSkippedEntries names no way forward (lens:way-forward): it is a warning for a harmless skip that D5 defines, not a refusal, and D5 asks for the file and entries only.
- Card 3's `fingerprintOf` change as unlicensed scope: D4 makes the overlay part of the template every reconcile judges against, and the build stamp exists to re-run reconcile when that template changes (`PATTERN-binary-change-reconcile`), so it is supporting scope.
- Removing PATTERN.md's intro line as unlicensed: both `Added after Discussion` rationales size the cuts against the 171-byte line and note the intro alone saves only about 82 bytes. The two clauses save 99 bytes, so all three cuts are needed (181 at least 171), and the overview shrinks from 15703 bytes.
- `TemplateUnder` as a second exported composer, against D4's "one function": it reads the base file and delegates composition to `EffectiveTemplate`, which keeps layer-file reading inside configengine (`PATTERN-config-layers`).
- `PATTERN-config-layers` keeping the backticks around `configengine` against the declared text: the card cites the departure, it is typographic, and no mechanical check compares the line to the declaration.
- The header comments of `set.go`, `edit.go` and `configsync.go` sit outside every member span: webster's scope guard is file-level and informational (`internal/websterengine/digest.go:37`), and listing the files beside their members is refused by `redundant-file-target`.
- Re-sign arrows: `readLayers`, `load`, `BelowLayer`, `Set` and `fingerprintOf` keep their signatures. `setSparse`, `reconcileSparseFile` and `stripBase` carry arrows, and their callers sit on the same card.
- Card order: each card uses only symbols and layers from earlier cards. Card 4 drops card 3's go-equals-base assertion after card 3 runs, and cards 5 and 6 use the csharp layer from card 4.
- Fidelity: D1 to D6, the three `Added after Discussion` entries and the four Constraints are each carried by a card. No card edits the task's board entry, carries a research step, or has a Verify line.
