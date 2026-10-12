# Card 8 — gitrepo-history-reads

**Create:**
- `plan:internal/gitrepo#TagRef` -> `type TagRef struct`
- `plan:internal/gitrepo#Repo.TagRefs` -> `func (r *Repo) TagRefs(prefix string) ([]TagRef, error)`
- `plan:internal/gitrepo#LogQuery` -> `type LogQuery struct`
- `plan:internal/gitrepo#LogCommit` -> `type LogCommit struct`
- `plan:internal/gitrepo#Repo.Log` -> `func (r *Repo) Log(tip string, q LogQuery) ([]LogCommit, error)`
- `plan:internal/gitrepo#Repo.SubdirsAtRevision` -> `func (r *Repo) SubdirsAtRevision(rev, dir string) ([]string, error)`
- `//internal/gitrepo/reportreads_integration_test.go`
- `plan:internal/gitrepo_test#TestReportReads` -> `func TestReportReads(t *testing.T)`

**Edit:**
- `//internal/gitrepo/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D3 reads every record from git objects through `internal/gitrepo`, which today has no tag listing, no walk from an arbitrary tip and no directory listing beyond regular files.
The three reads land in a new file, `internal/gitrepo/historyreads.go`, each a go-git local read through `readGoGit` (`PATTERN-gogit-read-helper`, `PATTERN-gitrepo-client-boundary`), spawning no git.

`TagRefs` lists every tag whose short name starts with `prefix`, an annotated tag peeled to its commit and a tag naming no commit skipped, as `TagRef{Name, Commit, Committed}` with `Name` the short name and `Committed` the commit's committer time, sorted by `Committed` and then `Name`.

`Log` walks the commits reachable from `tip`, newest first, as `LogCommit{SHA, Parents, Tree, Committed, Message, Paths}`; `tip` must be a valid hex object name, else `ErrInvalidSHA`.
`LogQuery` shapes the walk:
- `FirstParent` follows only each commit's first parent, which is D10's walk of the parent branch; without it the walk is go-git's committer-time order over every parent.
- `Since`, when non-zero, ends a first-parent walk at the first commit committed before it and drops such commits from a full walk.
- `PathPrefix`, when non-empty, keeps only commits whose tree changes a path equal to or under the slash-separated prefix against their first parent, every path for a root commit, and fills `Paths` with those changed paths, sorted, a deleted path included; with an empty prefix `Paths` stays nil and no tree is diffed.
- `Limit`, when positive, caps the number of commits returned.
This is the one walk the report makes over the records branch with the shed runs root as prefix, over the parent branch first-parent, over the board branch, and from an archive tag to its newest commit carrying a trailer.

`SubdirsAtRevision` returns the names of the directories directly in `dir` in `rev`'s tree, sorted, the directory counterpart of `FilesInDirAtRevision`: an absent directory is an empty result, `""` or `.` names the root, and an invalid `rev` is `ErrInvalidSHA`.

`doc.go`'s read-surface sentence and its Repo API list name the three reads as go-git local reads.

`TestReportReads` (new, in `reportreads_integration_test.go`, package `gitrepo_test`, `//go:build integration`) builds one repository with `newRepo`, `writeFile`, `gitkit.Git` and `commitAllAt` from `blobread_integration_test.go`, each commit at a fixed committer date: commit A adds `runs/a/status.json` holding the four lines `l1`, `l2`, `l3`, `l4`, B rewrites it to `l1`, `L2`, `l4` and adds `docs/x.md`, a side branch adds `side.txt` in S, and M merges S into the main line with `--no-ff`; an annotated tag `archive/a/1` points at A, a lightweight `archive/b/2` at M and `other` at B.
Its steps run serially over the shared repository:
- `TagRefs("archive/")` returns the two archive tags, the annotated one peeled to A, ordered by date, and `TagRefs("nope/")` returns none.
- `Log(M, LogQuery{})` returns M, S, B and A; with `FirstParent` it returns M, B and A; with `FirstParent` and `Since` at B's date it returns M and B; with `PathPrefix: "runs"` it returns B and A, each with `Paths` `[runs/a/status.json]`; with `Limit: 1` it returns M alone, whose `Parents` name B and S; `Log("not-a-sha", LogQuery{})` is `ErrInvalidSHA`.
- `SubdirsAtRevision(B, "runs")` returns `[a]`, `SubdirsAtRevision(B, "")` returns `[docs runs]`, an absent directory returns none, and an invalid rev is `ErrInvalidSHA`.

`cmd/lyx/testdata/test-budget.yaml` raises `internal/gitrepo` to 45.

**ImpactSummary:** Additive reads in a new file plus doc lines; no existing `gitrepo` method or caller changes.

**Commit:** `8: gitrepo tag, log and subdirectory reads`
