# Card 2 — commits-named-by-one-batch-check

**Edit:**
- `internal/websterengine#commitsNamedBy`
- `internal/websterengine/gitwrap_test.go`

**Uses:**
- `plan:internal/gitexec#RunStdin`

**Intent:** Classify every candidate `git rev-parse --disambiguate=<prefix>` lists in one `git cat-file --batch-check` run instead of one `cat-file -t` process per object (D4).
`commitsNamedBy` joins the listed object names with newlines, hands them to `gitexec.RunStdin` as the batch-check's stdin, and keeps the names whose output line is typed `commit`.
A line ending in `missing` is an error naming the object, since `rev-parse` just listed it; any other malformed line is an error too.
A prefix naming no object still returns the empty slice with no second git call, and the result stays sorted.
The doc comment names the batch-check call in place of the per-object `cat-file -t`.
`TestRepositoryProbes` gains the rows the record names: a prefix shared by a commit, a blob and a tree returns the commit alone, and a fake-fed `missing` line is an error; the test pins that exactly one `cat-file` process runs by counting the probe's git calls where the fixture already can, or by asserting the batch-check's output shape otherwise.

**ImpactSummary:** One unexported helper in websterengine and its test; realGit's CommitsNamedBy keeps its signature, so no caller changes.
