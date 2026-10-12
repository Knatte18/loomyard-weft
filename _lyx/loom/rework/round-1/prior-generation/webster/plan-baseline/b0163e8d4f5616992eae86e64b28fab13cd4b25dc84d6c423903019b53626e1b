# Card 9 — gitrepo-landing-reads

**Create:**
- `plan:internal/gitrepo#Repo.PreImageLines` -> `func (r *Repo) PreImageLines(sha string) (map[string][]int, error)`
- `plan:internal/gitrepo#Repo.BlameLines` -> `func (r *Repo) BlameLines(rev, relPath string, lines []int) (map[int]string, error)`

**Edit:**
- `plan:internal/gitrepo_test#TestReportReads`
- `//internal/gitrepo/doc.go`

**Intent:**
D11 blames, for each `fix` and `unmapped` landing, the pre-image lines it modifies or deletes, at the landing's first parent, attributing each line to the first-parent commit that brought it.
D3 names the blame read as one `gitrepo` gains; both reads land in a new file, `internal/gitrepo/landingreads.go`.

`PreImageLines` diffs `sha`'s tree against its first parent through go-git inside `readGoGit` and returns, per slash-separated path the commit modifies or deletes, the ascending 1-based line numbers of the first parent's version that the commit removes or replaces.
A path the commit only adds has no entry, a binary file has no entry, a root commit returns an empty map, and an invalid `sha` is `ErrInvalidSHA`.
Which paths count is the report's choice (card 13), not this read's.

`BlameLines` runs `git blame --porcelain --first-parent` over `relPath` at `rev`, one `-L <n>,<m>` per run of consecutive requested lines, through `r.runChecked`, and returns each requested line's attributing commit SHA.
It is CLI-bound, the one local read beside `IsAncestor` and `CommitsNotIn` that is: go-git's blame walks the whole history of the file per call and has no first-parent mode, which D11 needs so a merged task branch's lines are attributed to the merge that landed them.
`rev` is validated as a hex object name and `relPath` is refused when it starts with `-`, both before any spawn (`ErrInvalidSHA`); an empty `lines` returns an empty map without spawning.
`doc.go` adds `BlameLines` to its list of `runChecked` methods with that reason and `PreImageLines` to the go-git read surface.

`TestReportReads` gains a commit and two steps.
After its existing fixture, a commit F on the main line, at a later fixed date, rewrites `runs/a/status.json` from `l1`, `L2`, `l4` to `X1`, `L2`, `l4`:
- `PreImageLines(B)` returns `{"runs/a/status.json": [2, 3]}` and no `docs/x.md` entry; `PreImageLines(F)` returns `{"runs/a/status.json": [1]}`; `PreImageLines(A)` returns an empty map; `PreImageLines(M)` against its first parent B returns an empty map, since S only added a file.
- `BlameLines(M, "runs/a/status.json", []int{1, 2, 3})` attributes lines 1 and 3 to A and line 2 to B; `BlameLines(M, "side.txt", []int{1})` attributes the side branch's line to M, not S, which pins `--first-parent`; an empty `lines` returns an empty map; a rev of `not-a-sha` and a path of `-x` are each `ErrInvalidSHA`.

**ImpactSummary:** Additive reads plus one more test commit after the merge, which no step of card 8 reads, since each of those names its revision.

**Commit:** `9: gitrepo pre-image line and first-parent blame reads`
