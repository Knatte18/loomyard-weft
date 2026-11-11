---
round: 1
focus:
  - "internal/gatecli/testmappackage.go: do both dirty checks read fresh worktree status immediately before the build and after the last test, each over every linked package's files, and does a dirty second read write no map and leave the earlier one?"
  - "internal/gatecli/testmaptree.go and internal/fabricengine/codereads.go: does the per-linked-package hash set follow impactset's path-to-package rule (own Go files plus every other file outside nested package directories, go:embed targets in subdirectories included), and are nested package directories excluded?"
  - "internal/testcover/coveragemap.go and blockkey.go: the map shape carries the linked-package field, per-test spawn verdict and a format marker with Commit and BuiltAt also on a failed-build map; the record's later D13 entry says function sets replace import-path block keys, so check the diff against that amended text and that undeclared keys and older maps still read tolerantly."
  - "Do the redundancy report and mapDrift keep their verdicts, meaning the redundancy builds stay on an own-package coverpkg and mapDrift compares own-package files only, while tests that cover only another package's code now enter the map?"
  - "Does a change to a linked package's file alone leave a map fresh, with no staleness-rule change in the diff?"
  - "Test economy: the new integration tests in internal/gatecli/testmap_integration_test.go and internal/testcover/runner_test.go against the existing tests, and whether each acceptance criterion has coverage."
  - "Doc updates: docs/overview.md, internal/gatecli/doc.go, internal/testcover/doc.go and pattern/PATTERN-test-speed.md against the repository's constraints document, and the commit messages' claims against the diff."
---

Seeded from the decision record's acceptance criteria and the review range's three feature commits.
