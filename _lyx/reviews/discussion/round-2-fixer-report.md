# Fixer report — discussion, round 2

## Changes

- F1 (MEDIUM, design), fixed as proposed.
  The premise holds: `webster-body-implementer.md` step 2's "A `plan:` handle names a member the card creates and does not resolve yet" goes only into the quarry fragment, and D7's pack fragment told the fork to locate every unresolved entry.
  D4 now lists unresolved member glyphs and `plan:` refs in separate sentences, and gives a `plan:` entry a fixed reason: it names a member a card creates and does not resolve yet.
  D7's pack fragment now says a `plan:` entry has nothing to locate, and limits the quarry-or-grep instruction to unresolved member glyphs, as does the sentence that follows it.
  The D7 acceptance criterion now pins the `plan:` sentence in the `claims_test.go` row.
- F2 (MEDIUM, design), fixed as proposed, using the review's first option.
  The premise holds: `cardOwnHandles` (`internal/planglyph/handle.go:375`) is unexported, `postBatchChecks` returns the not-done refusal before `BindHandles` (`internal/websterengine/recordbatch.go:779-781`), and `BindHandles` returns `nil, nil` under `language: none` (`handle.go:412-414`).
  D2 now defines the owned handles directly: Create declarations' handles and handle-shaped Rename new sides.
  It also moves that ownership rule into one exported `internal/planparser` function, which `BindHandles` and `internal/websterengine` both call, in line with `PATTERN-ref-shape-registry`.
- F3 (LOW, consistency), fixed as proposed.
  The premise holds: `postBatchChecks` returns `in.Geom.index()`'s error (`recordbatch.go:764-767`, `geometry.go:88-93`) before `DoneChecks` runs.
  D2's list of passes that log no bind lines now includes any error before or in `DoneChecks`, the `Index` lookup among them.
  The acceptance criterion now also names the `Index` lookup error.
- F4 (LOW, consistency), fixed as proposed.
  The premise holds: the Notes for the plan writer carry no banner note, and `contracts/stencils/webster/webster-prefix-fork.md:2` still names `_lyx/webster/prompts/`.
  The support log's Interview item 6 now says the note was dropped in round 1 as outside this task's scope.

## Disputed

None.

## Deferred

None.
