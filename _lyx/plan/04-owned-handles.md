# Card 4 — Handle ownership moves into planparser

**Create:**
- `internal/planparser#OwnedHandles`

**Delete:**
- `internal/planglyph#cardOwnHandles`

**Edit:**
- `internal/planglyph#BindHandles`
- `internal/planglyph/doc.go`
- `internal/planglyph#TestBindHandles_RenameFileSidePairIsNotAHandle`

**Intent:**
Decision D2: the rule for which `plan:` handles a card owns moves into one exported `internal/planparser` function.
`BindHandles` and, from card 5 on, `internal/websterengine` both call it, so the passes that never reach `BindHandles` list the same handles it binds.
`PATTERN-ref-shape-registry` puts this decision in `planparser`.

- Add `OwnedHandles` to `internal/planparser/handle.go`.
  It returns a card's Create declarations' handles followed by every Rename pair's `New` side that `IsHandleRef` reports as a handle, in body order.
  That is exactly what `cardOwnHandles` returns today.
  Carry over its doc comment, including why a Rename-only card's handle-shaped `New` side counts.
- Delete `cardOwnHandles` from `internal/planglyph/handle.go`, and make `BindHandles` call `planparser.OwnedHandles` in its place, for both its findings and the outcomes card 3 added.
  `BindHandles`' signature does not change on this card.
- Rewrite every mention of `cardOwnHandles` to name `planparser.OwnedHandles`: in `BindHandles`' doc comment and body comments, in `TestBindHandles_RenameFileSidePairIsNotAHandle`'s doc comment, and in `internal/planglyph/doc.go`'s `bind-count-mismatch` entry.

The existing `BindHandles` tests already pin the ownership rule, so the move needs no test of its own.
They cover a Rename new-side handle binding, a file-rename pair that is not a handle, and a zero-handle card being skipped.

**ImpactSummary:** a pure move: `cardOwnHandles` has `BindHandles` as its only caller and becomes `planparser.OwnedHandles` with the same result, so no behavior changes and no caller outside `internal/planglyph` is touched.
