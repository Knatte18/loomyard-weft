# Round 1 fixer report — discussion

Every finding's premise held against the code: `refMember` in `internal/planparser/targetdirs.go` is the only member-against-self classifier and is unexported, `ScopeGuard` in `internal/planglyph/scope.go` picks one side of a Modified symbol and `To.File` for a rename, and the plan spec paragraph under `## Deferred / forward-compat` carries the "This union is defined" sentence.

## Changes

All in `_lyx/discussion/decision-record.md`.

- **F1** fixed as proposed: Scope In gains an `internal/planparser` bullet for an exported form of `refMember`; the `PATTERN-ref-shape-registry` constraint says why `RefScope` and `RefCovers` cannot serve.
- **F2** fixed as proposed: D1 states the deciding file per delta shape (every `Before` and `After` file for Modified, both `From.File` and `To.File` for a rename's file and package fallback, own `File` for Created and Deleted); Acceptance criteria gain the moved-symbol row.
- **F3** fixed as proposed: D2's `**Uses:**` bullet now says Uses entries add no coverage, with the batch-target exception.
- **F4** fixed as proposed, taking the drop option: the "names a glyph or changes-files union" clause is gone from the prompt criterion, the package criterion names both target shapes, and a criterion for the step-2 `**Uses:**` rule is added in F3's wording.
- **F5** fixed as proposed: D3 says the "This union is defined over each card's flat target set" sentence is dropped.
- **F6** fixed as proposed: D1's rationale sentence about the board entry's bug is deleted.

`support-log.md` is unchanged.

## Disputed

None.

## Deferred

None.
