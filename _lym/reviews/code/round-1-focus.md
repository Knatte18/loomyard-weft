---
round: 1
focus:
  - "internal/testcover/testselect.go: does the selector take its target (working tree or commit via fabricengine.Fabric.CodeFileAt) and give identical verdicts for an equal tree and commit, with every file read only through the fabric (PATTERN-fabric-boundary)?"
  - "internal/testcover/testselect.go and declcompare.go: are own non-test Go files compared by function, and are test files compared with every top-level Test function's text removed, so a changed Test selects that test alone while a helper, TestMain, init or package-level declaration change is a miss?"
  - "internal/testcover/testselect.go: are own non-Go files, testdata and its Go files included, a miss unless inert_paths exempts them and no go:embed pattern names them, and are outcome and spawn entries treated as no recorded test while a build-failure record beside test entries is no miss?"
  - "internal/testcover/coveragemap.go and internal/gatecli/testmap*.go: is the environment stamp built with the toolchain, GOOS/GOARCH, go.mod and go.sum hashes, the mapped tiers' command and environment digest taken before {parallel}, and the env.yaml variables digest; does a differing stamp miss, and does a changed parallel value leave it unchanged?"
  - "internal/testcover/coveragemap.go: does the map record linked-package coverage as function sets by import path and file, with no block key of another package, plus hashes of the package's own non-Go files; are the added fields tolerant on read (PATTERN-tolerant-records)?"
  - "internal/gatecli/coveragescope.go and testmappackage.go: does the gate still run a touched package whole, and do the selector's new signature and verdict reach every caller correctly, including failure paths for unreadable commits or files?"
  - "internal/testcover/testselect_test.go and internal/gatecli/coveragescope_test.go: do the tests cover each new behavior above (a behavior with no coverage is a finding), and does any duplicate an existing test's coverage?"
  - "internal/testcover/doc.go and internal/gatecli/doc.go: do the doc changes match the code as the diff leaves it, and do the commit messages' Pattern-Departure trailers, if any, name entries that forbid the way taken?"
---

Seed focus drawn from the decision record's D2, D3 and D6 and its acceptance criteria.
