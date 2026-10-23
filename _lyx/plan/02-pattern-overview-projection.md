# Card 2 — pattern overview projection

**Create:**
- `internal/pattern#ProjectOverview`
- `internal/pattern#EntryBullet`
- `internal/pattern#TestProjectOverview`
- `internal/pattern#TestEntryBullet`

**Edit:**
- `internal/pattern/doc.go`

**Intent:**
D1's projection, as pure byte work beside `Headroom`, in a new file of `internal/pattern` that imports only the standard library, so the package keeps its import set (PATTERN-leaf-packages, `leaf_enforcement_test.go`).

An entry's bullet is its entry line, the line opening ``- `<id>` `` as `entryHeadRe` reads it, plus every following line `continues` reports, so a malformed multi-line bullet is replaced whole.

`ProjectOverview(overview, prescribed, removes)` returns a new byte slice and never changes `overview`:
it applies each line of `prescribed`, in order, by its entry ID: the line replaces the whole bullet of that ID, continuation lines included, or is appended at the end of the overview, after a newline when the overview does not end in one, when the ID is absent.
Applying in order makes the last line of an ID win.
It then deletes the bullet of every ID in `removes`, continuation lines included; an absent ID is no change.
A `prescribed` line whose head names no entry ID is skipped.
CRLF in `overview` is read as LF, as `patternlint.readOverview` does.
A nil or empty overview projects to the prescribed lines alone.

`EntryBullet(overview, id)` returns the bullet of `id` as it stands, entry line and continuation lines joined by newlines, and false when the overview holds no such entry.

`doc.go` gains a paragraph on the projection: what it replaces, appends and deletes, that it is idempotent over an overview that already holds the prescribed lines, and that the plan gate measures it with `Headroom`.

Tests, table-driven in package `pattern`:
`TestProjectOverview` rows: a prescribed line replacing a one-line bullet, a bullet with a continuation line replaced whole, an absent ID appended, an overview without a trailing newline, two lines for one ID projecting the second, a removes ID deleted with its continuation line, an absent removes ID, a line with no entry head skipped, a nil overview, and a projection of an overview already holding the line returning equal bytes; each asserts the exact output bytes.
`TestEntryBullet` rows: a one-line bullet, a bullet with a continuation line, and an absent ID.

**ImpactSummary:** additive only: two new exported functions in a leaf package with no existing caller changed; `doc.go` gains one paragraph.
