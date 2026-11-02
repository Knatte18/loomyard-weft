# Fixer-report, code review round 1

## Changes

- **F1** (fixed with own scope): `summaryRenderer.list` in `internal/output/summary.go` sends every plain list under a note key through `noteLines`, so each non-empty value prints on its own line.
  The proposal limited this to lists past the collapse limit.
  Applying it to every length is the one consistent rule: a short `warnings` list printed joined while a long one printed per line would differ for no reason, and the entry asks for one line each.
- **F2**: `TestUseReadableOutput` gains a `wantCode` column and two rows.
  `failing` writes an unterminated `{"ok":true}` and errors, and the row expects the summary before the cobra error envelope, exit 1.
  `localenvelope` declares its own `envelope` flag, and the row expects the envelope.
- **F3**: `TestSummary` gains rows for a five-item list (joined, no footer), a long note list, and a failed envelope whose nested object carries `way_forward` and `x_warning` beside a `fetch_error`.
- **F4**: `pattern/PATTERN-cli-cobra.md` gains the bullet on the global `--envelope` flag and its shadowing.
- **F5**: `internal/shedverbs/human.go` builds `writerIsTerminal` from `clihelp.WriterIsTerminal`, keeping the test seam.
  `lym shed status` so uses `term.IsTerminal`, like every other operator verb.
- Self-check follow-ups: the `Summary` doc comment and the bounds comment above `summaryListLimit` name the note-key exception, and a `ShortNoteList_OneLinePerWarning` row pins a short note list.

- Gate repair: the two doc comments the follow-up added to `internal/output/summary.go` broke a sentence across lines at a comma.
  Each now holds one sentence per line: the bounds comment states the note-key exception as its own sentence, and the `Summary` comment lists it as its own semicolon clause.

Gate: `lym gate test ./internal/output ./internal/clihelp ./internal/shedverbs` passed before the self-check follow-ups, which are comment text and one test row.
The gate repair changes comment text only.

## Self-check

- fix-code-correctness, LOW: stale `Summary` doc comment — fixed as proposed.
- fix-consistency, LOW: stale `Summary` and bounds comments — fixed as proposed (merged with the one above).
- fix-consistency, LOW: no row for a short note list — fixed as proposed.
- fix-proposal, LOW: stale `Summary` doc comment — merged with the one above.
- fix-affected-set, LOW: `internal/loomcli/overlay.go:30` terminal check — disputed, `premise false`: `inputIsTerminal` decides whether a reader is a terminal before any read, not where output goes.

## Disputed

- fix-affected-set LOW, `internal/loomcli/overlay.go:30`, ground `premise false`: the function is `inputIsTerminal(in io.Reader)`, "checked before any read, so a terminal never blocks the verb".
  It is an input check and not a second output-writer check.

## Deferred

Nothing deferred.
