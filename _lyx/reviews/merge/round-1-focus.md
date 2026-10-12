---
round: 1
focus:
  - "internal/shedrun/seed.go: the parent made `decodeSeed` tolerant (PATTERN-tolerant-records) while the task added `ParseSeedTolerant` beside `ParseSeed`. Settle whether the two now decode identically, whether `decodeSeed`'s doc comment still claims it rejects an unknown key, and whether the task's second parser is redundant or still needed (sole-parser rule in PATTERN-sole-parsers)."
  - "internal/shedengine/status.go: the task added `ParseStatusTolerant`, and the parent moved shed status reads to `state.ReadJSONNoMkdir` and tolerant decoding. Settle whether `ParseStatusTolerant` duplicates or contradicts the parent's read path, and whether the task's package doc still describes the status read gate correctly."
  - "PATTERN-tolerant-records against the task's new `internal/reportengine` readers: settle whether any reader of a Go-written JSON record refuses an unknown key or a State value, and whether `cmd/lyx/tolerantrecords_test.go` covers the new package."
  - "PATTERN-leaf-packages.md, PATTERN-sole-parsers.md and PATTERN-cli-cobra.md: the parent added `pauserecord` and `pausecli` entries and the task added its own entries to the same files. Settle whether the merged text is coherent, and whether `reportengine` and the report CLI module follow the import-direction rules and the cobra deviation list."
  - "cmd/lyx/main.go, cmd/lyx/seamsignature_test.go and cmd/lyx/testdata/test-budget.yaml: both sides registered modules and test budgets. Settle whether the report module is mounted next to the parent's `pause` and `resume` roots, whether the seam-signature expectations hold for both, and whether the budget rows fit the merged test counts."
  - "docs/overview.md and tools/sandbox/SANDBOX-CORE-SUITE.md: both sides edited the module table and the sandbox coverage. Settle whether the merged tables name the removed webster pause verbs nowhere and list `lyx report` once, and whether the sandbox suite still matches the verbs that exist."
  - "internal/shedadapters, internal/shedrun/paths.go and internal/battenshed/innerrun.go: the task added round and archive readers over run records the parent changed (pause fields, `pause_after`, run-level pause flow). Settle whether the report counts of stops, repairs and review rounds still read the fields the parent now writes, or silently count a retired field."
  - "Test economy across the merge: settle whether a task test now repeats a parent test over the same record read, and whether the interaction of the report readers with the parent's new pause record and tolerant decoding has any covering test."
---

The focus reads the merged tree against the parent's two new invariants: tolerant records and the pause record.
Both are checked against the task's readers, which parse records the parent changed in the same range.
