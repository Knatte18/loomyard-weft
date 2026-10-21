# Card 1 — open-map-flow-values

**Edit:**
- `internal/yamlengine#setOpenMapEntry` -> `func setOpenMapEntry(root *yaml.Node, open, name string, pair KV) error`
- `internal/yamlengine#SetValues`
- `internal/yamlengine#SetSparse`
- `internal/yamlengine#TestSetValues`
- `internal/yamlengine#TestSetSparse`
- `internal/yamlengine/doc.go`

**Intent:**
Let `lyx config <module> --set` write a whole open-map entry as a YAML flow mapping or a flow list (D1), so the orch writes a gate tier with one `--set 'tiers.tier0={command: "go test -json {targets} {args}", timeout_min: 60, counter: go-json}'` and a gate list with `--set 'gates.push=[0, 1, 2]'`.

An open-map pair `P.<name>=<value>` keeps writing a string scalar, except:
a value whose first non-blank character is `{` parses as a YAML flow mapping and replaces the entry whole, and a value whose first non-blank character is `[` parses as a YAML flow list and replaces the entry whole.
A value of either shape that does not parse as one YAML node of that kind, or carries text after its closing bracket, is refused naming the key and the quoting rule: inside the mapping or list a value holding a brace, comma or bracket is a double-quoted string, since a flow-context plain scalar may hold none of them.
The refusal is returned before any write, so the file is unchanged.
`setOpenMapEntry` takes the pair so its refusal names the key; `SetValues` passes it, and `SetSparse` builds the same entry node through one shared helper instead of `stringNode(pair.Value)`, so the hub layer and a sparse machine layer write the same shape.
The parsed node keeps its own scalar tags, so `timeout_min: 60` reads back as an integer and `parallel: 4` as an integer.

Bound: the change widens only what an open-map entry may hold; a key outside an open map, a per-name map entry and a scalar-or-list open map at P itself keep today's shapes, and the module's own decode still validates the entry, as gate's loader does (card 2).

Tests: `TestSetValues` gains rows for a flow-mapping entry appended and replacing an existing scalar entry whole, a flow-list entry, a quoted value holding a comma, and the refusal of `{a: 1` and `{a: 1} x` with the file unchanged;
`TestSetSparse` gains the flow-mapping row on a sparse layer.
`doc.go`'s open-map list states the two new value shapes.

**ImpactSummary:** one unexported helper re-signed with its one caller in the package; every `--set` caller of an open map gains the two shapes, no existing value changes meaning except a value starting with `{` or `[`, which no open map stores today as a scalar.
