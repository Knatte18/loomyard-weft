# Card 1 — gitrepo-tags-under

**Create:**
- `plan:internal/gitrepo#TagRef` -> `type TagRef struct`
- `plan:internal/gitrepo#Repo.TagsUnder` -> `func (r *Repo) TagsUnder(prefix string) ([]TagRef, error)`

**Edit:**
- `internal/gitrepo_test#TestHistoryReads`

**Uses:**
- `internal/gitrepo#readGoGit`
- `internal/gitrepo#branchAndTagCommits`

**Intent:** Add a go-git tag listing to `gitrepo`'s read side so the token counter can find a run's `archive/<slug>/<sha>` tags and their commits without running git.
`TagRef` carries the tag's short name (`archive/<slug>/<sha>`, without `refs/tags/`), the full hex SHA of the commit it reaches, annotated tags peeled the way `branchAndTagCommits` peels them, and the commit's committer time.
`TagsUnder(prefix)` returns every tag whose short name starts with `prefix`, sorted by name, in one `readGoGit` closure that resolves the refs, the peeled tags and the commits from the handle it is given and calls no other `Repo` method.
A tag reaching a non-commit object is skipped, and an empty prefix lists every tag.
It lives in `blobread.go` beside `CommitsWithSubject`, and that file's header comment names it.
Extend `TestHistoryReads` with one step: beside the existing lightweight `v1` tag, add an annotated tag and a second lightweight tag under a shared prefix, two of them on commits made with `commitAllAt` so their committer times differ, and assert that the prefix filter keeps only the prefixed tags, that each returned SHA is the full commit SHA with the annotated tag peeled, that the committer times match the dates given, and that the empty prefix lists every tag.

**ImpactSummary:** Adds one read-only tag listing and its test step; no existing gitrepo caller or behavior changes.

**Verify:** lyx gate test --tags integration ./internal/gitrepo
