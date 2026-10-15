# Decision record — plan-test-file-path

## Goal

A plan card never lists a plain `_test.go` path under `**Edit:**`.
Tests a card adds to an existing package are `**Create:**` declarations with `plan:` handles, tests it changes are member glyphs under `**Edit:**`, and the plan gate refuses a plain test-file path under `**Edit:**` with its way forward.
The card then names no file for a new test: the handle names the package, and the fork places the test (D1).

## Scope

In:

- `contracts/stencils/loom/loom-template-plan.md`: drop the test-file exception from the `**Edit:**` rules, rewrite the paired rules that an implementation card lists its glyphs "beside the existing test file that covers its surface" and that a `**Create:**` test file appears only "for a surface no existing test file covers", add D2's rule, and replace the skeleton's `path/to/list_test.go` `**Edit:**` bullet.
- `contracts/specs/loom-plan-spec.md`: a new validation row for the gate check, its name in the validation-check preamble beside the other out-of-entry-point checks, the shape-classifier sentence that limits `Glyph.UnitPath` to the two disk-existence checks (D4), and its grammar examples that list a `_test.go` file under `**Edit:**`.
- The new plan-gate check, its wiring into the plan index's format and rework validation, its tests, and the `doc.go` of each package it touches.
- Any existing test, stencil test included, that this task's edits break.

Out:

- `**Uses:**`, `**Delete:**`, `**Rename:**`, `**Move:**`, `**Prosa:**` and `**Custom:**` groups: a `_test.go` path stays legal there.
- Non-test `.go` file paths under `**Edit:**`: the board entry does not ask to gate them.
- A new test file: it stays a `**Create:**` path.
- Webster's fork stencils and kick-start pack code (D6).
- Committing the 2026-10-15 pack audit under `docs/research/`: the board entry names it as context, not as this task's work.
- Removing or loosening stencil wording tests: `lighter-test-rules`, which depends on this task, does that.

## Decisions

### D1 — The plan does not pin the file a new test goes in

A `plan:` handle's unit is the package directory, or `<dir>_test` for an external test package, never a `.go` file: `handle-malformed` refuses a file unit under `language: go`.
The fork picks the `_test.go` file inside that package, as it does for any production member a `**Create:**` handle declares.
Rationale: test members follow the same glyph rules as production members, which is the operator's stated view, and Go does not care which `_test.go` file of a package holds a test.

### D2 — A card that adds tests beside existing ones names them under `**Uses:**`

The plan stencil gains one rule: a card whose new tests reuse an existing test's setup, or call existing test helpers or fakes, names those members as glyphs under `**Uses:**`.
The kick-start pack resolves them, so the fork sees their spans and their file, and places the new test beside them.
Rationale: the board entry asks that the pack and the fork prompt still tell the fork where new tests go; once the plain path is gone, a resolved neighbour is how they do, through a glyph rather than a whole-file path.

### D3 — New plan check `edit-test-file`, blocking, at the plan gates only

A file self glyph in a card's `**Edit:**` group whose path ends in `_test.go` is one blocking finding, attributed to the card, with `Ref` the entry.
It reads plan text alone, so it fires whether or not the file exists.
It is skipped under `language: none`, and a path with a `testdata` segment is exempt, since such a file is fixture data, not a test.
It is appended by the plan index to its `ValidateFormat` and `ValidateRework`, never to `ValidateDispatch`.
At `ValidateFormat`, cards in the told done set are skipped, since their work is history and `done-card-edited` bars editing them; this differs from `card-fabric-reference`, which filters nothing, so the check takes the done set.
At `ValidateRework` no skip applies: a rework generation is a fresh plan whose prior cards are archived (`PATTERN-plan-generation`).
Bound: a plan an earlier binary admitted can now be refused, but only at a plan gate whose writer can fix it; a run past its plan gate never halts on the check at dispatch.

### D4 — The check lives in `internal/planparser`, outside its two entry points

It is a `planparser` function the plan index appends, as `card-fabric-reference` and `fabric-vocabulary-name` are, because `planglyph`'s dispatch validation calls `planparser.ValidateFormat`.
Its ref-kind decision gets its own `refGate` and kind-policy ledger entry in `internal/planparser/shape.go` (`PATTERN-ref-shape-registry`).
It reads the self glyph's path through `Glyph.UnitPath`, the sanctioned converter, never by reading `Glyph.Unit` as a path or matching the glyph string (`PATTERN-glyph-conversion-chokepoint`); the spec sentence that limits `UnitPath` to the two disk-existence checks gains this third reader.
Rationale: deciding a ref's shape is `planparser`'s alone.

### D5 — The way forward the finding names

Name each existing test function, fake or helper the card changes as a member glyph under `**Edit:**`;
declare each new one as a `**Create:**` `plan:` handle on the package directory, or on `<dir>_test` for an external test package;
a new test file stays a `**Create:**` path.
A change confined to a test file's header, such as its build constraint, changes every member of that file, so the card names those members.
Rationale: `PATTERN-refusal-way-forward` asks every refusal to name its way forward; the row lands in `loom-plan-spec.md`'s validation table, where every plan check's row and way forward live, as `redundant-file-target`'s does.

### D6 — Webster's stencils and pack code stay as they are

The pack already lists a card's `plan:` handles beside its resolved entries, and the fork fragments say a handle names a member the card creates and is never located.
`ScopeGuard` counts a symbol-less change in a handle's package directory as covered, so a test file edited beside an internal-test handle is no deviation; for an external-test handle see Open risks.
Rationale: nothing in the fork's path depended on the plain test-file path beyond a whole-file read, which D2 replaces.
`webster-deviations-list` will later reword the fork's deviation rule against `ScopeGuard`; nothing here constrains it.

## Constraints

- `PATTERN-sole-parsers`, `PATTERN-ref-shape-registry` and `PATTERN-glyph-conversion-chokepoint`, as D4 applies them.
- `PATTERN-refusal-way-forward`: the finding names its way forward, its row lands in `loom-plan-spec.md` in the same commit, and a test reaches it.
- `PATTERN-test-economy`: extend an existing table where one covers the plan index's appended checks; a new top-level test that pushes a package past its row in `cmd/lyx/testdata/test-budget.yaml` puts that file under the card's `**Edit:**`.

## Auto-mode assumptions

- No operator answered; every pick is the writer's, logged in the support log's question ledger.
- The fork's free choice of file inside the handle's package (D1) is acceptable; nobody asked for the plan to pin test file placement.

## Open risks

- A fork may put a new test in a new file of the package instead of beside its neighbours; D2's `**Uses:**` rule mitigates this only where the plan writer follows it, and review sees the result.
- `ScopeGuard`'s package-directory coverage maps a glyph's `<dir>_test` unit to its directory, but whether a `plan:` handle on that unit gets the same mapping is unverified; a miss is an informational deviation only, never a failure.

## Acceptance criteria

- `lyx loom validate-plan` reports `edit-test-file` for a card whose `**Edit:**` lists a `_test.go` file, with a detail naming the way forward of D5.
- The rework plan gate reports it too, for a card of the new generation.
- It reports nothing for a `_test.go` path under `**Create:**`, `**Delete:**` or `**Uses:**`, for a test member glyph under `**Edit:**`, for a `_test.go` path with a `testdata` segment, for a path that only contains `_test.go` without ending in it (such as `foo_test.go.orig`), for a production `.go` or `.md` path under `**Edit:**`, and for any plan under `language: none`.
- At the format gate, a done card carrying a `_test.go` path under `**Edit:**` gets no finding, and dispatch validation never reports the check.
- The plan stencil no longer admits a plain test-file path under `**Edit:**`, states D2's `**Uses:**` rule, and its skeleton shows no `_test.go` under `**Edit:**`.
- `loom-plan-spec.md` carries the `edit-test-file` row with its way forward, and none of its examples lists a `_test.go` file under `**Edit:**`.

## Notes for the plan writer

- Plan fixtures elsewhere (planglyph, loomshed, sandbox) that list a `_test.go` file under `**Edit:**` and run through the index's format validation will start to report the finding; find and update them.
- `internal/planglyph/glyphchain_test.go` already binds a `Create` handle on an external test unit, so D5's `<dir>_test` route needs no new binding work.
