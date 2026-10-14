---
round: 1
focus:
  - "internal/yamlengine/set.go and layers.go: does the open-map-key list path reach both `SetValues` and `SetSparse`, and is a value never written as a string when it begins with `[` but fails to parse as a list of scalars?"
  - "internal/yamlengine/set.go: for a key that is a declared open-map path, are mapping-valued open maps (board `labels`, `P.<name>` entries) unaffected, and does the replaced node keep its comments?"
  - "Board entry's wanted behavior: a per-round model key accepts a list and a scalar both ways (scalar to list and back), and an unwritable value is refused with nothing written; check the JSON-quoted form from the entry as well as the unquoted one."
  - "internal/yamlengine/set_test.go and layers_test.go: do the new rows cover each added behavior on both write paths, and does any row duplicate an existing test's coverage?"
  - "Docs in the same diff: do `internal/configcli/doc.go`, `internal/yamlengine/doc.go`, `docs/overview.md` and the `configcli.go` help text match the code, and does the summary's claim about help-text caveats moving to the package doc hold?"
---

Seed focus from the board entry, the change description and the diff stat; the diff itself has not been read in depth.
