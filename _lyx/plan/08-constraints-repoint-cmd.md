# Card 8 — constraints-repoint-cmd

**Edit:**
- `cmd/lyx/boardguard_test.go`
- `cmd/lyx/checkedcall_test.go`
- `cmd/lyx/configstrictness_test.go`
- `cmd/lyx/constraintchokepoint_test.go`
- `cmd/lyx/crosscompile_test.go`
- `cmd/lyx/cwdmutation_test.go`
- `cmd/lyx/destructiveguard_test.go`
- `cmd/lyx/ghguard_test.go`
- `cmd/lyx/gitrepoboundary_test.go`
- `cmd/lyx/hermeticenv_test.go`
- `cmd/lyx/rawgitmutation_test.go`
- `cmd/lyx/sandbox_coverage_test.go`
- `cmd/lyx/seamsignature_test.go`
- `cmd/lyx/spawnobservability_test.go`
- `cmd/lyx/testmain_test.go`
- `cmd/lyx/tierpurity_test.go`
- `cmd/lyx/tmuxisolation_test.go`
- `cmd/lyx/verifiedtree_test.go`
- `tools/sandbox/pathresolve_guard_test.go`

**Intent:** Repoint every `CONSTRAINTS.md` citation in `cmd/lyx` and `tools/` to its PATTERN entry name (D7).

A citation of the form "`CONSTRAINTS.md`'s X Invariant", "X Invariant violated (see CONSTRAINTS.md)" or "see CONSTRAINTS.md" becomes the entry name from the Shared Decisions `entry-map` table.
For example: "Verified-Tree Invariant violated (see CONSTRAINTS.md)" becomes "`PATTERN-verified-tree` violated".
This applies alike to comments and to failure-message strings.
Test function names stay unchanged.
Use the `entry-map` table only; never invent a name.

Every changed failure message is test output, not asserted elsewhere.
Grep for each old message substring anyway, and any test asserting one joins this card.

**ImpactSummary:** Comment and failure-message rewording in the listed test files; no behavior change.
