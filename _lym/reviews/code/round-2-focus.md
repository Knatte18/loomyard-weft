---
round: 2
focus:
  - "internal/testkit/gofmt_test.go, gofmtFinding and the table in TestGofmt_ModuleFilesAreFormatted: for every way a source can differ from its format.Source output (a missing final newline, extra trailing blank lines, line endings, a changed line), does the function return a finding that names the file, a correct line and the fix hint, and do the table rows assert that from independent expected values?"
  - "_lym/landing/summary.md and the doc comments in internal/testkit/gofmt_test.go: do their statements of which Go files the check reads and exempts match the walk in internal/testkit/scankit/scankit.go (skipDirNames, skipDir) and the filter the test passes?"
  - "internal/testkit/gofmt_test.go against the repository's rules: are its tier and marker comments (guard, watch, only) placed and scoped as PATTERN-test-speed and its background require, and is internal/testkit an allowed home for a whole-repository guard under PATTERN-test-isolation?"
---
