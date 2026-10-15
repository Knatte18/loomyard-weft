---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/plan/02-scope-guard-file-package-cover.md:25
    summary: Card 2 prescribes an empty-files guard in symbolFilesCovered and two Modified rows whose inputs quarry never emits, since compareTables builds a Modified entry only when both sides hold occurrences
    fix: >-
      In card 2's Intent, drop ", and false for an empty `files`" from the `symbolFilesCovered` sentence (line 25) and drop "; a Modified entry with neither side has no files and is reported as today" from the Modified bullet (line 32); drop the two rows "package self glyph `sub#`, Modified `sub#Foo` with only `After` in `sub/a.go`: none" (line 57) and "package self glyph `sub#`, Modified `sub#Foo` with no `Before` or `After`: one finding with an empty `File`" (line 70). The Before-and-After row at line 56 and the two mirror Modified rows keep the Modified shape covered.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/02-scope-guard-file-package-cover.md:69
    summary: Card 2's no-scope skip is reached only by the plan:nohash row, a target the plan gate's handle-malformed check refuses before any plan reaches ScopeGuard
    fix: >-
      In card 2's Intent, end the RefScope clause at "then maps the ref through `planparser.RefScope(nil, t)`" by dropping ", skipping a ref with no scope" (line 26), and drop the row "unmappable handle `plan:nohash`, Created `sub#Bar` in `sub/a.go`: `sub/a.go`, the row that reaches the no-scope skip" (line 69). A ref RefScope cannot map yields no scope that equals a file or a `path.Dir`, so it covers nothing without a prescribed branch.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/01-planparser-ref-member.md:18
    summary: Card 1 changes refMember's behavior, a new nil-plan default, under a Rename label alone, whose operation is an AST rename plus grep verify and carries no Edit check for a body change
    fix: >-
      In card 1, add `plan:internal/planparser#RefMember` as the first bullet of the `**Edit:**` group, so the nil-plan addition the Intent prescribes is typed as an Edit beside the Rename; validate-plan --overlay admits this target list with no new finding.
---

### [LOW] Unreachable empty-files guard and one-sided Modified rows

**Location:** `_lyx/plan/02-scope-guard-file-package-cover.md:25`, with lines 32, 57 and 70.
**Issue:** Card 2 has `symbolFilesCovered` return false for an empty `files`, says a Modified entry with neither side is reported as today, and adds a row for a Modified symbol with only `After` and one with neither side.
quarry's `compareTables` (`internal/engine/delta.go` in quarry v0.2.1) puts a key into `modified` only in its `default` case, after the `len(beforeOccs) == 0` and `len(afterOccs) == 0` cases have sent it to `created` or `deleted`, so every `ModifiedSymbol` it emits holds both a `Before` and an `After` occurrence.
A Created or Deleted symbol always carries its own `File`.
No real delta reaches the empty-files branch or the one-sided shapes, so the card prescribes a guard against a state the upstream rules out and two rows that pin it.

### [LOW] No-scope skip reached only by an input the plan gate refuses

**Location:** `_lyx/plan/02-scope-guard-file-package-cover.md:69`, with line 26.
**Issue:** The `plan:nohash` row is named as the row that reaches the no-scope skip.
`handle-malformed` in the plan spec's validation checks refuses a handle-shaped entry whose text after `plan:` carries no `#`, and `ScopeGuard` runs over the cards of an approved plan, so no target reaching it is unmappable.
`RefScope` maps every other ref shape a gated plan can hold, so the skip has no reachable input.

### [LOW] Card 1 types a behavior change as Rename only

**Location:** `_lyx/plan/01-planparser-ref-member.md:18`.
**Issue:** The Intent keeps `refMember`'s body "with one addition: a nil plan reads as the default language", a change to what the function does on a nil plan, which `TestRefMember`'s nil-plan row pins.
The card types it only under `**Rename:**`, whose spec row is an AST-aware rename plus a grep verify; the plan spec's multi-label rule runs each group's check against that group's own targets, so no `Edit` group covers the body change.
The overlay with `plan:internal/planparser#RefMember` added to the `Edit` group reported `ok: true`, the same as the `{}` baseline.

## Rejected

- Card 1's `TestRefMember` duplicating `TestRefFile` or `TestRefCovers`: not a defect. `TestRefFile` asserts a ref's file and `TestRefCovers` a boolean cover verdict; neither observes the canonical member string `RefMember` returns, a non-Go plan's refusal, or the nil-plan default, which `RefCovers` never reaches because it replaces a nil plan before calling `refMember`.
- Card 1's `Edit` of `RefCovers`: correct. The rename rewrites `RefCovers`'s two call sites, a Modified symbol the scope guard would otherwise report, and the card's `ImpactSummary` names that blast radius.
- Card 2's re-sign arrow: correctly absent. `ScopeGuard` keeps `func ScopeGuard(cards []planparser.Card, delta quarry.GitDeltaAnswer) []Finding` and `targetGlyphUnion` changes only its doc comment; the `report` closure is local, not a member.
- Card 2's rows otherwise: each other new row reaches a distinct input class at `ScopeGuard`'s surface (file self glyph, plain file path, package self glyph, directory path, subdirectory for both package forms, the four one-file-uncovered Modified and Renamed directions, two file targets jointly covering one symbol, a file-unit member, a member handle), and the three carried-over rows replace the three deleted single-case tests one for one. `TestScopeGuard_DeletedAndModifiedAlsoChecked`, the Renamed tests and `TestScopeGuard_FileLevelCoverage` cover what the table does not.
- Card 2's test budget: `internal/planglyph` holds 87 top-level tests against its row of 87, and the card adds one and deletes three. Card 1 raises `internal/planparser` from 95, its current count and row, by one.
- Card 3's attack surface: the Intent names each sanctioned deviation with its bound (the guard step's named file, the guard-fix line's file and card, the self-fix section's two-pronged test and attempt cap) and closes every other change outside the targets.
- Card order and fidelity: card 2 uses card 1's `RefMember` and follows it; cards 3 and 4 depend on no code. D1 is carried by card 2 (with card 1 for the `PATTERN-ref-shape-registry` constraint), D2 by card 3, D3 by card 4, and the doc-comment, semantic-line-break, stencil-ownership and test-economy constraints by the cards that touch those files. No card adds scope the decision record does not license.
- `## verify:`: `go test ./...` and `go test -tags integration ./...` run every targeted package, `internal/planparser`, `internal/planglyph`, `cmd/lyx` and the `contracts/` stencil and spec packages included, in the order the plan spec's verify model states.
