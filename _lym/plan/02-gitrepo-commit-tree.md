# Card 2 — gitrepo-commit-tree

**Create:**
- `plan:internal/gitrepo#Repo.WriteCommitTree` -> `func (r *Repo) WriteCommitTree(tree string, parents []string, message string) (string, error)`

**Edit:**
- `internal/gitrepo_test#TestMergePrimitives`

**Uses:**
- `internal/gitrepo/doc.go`
- `internal/gitrepo#Repo.MergeTree`
- `internal/gitrepo#Repo.runChecked`

**Intent:** Add the `git commit-tree` call D4 needs, beside `Repo.MergeTree` in `internal/gitrepo/merge.go`.
`Repo.CommitTree` already exists and reads a commit's tree, so the new call is named `WriteCommitTree`.
It writes a commit object holding `tree`, with `parents` in order as its parents and `message` as its message read from standard input, and returns the new commit's SHA.
It moves no ref and touches no worktree or index.
A `tree` or parent that is not a full SHA returns `ErrInvalidSHA` before git runs, as `MergeTree` does.
It runs through `Repo.runChecked`, so a non-zero exit is a `*gitexec.GitError` (`PATTERN-gitrepo-client-boundary`).

`TestMergePrimitives`, tier1, gains a step: it merges two commits with `MergeTree`, writes the tree with `WriteCommitTree` once with one parent and once with two.
It asserts each commit's parents with `git log -1 --format=%P`, its tree with `git log -1 --format=%T`, and its message byte for byte with `--format=%B`, against literals.
A further row asserts a non-SHA tree returns `ErrInvalidSHA`.

**ImpactSummary:** Adds one method with no existing caller and extends one tier1 test; no production behaviour changes.
