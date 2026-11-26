# Card 3 — yaml-move-keys

**Create:**
- `plan:internal/yamlengine#Move` -> `type Move struct`
- `plan:internal/yamlengine#MoveKeys` -> `func MoveKeys(existing []byte, prefix string, moves map[string]string) ([]byte, []Move, error)`
- `plan:internal/yamlengine#TestMoveKeys` -> `func TestMoveKeys(t *testing.T)`

**Extends:**
- `internal/yamlengine/doc.go`

**Intent:** Add the node-level key move D5 and D6 build on.
`MoveKeys(existing, prefix, moves)` rewrites the YAML document `existing`: under the mapping at the dotted path `prefix` (the root when `prefix` is empty), each old dotted key of `moves` is moved to its new dotted path, creating the intermediate mappings in block style.
`Move` records one change: `Old` and `New` paths (relative to `prefix`) and `Outcome`, one of `moved`, `dropped-null` and `dropped-new-set`.
The rules, from D5:
- the move keeps the value and its head, line and foot comments, and every other key with its comments, in place;
- an old key equal to its new path's first segment, such as `discussion` → `discussion.model`, is the old key only while its value is not a mapping; a mapping there is the new area and is left alone;
- an old key whose value is null is dropped and reported `dropped-null`, since the new path's absence reads the same default;
- a document holding both an old key and its new path keeps the new path's value, drops the old key and reports `dropped-new-set`;
- a document holding no old key, or an absent prefix, comes back byte-identical with no moves, so a second run changes nothing.
Moves run in sorted old-key order, so the result is deterministic.
An unparseable document is an error.

`TestMoveKeys` is a new tier0 table test with rows for: a flat key moved under a new area beside an existing key of that area; a scalar `discussion: sonnet[medium]` moved to `discussion.model`, and a `discussion:` mapping left alone; a null old key dropped; an old and a new key together, the new value kept; a commented key keeping its comment and its neighbour's; a non-empty prefix moving keys only below it; and a second pass over each result returning it unchanged with no moves.

`doc.go` gains one sentence naming `MoveKeys`.

**ImpactSummary:** One new exported function and type in yamlengine with no caller yet, plus a one-sentence doc addition.
