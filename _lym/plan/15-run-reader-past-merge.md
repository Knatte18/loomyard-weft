# Card 15 — run-reader-past-merge

**Edit:**
- `internal/pairteardown#runReader.read`
- `internal/pairteardown#TestReadRun`

**Uses:**
- `internal/pairteardown/doc.go`
- `internal/pairteardown#runReader`
- `internal/pairteardown#hubRunReader`
- `internal/pairteardown#runFakes`

**Intent:** D5: a resume after the local merge never re-drives it, and no reader takes the run as past its merge while the parent it judges lacks the work.

`runReader.read`'s past-the-merge read takes the local parent whenever origin's parent lacks the work while the local parent holds it,
which covers every landing whose parent push the parent target still owes, refused or not:
it calls `originParentHolds`, and on false `localParentHolds`, and a run whose either parent holds the work reads past its merge.
With pushes off it reads the local parent alone, as today.
A batten record at Land or Worktree-Teardown reads the run as past its merge only when that same read finds the work on a parent, while the pair stands here;
with the pair gone, which D13 allows only once origin holds both tips and no landing entry stands, it reads past the merge as today.

`TestReadRun` gains tier0 rows, adding under a second to one tier0 run:
a child at Finalize whose origin parent lacks the work and whose local parent holds it reads past its merge, where the merge base reads not past it;
batten at Land with the pair here and neither parent holding the work reads not past its merge;
batten at Land with the pair gone reads past its merge.

**ImpactSummary:** Changes one derived input of the run FSM, read by every verb that judges a run; no reader's signature changes.
