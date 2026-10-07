# Card 2 — gitrepo reports commits the upstream holds that HEAD lacks

**Create:**
- `plan:internal/gitrepo#Repo.HasUnpulled` -> `func (r *Repo) HasUnpulled() (bool, error)`

**Edit:**
- `internal/gitrepo/push.go`
- `internal/gitrepo/doc.go`
- `internal/gitrepo/push_test.go`

**Uses:**
- `internal/gitrepo#Repo.HasUnpushed`
- `internal/gitrepo#Repo.Fetch`

**Intent:**
Add `HasUnpulled`, the mirror of `HasUnpushed`: it reports whether the branch's upstream holds commits HEAD lacks, through `rev-list --count HEAD..@{u}` on `runChecked`, against the remote-tracking ref as last fetched.
A git non-zero exit (no upstream configured) folds into `(true, nil)`, the conservative answer for its one consumer: card 3's push retry reads `true` as "the remote holds something local lacks" and leaves the rejection alone.
A spawn failure returns `(false, err)`, as `HasUnpushed` does.
It sits beside `HasUnpushed` in `push.go`, and `doc.go`'s list of CLI-bound methods on `runChecked` gains it.

Extend `push_test.go` with a table-driven test over the existing bare-remote and clone fixtures: an upstream equal to HEAD, an upstream that is an ancestor of HEAD (local ahead), and an upstream that another clone advanced and that this clone fetched, each with its expected answer; and a branch with no upstream answering `true`.

**ImpactSummary:** additive: one new `Repo` method in `internal/gitrepo` and its doc listing; no existing method changes.

**Commit:** 2: gitrepo: HasUnpulled
