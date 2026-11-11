---
round: 1
focus:
  - "internal/testcover/testselect.go: does every D5 miss case (map unreadable/unmarked/marker Commit and BuiltAt mismatch, unreadable Commit, own files incl. tests added/removed/rehashed, non-Go file changes, go:embed carve-out from inert_paths with directory patterns) widen to a whole run, and does no error path narrow to a partial selection?"
  - "internal/testcover/declcompare.go: does the non-body change test match D5 exactly: comments, formatting, reordered or regrouped imports, and the added-declaration exemptions (const outside iota/implicit group, var without initializer, type without methods) are no miss, while changed or removed const/var/type, struct fields, iota-group specs, added var with initializer, and import adds or removes other than linked-set or standard-library are a miss?"
  - "internal/testcover/declcompare.go: do method set changes (receiver, signature), //go: directive lines anywhere in the file, added init count, predeclared-name additions and parse failure at either side each produce a miss; and does D4's key-name grouping (several init functions changing together) and whole-file deletion select the right tests?"
  - "internal/testcover/testselect.go: does selection match a test's covered blocks to changed functions using the same enclosing-function naming as the map's block keys, including methods and closures, and is file content at the map's Commit read only through fabricengine.Fabric.CodeFileAt (PATTERN-fabric-boundary)?"
  - "internal/testcover/testselect_test.go: do the tests cover each acceptance criterion in the decision record (D4 and D5 lists) without duplicating one another; is any listed miss case or no-miss case left without coverage?"
  - "internal/gatecli/testmaptree.go and internal/testcover/packagedirs.go: is the NestedPackageDirs move behavior-preserving, with no caller or reference left on the old location?"
  - "Scope: does the diff stay within the record's Exports (a selector in internal/testcover) and leave gate wiring to coverage-gate-scope; do docs/overview.md and doc.go match the behavior the diff lands, and do commit messages carry no unexplained departure?"
---

Seed focus for round 1 over the full range. Each entry names a place and a question; none states a verdict.
