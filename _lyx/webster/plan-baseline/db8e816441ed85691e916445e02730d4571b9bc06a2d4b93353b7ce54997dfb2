# Card 2 — gitrepo reports commits the upstream holds that HEAD lacks

**Create:**
- `internal/gitrepo#Repo.HasUnpulled`

**Edit:**
- `internal/gitrepo/push.go`
- `internal/gitrepo/doc.go`
- `internal/gitrepo/push_test.go`
- `cmd/lyx/gitrepoboundary_test.go`

**Uses:**
- `internal/gitrepo#Repo.HasUnpushed`

**Intent:**
Add `HasUnpulled`, the mirror of `HasUnpushed`: it reports whether the branch's upstream holds commits HEAD lacks, through `rev-list --count HEAD..@{u}` on `runChecked`, against the remote-tracking ref as last fetched.
A git non-zero exit (no upstream configured) folds into `(true, nil)`, the conservative answer for its one consumer: card 3's push retry reads `true` as "the remote holds something local lacks" and leaves the rejection alone.
A spawn failure returns `(false, err)`, as `HasUnpushed` does.
It is CLI-bound for the same reason as its mirror `HasUnpushed`, which `doc.go` already lists among the `runChecked` methods.
It sits beside `HasUnpushed` in `push.go`, and `doc.go`'s list of CLI-bound methods on `runChecked` gains it.
`cmd/lyx/gitrepoboundary_test.go` pins the set of `Repo` methods that call `r.run` or `r.runChecked` (`PATTERN-gitrepo-client-boundary`), so `HasUnpulled` joins `gitrepoPinnedRunBoundMethods` in this card; without that entry the boundary guard fails at this card's commit.

Extend `push_test.go` with a table-driven test over the existing bare-remote and clone fixtures: an upstream equal to HEAD, an upstream that is an ancestor of HEAD (local ahead), and an upstream that another clone advanced and that this clone fetched, each with its expected answer; and a branch with no upstream answering `true`.

**ImpactSummary:** additive: one new `Repo` method in `internal/gitrepo`, its doc listing and one entry in the boundary guard's pinned set; no existing method or caller changes.

**Commit:** 2: gitrepo: HasUnpulled
