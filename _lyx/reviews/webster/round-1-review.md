---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/configsync/configsync.go:primeSeedError, internal/configsync/movekeys.go:122
    summary: primeSeedError appends an "also delete the  line" step for any ErrInvalid seed failure, including a module with no overlay selector
    fix: In primeSeedError return err unchanged when m.Spec().Overlay.Selector is empty, before the errors.Is check; add one row to the existing prime-seed table showing a non-overlay ErrInvalid failure keeps its original message.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: scope
    location: internal/configengine/set_test.go, internal/configengine/edit_test.go
    summary: No row shows a hub worktree Set or Edit that writes the selector with the default file's value succeeding; only Load pins the equal-value branch
    fix: Add one row to the existing hub-worktree table in set_test.go and one in edit_test.go where the worktree write sets the selector to the value the default file holds and succeeds.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: scope
    location: internal/configengine/set.go:writeTemplate
    summary: The layer-above-base branch through TemplateUnder has no row where an unknown selector value in the base makes the write fail before anything is written
    fix: Add one row to the existing Set table in set_test.go where the default file holds an unknown selector value and a worktree or local write is refused with nothing written.
    origin: lens:code-tests
  - id: F4
    severity: NIT
    class: design
    location: internal/lenses/config.go:applyRemove
    summary: A name listed twice in remove that matches no lens is appended to skipped twice, so the warning repeats it
    fix: In applyRemove skip a name already present in skipped before appending it, and add a repeated unmatched name and a repeated matched name to the existing remove subtest.
    origin: lens:code-correctness
---

### [LOW] primeSeedError misreports a non-overlay seed failure

**Location:** internal/configsync/configsync.go:primeSeedError, internal/configsync/movekeys.go:122
**Issue:** `primeSeedError` fires on any `ErrInvalid` from `sparseSeed`. For a module with an empty `Overlay.Selector` the message gains "also delete the  line from <prime path>", a step that does not apply.

### [LOW] Hub worktree write with the default's selector value is untested

**Location:** internal/configengine/set_test.go, internal/configengine/edit_test.go
**Issue:** The branch of `worktreeSelectorRefusal` that allows a value equal to the default's is pinned for `Load` only. `Set` and `Edit` have refusal rows but no success row for this branch.

### [LOW] writeTemplate's layer-above-base failure path is untested

**Location:** internal/configengine/set.go:writeTemplate
**Issue:** The doc promises that an unknown selector value fails before anything is written, but no row reaches the `TemplateUnder` error for a layer above the base.

### [NIT] Repeated unmatched remove name is warned twice

**Location:** internal/lenses/config.go:applyRemove
**Issue:** The duplicate skip applies to matched names only. An unmatched name listed twice enters `skipped` twice, and the repeat has no test row.

## Rejected

- code-tests F1 (the `go` layer is not pinned byte-exact against the old prose): I compared `sets/go.yaml` with the removed `template.yaml` lines and the five lenses are verbatim. A golden of the old prose would be a fragile test, and the focus question is answered without it.
- code-correctness NIT on `scaffoldWithoutPerNameEntries` reading the raw template in `Edit`: harmless today, since the overlay touches only per-name maps, which are emptied anyway. It needs no change.

## Holistic notes

- PATTERN: all three declared entries land with the declared text. `PATTERN.md` goes from 15703 to 15693 bytes, so the overview ends smaller. No commit carries a `Pattern-Departure:` trailer.
- The `go` layer is the five Go-worded lenses verbatim, `remove` applies in `lenses.Load` with the emptied-fan refusal and the once-per-process warning, and `lenses` imports only stdlib and `configengine`.
- Every judging path in `configengine` and `configsync` uses the effective template. The remaining base-template uses are the key-move scaffolds and the seed-only paths, which the `configsync` doc names.
