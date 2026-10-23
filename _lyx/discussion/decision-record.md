# Decision record — prime-owner-visible

## Goal

Show which machine owns the hub's prime wherever task owners show: at the top of the rendered board README and in `lyx board list`, both its JSON and its `--text` output.

## Scope

- `internal/boardengine`: the README renderer takes the board's prime owner and states it under the render time; every render call site passes the store's prime owner.
- `internal/boardengine`: a `Board` read that returns the recorded prime owner.
- `internal/boardcli`: `list` carries the prime owner in its JSON envelope and opens its `--text` listing with a prime-owner line.
- The package docs of both packages and the board skill's list paragraph.

Out of scope: `find`, `get`, `list-full`, and any change to how the prime owner is recorded or replayed.

## Decisions

### The README states the prime owner on the line after the render time

Decision: the README's opening paragraph gains `Prime owner: ` followed by the machine in a code span, or `none recorded` when board.json records none.
Rationale: the top of the README is where the board entry asks for it, and stating the empty case keeps the line's place stable and tells the reader no machine owns the prime.

### Render takes the prime owner as an argument

Decision: `Render` and `RenderToDisk` take the prime owner beside the tasks, and every caller passes its store's record.
Rationale: the renderer stays a pure function of its inputs, and each caller already holds the loaded store.

### `list` JSON carries a top-level `prime_owner` key, always present

Decision: the `list` envelope is `{"ok":true,"prime_owner":"<machine>","tasks":[…]}`, the value empty when none is recorded.
Rationale: the prime is not an entry, so it belongs beside the tasks array rather than in it, and an always-present key lets a reader tell an empty record from an older binary.

### `list --text` opens with a `prime owner:` line

Decision: the text listing's first line reads `prime owner: <machine>`, or `prime owner: none` when none is recorded, followed by the entry lines unchanged.
Rationale: the operator scans the text listing, so the owner has to be on it; `find --text` stays a pure entry listing.

### The list reads the prime owner through its own `Board` read

Decision: a `Board.PrimeOwner` read loads the store and returns the recorded prime owner, and `list` calls it beside `ListTasksBrief`.
Rationale: it keeps `ListTasksBrief`'s signature and callers untouched; the second load is a read under the swap lock, and a write landing between the two reads only shows a fresher owner.

## Constraints

- `PATTERN-tolerant-records`: no change to the board.json shape.
- `PATTERN-comment-line-breaks`: one sentence per Go comment line.
- `PATTERN-test-economy`: extend the existing README render table, the owners CLI step and the `--text` step rather than adding tests.
- Docs land in the same commit as the code: `internal/boardengine/doc.go` and the render file header, `internal/boardcli/doc.go`, and the board skill.

## Auto-mode assumptions

- The prime owner shown is the recorded `prime_owner`, not a resolved one: the prime has no parent to inherit from.
- The `--text` header line is lower-case to match the listing's lower-case kind and status cells.

## Open risks

- A script parsing `list --text` line by line now sees one extra first line; no consumer in the repository parses it.

## Acceptance criteria

- A README rendered from a board recording a prime owner states that machine at its top; one rendered from a board recording none states that none is recorded.
- `lyx board list` prints the recorded prime owner as `prime_owner`, and an empty string when none is recorded.
- `lyx board list --text` opens with the prime-owner line, and `find --text` does not.
- Every existing board render and listing test passes with the new line in its expected output.
