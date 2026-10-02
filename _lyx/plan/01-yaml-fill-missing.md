# Card 1 — yaml-fill-missing

**Create:**
- `plan:internal/yamlengine#FillMissing` -> `func FillMissing(template, existing []byte) (filled []byte, keys []string, err error)`
- `internal/yamlengine/fill.go`
- `internal/yamlengine/fill_test.go`

**Prosa:**
- `docs/shared-libs/yamlengine.md`

**Intent:**
Add `FillMissing` in a new `internal/yamlengine/fill.go`: it merges every mapping key the template holds and `existing` lacks into `existing`'s own document, in memory, and returns the merged bytes plus the sorted dotted key-paths it inserted.
This is the merge D1 needs; `Reconcile` cannot serve, because it builds on the template (dropping file keys the template lacks and replacing a mismatched shape) and its `added` lists keys inside list elements it never fills.

The walk runs over the two node trees in parallel, starting at the document root:

- A template key the file's mapping lacks is appended to that mapping as a deep copy of the template's key and value nodes, after the file's own keys, in template order.
  The reported path is the inserted key's own dotted path (`selvage.height_rows`, or `status_line` for a whole missing mapping), never its descendants.
- A template mapping whose file counterpart is also a mapping is recursed into.
- A shape mismatch returns an error naming the key-path and both shapes: a scalar or null (`status_line:` with no value parses as a null scalar) where the template holds a mapping, or a mapping where the template holds a scalar.
  A non-mapping document root is the same mismatch at the root.
- A key present with any other value, empty string and null included, keeps its value; a key the template lacks stays as the file has it.
- A present sequence is carried whole, keeping its length; nothing is filled inside its elements, and `FillMissing` does not descend into sequences at all.
  A template key missing inside an element of a present list therefore stays missing, for `MissingKeys` to report as it does today.
- An empty or comments-only `existing` returns the template bytes verbatim, reporting every top-level template key.
- When nothing is missing, it returns `existing` byte-for-byte, with an empty key list, so a complete file resolves exactly as before.
- Unparseable YAML on either side returns a parse error, as `MissingKeys` does.

`fill_test.go` covers each rule above with inline fixture templates: a missing top-level key, a missing nested key, a missing whole mapping, a present empty string and a present null kept, an emptied list kept empty, an extra file key surviving beside a filled one, file key order preserved with the fill appended, the empty and comments-only file, the no-op byte identity, each shape-mismatch refusal naming its key-path, and a missing key inside a present list's element left for `MissingKeys`.

`docs/shared-libs/yamlengine.md` gains a `FillMissing` section beside `MissingKeys`, stating these rules and how they differ from `Reconcile`.
