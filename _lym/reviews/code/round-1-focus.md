---
round: 1
focus:
  - "internal/yamlengine/set.go `SetEntries`/`setEntryValue`: does a dotted key such as `docs.darn.loom.code_max_bounces=0` set exactly that one leaf, per the board entry's Expected section, and keep sibling keys, order and comments?"
  - "internal/yamlengine/set.go `setEntryValue`: check the scalar tag resolution (`scalarSetTag` with a `!!null` leaf tag), the empty-value case, and flow `{`/`[` values for correct YAML output, including values such as `0`, `true`, `null`, quoted text and strings that resemble numbers."
  - "internal/yamlengine/set.go `parentMappingOf` use: does a dotted key through a non-mapping, non-null value fail with a message naming the path, and do a key with an empty segment or a leading or trailing dot fail before anything is written?"
  - "internal/yamlengine/set.go: is the undotted-key path unchanged apart from the new refusal text, and does that text name the dotted form as the board entry's fallback expectation asks?"
  - "Callers of `SetEntries` and the JSONEntries spec path in internal/configengine/set.go: can a dotted key reach a layer or entry where a dot is part of a legitimate top-level name, so the new behavior changes an existing write?"
  - "internal/yamlengine/set_test.go: are new tests duplicating existing coverage, and is each added behavior (creating missing mappings, scalar tags, flow values, error cases) covered?"
  - "Docs and doc comments (docs/overview.md, internal/configcli/doc.go, internal/configengine/doc.go, internal/runprofile/doc.go, internal/yamlengine/doc.go): do they match the code as the diff leaves it, and do they follow the repository's comment conventions?"
---

Seed pass: the answer key is the board entry alone (issue #94); the record's other sections are empty.
The diff is one feature commit plus a comment-wrapping fix, concentrated in `internal/yamlengine/set.go`.
