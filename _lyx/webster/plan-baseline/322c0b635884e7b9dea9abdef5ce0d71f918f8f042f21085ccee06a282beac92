# Card 7 — pair-branch-snapshot

**Create:**
- `internal/gitrepo#BranchRef`
- `internal/gitrepo#Repo.BranchRefs`

**Edit:**
- `internal/gitrepo_test#TestReportReads`
- `internal/reportengine#Sources`
- `internal/reportengine#Open`

**Intent:**
Grow the report's one ref snapshot by the records repository's local branches and the snapshot time, so the timing report (card 8) sees runs in flight in the same history it reads landed runs from.

`gitrepo.BranchRef` is a local branch resolved to its tip: `Name` (the short name), `Commit` (full hex) and `Committed` (the tip's committer time), the shape of `TagRef`.
`Repo.BranchRefs` lists every local branch through `readGoGit`, as `TagRefs` reads tags, sorted by name; a branch whose tip is no commit is an error naming the branch, since a branch git wrote always names one.
`TestReportReads` in `internal/gitrepo/reportreads_integration_test.go` gains rows: after creating two branches on known commits, `BranchRefs` returns them and the repository's own branch, by name, each with its commit and committer date; after `git update-ref` points a further branch at a blob, `BranchRefs` returns an error naming that branch.

`Sources` gains `Branches []gitrepo.BranchRef`, every local branch of the records repository as `BranchRefs` lists it, and `SnapshotAt time.Time`, the UTC time the snapshot was taken.
Which of them are pair records branches is the timing selection's decision (card 8), so the filter is reached by the engine's in-memory tests.
`Open` lists them with the tips and tags, once, and stamps `SnapshotAt` with the current time after the last ref is resolved; a failed listing is an error naming the listing.
`convergence` reads neither field, so its output is unchanged.
The `reportengine` doc's "One ref snapshot" paragraph (card 8 rewrites the package doc) names both.

**ImpactSummary:** one additive read in gitrepo and two additive Sources fields; Open's only caller, reportcli, is unchanged and convergence's figures do not move.

**Commit:** 7: Snapshot the records branches with the report's refs
