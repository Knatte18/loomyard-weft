No findings that need a change; the one item below is LOW.

**LOW — `internal/testcover/doc.go:47-48`.** The package doc says an integer or rune constant is changed, and it gives no exception. The fix now leaves alone any literal or constant that sits only in an expression of integer and rune constants, such as `(1 << 63) / 2`. The `PlanMutants` doc in `mutant.go` was updated for this, but the package-level description of the change-constant operator was not.

Checked and consistent:
- The `mutantBuildRounds` doc matches the `round >= mutantBuildRounds` wiring in `buildMutants`.
- The `gatecli/doc.go:1190` wording ("From build `mutantBuildRounds` on") matches too.
- The `mutantBuildRounds` count of builds matches the new `TestGateRedundant` subtest, which asserts `mutantBuildRounds+1` builds and checks `len(redundantRefusedLines) == mutantBuildRounds`.
- The F1 test fixture addition (`Half`, `blockHalf`) is appended after all existing blocks, so earlier block indexes and line numbers in `TestPlanMutants` are unchanged.
- The embedded-line fixture now has six non-blank lines, and the `len(tests)-1` check follows from the table.

Outside the lens: the new test asserts `Unbuildable != 21` inline. That is the sum of the 20 and 1 the sibling assertion in the same test hard-codes, so one will go stale without the other.

Further tool calls:
1. `grep` of `internal/testcover/doc.go` and `internal/gatecli/doc.go` for constant, `mutantBuildRounds` and round-count wording, because my context did not hold either doc file.
