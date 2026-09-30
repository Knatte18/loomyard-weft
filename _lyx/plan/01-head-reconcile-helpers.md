# Card 1 — Merge-only head reconciliation and mid-merge probe

**Create:**
- `plan:internal/websterengine#refuseMidMerge` -> `func refuseMidMerge(worktree string) error`
- `plan:internal/websterengine#reconcileReportHead` -> `func reconcileReportHead(worktree, reportHead, subject string) (warning string, err error)`

**Edit:**
- `internal/websterengine/gitwrap.go`
- `internal/websterengine/gitwrap_test.go`

**Uses:**
- `internal/websterengine#headSHA`
- `internal/gitrepo#Repo.MergeHeadPresent`
- `internal/gitrepo#Repo.CommitParents`

**Intent:**
Add the two read-only git helpers both record-batch and recover-batch will call (decisions D1, D3 and D4), beside `headSHA` in `gitwrap.go`, and update the file's header comment to name them.

`refuseMidMerge` asks git whether `MERGE_HEAD` resolves in the repo at `worktree`, through `gitrepo.New(worktree).MergeHeadPresent()` — never by statting a `.git/MERGE_HEAD` file, which a linked worktree keeps in its per-worktree git dir.
When a merge is in progress it returns an error that says the worktree has a git merge in progress and names both routes to conclude it: `lyx fabric merge --continue` / `lyx fabric merge --abort` in a hub, and `git merge --continue` / `git merge --abort` for a standalone run.
A probe failure is returned wrapped; no merge in progress returns nil.

`reconcileReportHead` reads HEAD via `headSHA`.
HEAD equal to `reportHead` returns `("", nil)` — the unchanged fast path.
Otherwise it walks HEAD's first-parent chain with `CommitParents`, starting at HEAD.
At each commit it compares the SHA with `reportHead` first, before looking at the parent count: equal ends the walk and accepts, whatever that commit's parent count.
A commit not equal to `reportHead` must have two or more parents; its SHA is collected and the walk steps to its first parent.
A commit not equal to `reportHead` with fewer than two parents (a non-merge commit, or the root) ends the walk and is refused.
Comparing first matters when `reportHead` is itself a merge commit, for example a fork that committed nothing and reported a `StartSHA` an earlier parent merge left behind: a parent-count-first walk would collect it as a walked merge, step past it and refuse.
On acceptance it returns one warning naming `subject`, the report's `head_sha`, the current HEAD, and every collected merge SHA in walk order, and saying the batch is recorded at the report's `head_sha`.
On refusal the error starts with `webster: <subject>: head_sha %q does not match the worktree's actual HEAD %q` (the phrase existing tests pin) and goes on to state the merge-only rule: only merge commits, such as a parent merge-in, may sit between a fork's reported head and HEAD.
`subject` lets each caller keep its own report naming (`batch report <path>` for record-batch, `recovery report for batch NN-slug` for recover-batch).

Tests go in the existing integration-tagged `gitwrap_test.go`, built with its own scratch-repo helpers (the package `TestMain` is already hermetic):
- `reconcileReportHead`: HEAD equals the report head → empty warning, nil error;
  one merge commit on top (branch off, commit on a side branch, `git merge --no-ff`) → accepted, the warning names the report head, HEAD and the merge SHA;
  two successive merges → accepted, one warning naming both merge SHAs;
  a report head that is itself a merge commit, with a second `--no-ff` merge on top → accepted, the warning naming only the second merge's SHA;
  a fast-forward onto non-merge side-branch commits → refused;
  a non-merge commit after the report head, alone and with merges on either side of it → refused, error names both SHAs and states the rule;
  a report head not on HEAD's first-parent chain (for example a commit only on a merged-in side branch, or an all-zero hex SHA) → refused.
- `refuseMidMerge`: nil on a clean repo;
  an error naming both `--continue`/`--abort` routes after a conflicting `git merge` leaves `MERGE_HEAD`;
  nil again after `git merge --abort`;
  the same error in a linked worktree (`git worktree add`) with a conflicting merge in progress there.

**ImpactSummary:** new unexported helpers only; no caller changes until cards 2 and 3.

**Commit:** 1: websterengine: merge-only head reconciliation and mid-merge probe
