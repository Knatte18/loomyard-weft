---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/shedrecipe/entries_implementer.go:19
    summary: The implementers table is a second map[string]Constructor in internal/shedrecipe, which PATTERN-shed-recipe-registry limits to one, and PATTERN.md and the registry.go header are not amended
    fix: In the same diff, amend the PATTERN-shed-recipe-registry entry in PATTERN.md to allow the closed implementer table behind the Implementer entry, and correct the registry.go file header and registry comment that call the registry the single map every engine name is declared in. Alternatively fold the choice into the one registry map. Keep the Names and Lookup surface unchanged.
    origin: lens:code-pattern-fit
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shedrecipe/entries_implementer_test.go:25
    summary: No test shows that the other row Config keys reach the webster constructor and that implementer_setting is stripped; the success test passes only implementer_setting
    fix: Extend TestImplementerEntry_WebsterBuildsTheWebsterProducer, or add rows to it, with a Config holding implementer_setting plus a gates key that must build, and one holding an extra unknown key that must be refused by the constructor.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/loomengine/template.yaml:8
    summary: No test pins the implementer default webster or its category none in loomengine, so dropping either breaks no test
    fix: Add one row or assertion to the existing loomengine config test that loads the template and expects implementer to read webster, and that a local-layer value is accepted.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/shedrecipe/entries_implementer.go:441
    summary: A non-string implementer_setting is refused by configString with no way forward, and an empty one is reported as absent; no test reaches either
    fix: In implementerEntry, refuse a non-string implementer_setting value with a message naming the way forward (set it to a module.key string such as loom.implementer), and word the empty-string case as empty rather than absent. Add rows for both to TestImplementerEntry_RefusesAnUnusableSetting.
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: scope
    location: internal/creelengine/deliver_test.go:494
    summary: The creel hold-test probe-count change is unrelated to the implementer-seam board entry and is not licensed by it
    fix: Remove the deliver_test.go hunk from this diff and the creel bullet from the change description, and land the flake fix under its own task.
    origin: handler
---

### [BLOCKING] Second map[string]Constructor against PATTERN-shed-recipe-registry

**Location:** internal/shedrecipe/entries_implementer.go:19
**Issue:** PATTERN.md allows one `map[string]Constructor` in `internal/shedrecipe`.
The diff adds `implementers` as a second one and removes `Webster` from the registry, so `websterEntry` is reachable only through it.
`registry.go` still calls its map the single place every engine name is declared.
The decision record cites this PATTERN entry as met, but the diff changes neither the entry nor the header.

### [MEDIUM] Config pass-through untested

**Location:** internal/shedrecipe/entries_implementer_test.go:25
**Issue:** `implementerEntry` clones the row Config, deletes `implementer_setting` and forwards the rest.
The success test passes only the setting key, so the forwarding is never exercised through `Implementer`.

### [LOW] Template default and category untested

**Location:** internal/loomengine/template.yaml:8
**Issue:** The board entry asks for `implementer: webster` with category none.
No test fails if the template line or the `ConfigSpec` entry is dropped.

### [LOW] Non-string implementer_setting refusal has no way forward

**Location:** internal/shedrecipe/entries_implementer.go:441
**Issue:** `configString` refuses a non-string row value without a `way forward:`, and the empty string is reported as absent.
No test reaches either case.

### [LOW] Unrelated creel test change

**Location:** internal/creelengine/deliver_test.go:494
**Issue:** The board entry covers the implementer seam only.
The diff also changes a creel scheduler test and the change description claims it as a flaky-test fix.

## Rejected

- Stale `"Webster": websterEntry` in `entries_gate_test.go:30` (lens:code-pattern-fit): the map is keyed by constructor for gate tests and calls `websterEntry` directly, which still exists.
- Rename of `TestBatchifierEntry_RelativeAnchorPath` (lens:code-tests): a behaviour-preserving move to `loomPreflightEntry`, not a defect.
- Leftover `Batchifier` mentions in `docs/research/` and `docs/benchmarks/` (lens:code-affected-set): measurement reports of past runs.
