---
round: 2
focus:
  - "internal/testcover/testselect.go `embedPatterns`, `ownFilesMiss`, `linkedChanges`: does the inert-file carve-out read the importing package's own _test.go go:embed directives, while a linked package reads non-test Go files only?"
  - "internal/testcover/testselect_test.go `TestSelectorSelect`: does a row edit a method body so the Type.Method key the selector matches against the map's block keys is pinned?"
  - "internal/testcover/testselect_test.go `TestSelectorSelect`: does the table cover each miss and no-miss case the decision record's D5 lists, including marker BuiltAt and Version mismatch, commit-side parse failure, absent probe file, removed declarations, removed init, method named max, and own-file add, remove and non-Go change?"
  - "internal/testcover/testselect.go `embedPatterns`: are quoted, backquoted, all:-prefixed, multi-pattern and glob go:embed shapes, and a malformed directive, each pinned by a row that would fail if the parsing broke?"
  - "internal/testcover/declcompare.go `compareGoSources` and internal/testcover/testselect.go `importAllowed`: do a deleted file's imports and a dotless non-standard-library import path behave as D4 and D5 word them?"
---

Focus covers the changed selector code and its table test in `internal/testcover`.
