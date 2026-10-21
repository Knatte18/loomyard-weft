---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: PATTERN.md:11
    summary: >-
      Card 1's PATTERN.md edit silently drops a clause from the PATTERN-durable-vs-ephemeral-state index line, which the card says is unchanged and no declared change covers
    fix: >-
      In PATTERN.md restore the PATTERN-durable-vs-ephemeral-state index line to its pre-branch text, which ends "same subpath; nothing a run needs lives only under `.lyx`." followed by the unchanged background link (copy it from `git show 0ffd168b:PATTERN.md` line 11, which main still carries).
    origin: lens:code-pattern-fit
  - id: F2
    severity: BLOCKING
    class: design
    location: PATTERN.md:32
    summary: >-
      Card 1's PATTERN.md edit silently drops the detached-push clause from the PATTERN-push-both-sides index line, an entry no card or declared change touches
    fix: >-
      In PATTERN.md restore the PATTERN-push-both-sides index line to its pre-branch text, which ends "the pair's code branch goes along, and a per-transition status push goes through fabric's detached push." followed by the unchanged background link (copy it from `git show 0ffd168b:PATTERN.md` line 32, which main still carries and `pattern/PATTERN-push-both-sides.md` line 7 still states).
    origin: lens:code-pattern-fit
  - id: F3
    severity: LOW
    class: design
    location: internal/gateslot/config.go:368
    summary: >-
      LoadConfig names the local override for removal whenever the file exists, though configengine drops an empty or comment-only local file, so the error can send the operator to remove a harmless file
    fix: >-
      In `LoadConfig` set `local` only when the local file holds units, for example by reading it and checking `yamlengine.LeafPaths` is non-empty, or by taking it from the layers `configengine.Layers` returns, so an empty or comment-only local file leaves `local` empty and the error carries no `remove <local>` step.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/hubgeom/gateslots.go:24
    summary: >-
      The GateSlots wrapper naming the machine file after a failed limits read, and gatecli's reworded way forward in run.go, have no test that fails if either is reverted
    fix: >-
      In `TestWebsterGeometry` (internal/hubgeom/webstergeom_test.go) add an assertion that `GateSlots.Limits()` over a machine file holding `go_parallel: 0` fails with an error naming both the board's gate.yaml and the machine file as `load <hub> and <machine>`. In `TestGateTest_Scenario` (internal/gatecli/test_integration_test.go) add `take the step it names, then re-run the same command` to the machine-row `wantError` list.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/gateslot/config_test.go:131
    summary: >-
      No TestLoadConfig row covers the two-file way forward with a local removal clause, nor a local file whose anchor has no _lyx that must leave the clause out
    fix: >-
      In `TestLoadConfig` add a row with a machine `go_parallel: 0` file and a local file holding `slots: 9` under an anchor with `_lyx/`, asserting the error holds `; remove <local>, which takes no gate key` ahead of the role clause; add a row with the same files under an anchor with no `_lyx/`, asserting the error carries no `remove` clause.
    origin: lens:code-tests
  - id: F6
    severity: NIT
    class: design
    location: internal/configengine/set.go:27
    summary: >-
      The layerPath owner change for LayerMachine is not pinned, since every machine row in TestSet and TestEdit also has an anchor holding _lyx and would pass with the anchor as owner
    fix: >-
      In `TestSet` add a `LayerMachine` row whose board dir has no `_lyx/` beside an anchor that has one, wanting the refusal that the board dir is not initialised, and leaving no file.
    origin: lens:code-tests
---

### [BLOCKING] PATTERN-durable-vs-ephemeral-state index line loses a clause

**Location:** PATTERN.md:11
**Issue:** Commit `42b798c6` rewrites the entry from "...same subpath; nothing a run needs lives only under `.lyx`." to "...same subpath.".
Card 1 says this entry's index line is unchanged, and the declared PATTERN changes list only `PATTERN-config-layers` and `PATTERN-hub-wide-config`, so the entry is judged by its current text.
The background file `pattern/PATTERN-durable-vs-ephemeral-state.md` still states the dropped rule at line 6, and main's `PATTERN.md` still carries it.

### [BLOCKING] PATTERN-push-both-sides index line loses a clause

**Location:** PATTERN.md:32
**Issue:** The same commit cuts ", and a per-transition status push goes through fabric's detached push" from this entry.
No card or declared change touches it, and `pattern/PATTERN-push-both-sides.md` line 7 still binds the rule, so the diff weakens a cross-cutting invariant the task never mentioned.

### [LOW] Local removal step named for a local file the load ignores

**Location:** internal/gateslot/config.go:368
**Issue:** `LoadConfig` sets `local` when `os.Stat` of the local file succeeds, but `readLayers` drops an empty or comment-only local file.
For a machine `go_parallel: 0` beside such a file the error still says `remove <local>, which takes no gate key`, a step that cannot fix the failure.

### [LOW] Two changed behaviours of card 2 have no test

**Location:** internal/hubgeom/gateslots.go:24
**Issue:** `TestWebsterGeometry` asserts only successful limits, and the `TestGateTest_Scenario` machine row's path assertions are satisfied by `withConfigWayForward`'s own text, so the wrapper's `named += " and " + file` loop is unpinned.
No `wantError` holds the new `runTest` clause `take the step it names, then re-run the same command`.

### [LOW] Local-removal clause of the two-file form is untested

**Location:** internal/gateslot/config_test.go:131
**Issue:** The local rows in `TestLoadConfig` have no machine file, so the two-file form never carries `remove <local>`.
No row has a local file under an anchor with no `_lyx/`, where `local` must stay empty because the load never stacks that file.

### [NIT] layerPath owner mapping for the machine layer is not pinned

**Location:** internal/configengine/set.go:27
**Issue:** Every machine row in `TestSet` and `TestEdit` uses an anchor holding `_lyx`, so each would pass with `owner = dirs.Anchor`.
The card's reason for the `hubWide` row field was to check the board dir's `_lyx/`, which only a row with a `_lyx`-less board dir beside an initialised anchor exercises.

## Rejected

- Test economy overlaps between the `TestGateTest_Scenario` machine row and `TestLoadConfig`'s two-file row, between `TestWebsterGeometry` and `TestLoadConfig`, and between the `TestDispatchLayers` guard rows and the integration `guardRows` row, raised as a check by the code-tests lens: each is mandated by the plan and pins a distinct wiring layer, and the lens itself found no duplicate.
- `configengine/doc.go` adds a `# The machine rule` section while the card asked the `# The local rule` section to name the machine layer too, noted by the code-affected-set lens: the local rule section stays accurate and the machine layer's rule is documented in full, so there is no wrong consumer.
- `TestValidateLocal`'s doc comment reading awkwardly now that it holds `ValidateMachine` rows, noted by the code-tests lens: naming only, and the card prescribes the rows there.
- `buildConfigLong` sitting about five bytes under `maxLongBytes`, noted by the code-affected-set lens: `TestConfigLong` pins the limit and the card computed the margin, so nothing is wrong.
