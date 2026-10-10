# Card 2 — commits-named-by-one-batch-check

**Edit:**
- `internal/websterengine#commitsNamedBy`
- `internal/websterengine/gitwrap_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/websterengine#commitsFromBatchCheck` -> `func commitsFromBatchCheck(output string) ([]string, error)`

**Uses:**
- `plan:internal/gitexec#RunStdin`

**Intent:** Classify every candidate `git rev-parse --disambiguate=<prefix>` lists in one `git cat-file --batch-check` run instead of one `cat-file -t` process per object (D4).
`commitsNamedBy` joins the listed object names with newlines, hands them to `gitexec.RunStdin` as the batch-check's stdin, and passes the output to `commitsFromBatchCheck`.
`commitsFromBatchCheck` reads one `<name> <type> <size>` line per object and keeps the names typed `commit`; a `<name> missing` line is an error naming the object, since `rev-parse` just listed it, and any other malformed line is an error too.
A prefix naming no object still returns the empty slice with no second git call, and the result stays sorted.
The doc comment names the batch-check call in place of the per-object `cat-file -t`.
`TestRepositoryProbes`'s `commitsNamedBy` subtest keeps its real-git rows, which now run through the batch-check, and a new subtest beside it table-tests `commitsFromBatchCheck` over literal batch-check output: commit, blob and tree lines return the commit alone, a `missing` line is an error naming the object, a malformed line is an error, and empty output returns nothing.
Those rows assert returned values only, so a per-object `cat-file -t` loop would pass them; a new top-level `TestCommitsNamedBy_OneBatchCheck` in `gitwrap_test.go` pins the single call: it resolves the real `git` with `exec.LookPath`, puts a recording `git` script that logs its arguments and execs the real one first on `PATH` with `t.Setenv`, and asserts that `commitsNamedBy` over a prefix naming a commit, a blob and a tree returns the commit and logs exactly one `cat-file --batch-check` call and no `cat-file -t`.
It sets `PATH`, so it runs without `t.Parallel` under a comment naming that variable, and the websterengine row of `test-budget.yaml` rises by one.

**ImpactSummary:** One unexported helper in websterengine, its tests and a budget row; realGit's CommitsNamedBy keeps its signature, so no caller changes.
