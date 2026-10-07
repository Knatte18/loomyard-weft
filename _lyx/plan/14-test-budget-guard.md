# Card 14 — test-budget-guard

**Create:**
- `cmd/lyx/testbudget_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Edit:**
- `pattern/PATTERN-test-speed.md`

**Intent:**
Decision D9: a per-package test-count budget, so the suite cannot regrow unseen.

`cmd/lyx/testdata/test-budget.yaml` maps each package directory (module-relative, slash-separated) to the number of top-level `func Test…` functions in its `_test.go` files under every build tag, pinned at this card's tree, which is the after-state: every earlier card's tests are in.
An untagged guard in `cmd/lyx/testbudget_test.go`, marked `//lyx:guard`, counts them by a static parse through `scankit` (it spawns nothing) and fails when a package exceeds its budget or has tests but no row, naming the package, its count and its budget.
A count below budget passes, and so does a row whose package has no tests left.
Wall time is not pinned, and the count is not compared with `cmd/testtiming`'s `TESTS` column.

Bound: a budget raise is a one-line edit in the same commit as the tests that need it, so the guard forces a visible, reviewed decision and blocks nothing else.

The counting and comparison are pure functions over parsed files and the budget map; the same file's table test covers them over in-memory sources: a package over budget fails with count and budget, a package with tests and no row fails, a package under budget passes, and a test in a `tmux` or `llm`-tagged file counts.

`pattern/PATTERN-test-speed.md`'s background states the budget, its file and the raise rule.

**ImpactSummary:** Adds one guard, its budget file and a background paragraph; no production code changes.
