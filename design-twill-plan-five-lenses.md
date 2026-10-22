# The Plan writer's twill-plan fan runs 5 lenses instead of 8

`twill-plan-five-lenses` · Task · enhancement, burler, loom · done

## Seen

The Plan writer's `twill-plan` fan runs 8 lens forks, each inheriting the Plan lead's large context: `affected-set`, `guard-reach`, `way-forward`, `criteria-coverage`, `branch-coverage`, `row-reachability`, `forced-edits`, `test-economy`.
Audit `lens-overlap-audit` (2026-10-22) found, over the runs with all 8 lenses:

- `affected-set` and `forced-edits` are near-duplicates (12 of `affected-set`'s 20 findings shared, 5 its alone; no BLOCKING finding of `affected-set` was its alone).
- `criteria-coverage`, `branch-coverage` and `row-reachability` overlap heavily.
- `guard-reach` and `way-forward` overlap little and each raised BLOCKING findings no other lens did.
- `test-economy` is mostly LOW but unique.

## Decision (operator, 2026-10-22)

`twill-plan` goes from 8 lenses to 5, as the audit recommends:

| Lens | Replaces |
| --- | --- |
| `coverage` | `criteria-coverage`, `branch-coverage` |
| `row-validity` | `row-reachability`, `test-economy`, the nil-seam part of `affected-set` |
| `forced-edits` | `affected-set`, `forced-edits` |
| `guard-reach` | unchanged |
| `way-forward` | unchanged |

- The change is in lyx's lenses template (`internal/lenses/template.yaml`, and any language layer under `internal/lenses/sets/` that overrides these lenses), so every hub gets it.
- A lens definition another fan still uses stays defined: `review-plan` runs `affected-set`, `guard-reach` and `way-forward`, and is out of scope here.
- Only `twill-plan` changes; `twill-discussion`, the review fans and the fixer fans stay as they are.

## Draft focus texts from the audit

A starting point, to be fitted to the current lens texts' style and rubric references:

```yaml
coverage: |
    Own whether every requirement has a row that would fail if it broke. List every
    decision and every acceptance criterion of the record, and every branch and exit
    the cards' Intents introduce or change whether or not the record names it (error
    returns, early returns, refusals, a loop's or scan's end without a match, a
    default case). For each, find the card whose Intent carries it and the planned
    test row that reaches it. A criterion the plan argues holds without a test,
    waives or defers, a decision no card carries, and a branch no row reaches are
    findings. Apply the Plan-Review rubric's "Fidelity to the answer key". Check that
    each path, comparison or default a card states uses the operand the record
    names, not a near one. Your list: per decision, criterion and branch, the card
    and the row, or none.
row-validity: |
    Own whether each planned row can do what its card says, and whether it is needed.
    Open every test function a card extends or names, and the fakes, fixtures and
    helpers it runs on. Check that the row's input reaches the branch it claims (the
    fake can produce that state at that point of the sequence), that its assertion
    would fail if the code were wrong, that each new fake field or helper is listed
    on the card, and that a zero-value or nil seam of a new field has a stated
    meaning where a hand-built receiver drives the code. Then, for every new
    top-level test and every new row, open the existing tests of the same function
    and report a test that should be a row of an existing test and a row that
    repeats a behavior an existing test already reaches. Apply the Plan-Review
    rubric's "Reachability of a prescribed refusal", "Verify coverage" and "Test
    economy". Your list: per row, reaches and fails-if-wrong, or not; per new
    test, new or repeat.
forced-edits: |
    Own what each change forces outside the members a card names. For every output,
    key, string, file shape, signature or behaviour a card changes, find its callers,
    the tests that assert it, and the comments, doc files, skills, fixtures and
    stated reasons (such as a `//testtiming:keep` reason) that describe it, from the
    pre-read and a search for what the pre-read lacks, and report each one the change
    makes wrong that no card lists; a card that covers part of a set is one finding
    naming the members it missed. Check that each `Create` handle's package is the
    package clause of the file the Intent puts it in, and that each listed glyph is
    the one `lyx quarry` gives. A new type, function or executor that must be
    registered in a table or list, such as a guard table or a seam list, is a forced
    edit, and a registration no card lists is a finding. Apply the Plan-Review
    rubric's "Member glyphs on an `Edit` card" and "The re-sign arrow". Your list:
    per change, the set you found and the cards that cover it.
```
