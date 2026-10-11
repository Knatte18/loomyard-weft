# Fixer report — discussion round 1

Every premise was checked against the code and the fasit and holds: `BindHandles` returns `nil, nil` without a glyph language (`internal/planglyph/handle.go`), `loom-plan-spec.md` runs the handle checks under `language: none`, `postBatchChecks` returns `cardNotDoneError`/`ErrCardNotDone` on the paths D2 gives reasons for (`internal/websterengine/recordbatch.go`), the `webster-prefix-fork.md` banner says `_lyx/webster/prompts/`, and `pattern/PATTERN-leaf-packages.md` does not name `planindex`.

## Changes

- F1 (fixed as proposed): D7's pack fragment now says "each member glyph on the cards", scopes the quarry and grep ban to resolved entries, and gives an unresolved entry `lyx quarry resolve` or a grep as its locate path, stating that bound.
  The pack-fragment acceptance criterion now names the unresolved-entry instruction.
  D7's rationale is narrowed to match: the fork does not call quarry for what the pack already resolved.
- F2 (fixed as proposed): D2 drops the "gets past its done-check and delta calls without an error" entry condition, says the not-done refusal still logs its lines, and states the no-lines rule as the three named error cases only.
- F3 (fixed, sixth reason chosen): D2's closed set gains `language-none`, which replaces the `BindHandles`-side reasons and `delta-unavailable` but never the done-check reasons, so "exactly one line per handle" holds under `language: none` and matches D9's `language-none` state.
  The D2 criterion gains the `language: none` scenario.
- F4 (fixed as proposed): the D2 criterion gains the scenario that a pass ending in a `DoneChecks`, non-`ErrQuarryUnavailable` `Delta` or `BindHandles` error logs no bind lines.
- F5 (fixed, removal chosen): the banner note is removed from Notes for the plan writer.
  No Decision touches `webster-prefix-fork.md`, so the fix is outside this task's scope rather than a requirement.
- F6 (fixed as proposed): the Constraints bullet cites the `planindex` package doc alone.

`lyx loom validate-discussion` passes on the edited pair.

## Disputed

None.

## Deferred

None.
