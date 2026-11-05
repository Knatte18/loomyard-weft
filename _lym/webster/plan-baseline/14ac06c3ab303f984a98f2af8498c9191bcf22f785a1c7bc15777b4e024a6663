# Card 3 — drift-capture-diff

**Create:**
- `internal/driftengine#CompileKeyPatterns`
- `internal/driftengine#Identity`
- `internal/driftengine#CellKind`
- `internal/driftengine#CellChanged`
- `internal/driftengine#CellAddedLine`
- `internal/driftengine#CellRemovedLine`
- `internal/driftengine#CellAddedCase`
- `internal/driftengine#CellRemovedCase`
- `internal/driftengine#Cell`
- `internal/driftengine#ListedLine`
- `internal/driftengine#CaseListing`
- `internal/driftengine#ListCaptures`
- `internal/driftengine#DiffCaptures`
- `internal/driftengine#TestDiffCaptures`

**Extends:**
- `internal/driftengine/doc.go`

**Intent:** Build D1's diff of two capture directories, pure Go over files, with no config read: the caller passes the compiled `drift.key_patterns`.

- `CompileKeyPatterns` compiles each pattern and refuses one that fails to compile or lacks a named group `key` or `value`, naming the pattern's index and text; an empty list compiles to none, which makes every line unkeyed.
- `ListCaptures` reads every regular file under `dir`, at any depth, as a case named by its slash-separated path relative to `dir`, sorted.
  It normalises CRLF to LF, drops the empty line a trailing newline leaves, and classifies each line on its own: the first pattern that matches keys it by its `key` group with the `value` group as its value; an unmatched line is unkeyed, its value the whole line.
- `Identity` holds `Key`, `N` and `Line`.
  A keyed line carries its key and its occurrence `N`, counted from 1 among the file's lines holding that key; `Line` is 0.
  An unkeyed line carries `Line`, its position among the file's unkeyed lines counted from 1, and an empty `Key`.
  Key and occurrence stay separate fields, so a key such as `Test #3` reads unambiguously; a listing writes `n` only for a key that occurs more than once in that file.
- `ListedLine` holds an `Identity` and the line's text; `CaseListing` holds `Case` and its `Lines`.
- `Cell` holds `Case`, `Identity` (zero for a case cell), `Kind`, `Old` and `New`.
  `DiffCaptures` lists both sides and returns, in case order and then file order: `added case` for a file only in `currentDir`, `removed case` for one only in `baselineDir`, `changed` for an identity on both sides with different values, and `added line` or `removed line` for an identity on one side only.
  Unkeyed identities pair by position among unkeyed lines, so inserting a keyed line moves no unkeyed identity.
  A case named in `leaveOut` is skipped on both sides; it serves a run whose old scope record skipped tests (D3).

`TestDiffCaptures` (tier0, `t.TempDir` fixtures, `t.Parallel`) is table-driven over D1's acceptance criterion: added and removed cases; a changed keyed value; inserting a keyed line moves no other keyed or unkeyed identity; a repeated key is told apart by its occurrence; a line keyed on one side and unkeyed on the other yields a keyed `added line` or `removed line` and an unkeyed cell; an empty pattern list makes every line unkeyed; CRLF and LF copies of one text yield no cell; a non-case file under the directory is read as a case; a repeated `Test #3` key keeps `Key` `Test #3` and its `N` apart; a `leaveOut` case yields no cell; the `cs` line `Re = 127069, dP = 7925.267 Pa` keys as `Re` under the default patterns.
A sub-run covers `CompileKeyPatterns` refusing a pattern with no `value` group, one with no `key` group and an invalid expression, and `ListCaptures` writing the listing's identities.
The doc.go gains a `# Diff` section.

**ImpactSummary:** Adds a new pure diff surface to driftengine; nothing existing calls or changes.
