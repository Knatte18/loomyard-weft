# Card 9 — round-row

**Create:**
- `plan:internal/shedadapters#ReviewRound` -> `func ReviewRound(runDir string) (int, error)`
- `plan:internal/shedadapters#FixedRound` -> `func FixedRound(runDir string) (int, error)`
- `plan:internal/shedadapters#TestReviewRound` -> `func TestReviewRound(t *testing.T)`
- `plan:internal/lymdirs#HardenDirName` -> `const HardenDirName = "harden"`
- `plan:internal/hardenshed#RowKind` -> `type RowKind string`
- `plan:internal/hardenshed#RowReview` -> `const RowReview RowKind = "review"`
- `plan:internal/hardenshed#RowFix` -> `const RowFix RowKind = "fix"`
- `plan:internal/hardenshed#RowDeps` -> `type RowDeps struct`
- `plan:internal/hardenshed#WrapRow` -> `func WrapRow(inner shedengine.ShedProducer, deps RowDeps) shedengine.ShedProducer`
- `plan:internal/hardenshed#ArchiveDeps` -> `type ArchiveDeps struct`
- `plan:internal/hardenshed#ArchiveRound` -> `func ArchiveRound(deps ArchiveDeps, round int) error`
- `plan:internal/hardenshed#ArchiveFinalLedger` -> `func ArchiveFinalLedger(deps ArchiveDeps) error`
- `plan:internal/hardenshed#TestWrapRow` -> `func TestWrapRow(t *testing.T)`
- `plan:internal/hardenshed#TestArchiveRound` -> `func TestArchiveRound(t *testing.T)`

**Extends:**
- `internal/shedadapters/doc.go`
- `internal/lymdirs/doc.go`
- `internal/hardenshed/doc.go`

**Intent:**
The round row decorator that reaps around each Harden-Burler row and writes the round's Go files, and the round archive (D5, D6, D7).

`shedadapters.ReviewRound` returns the highest round whose review file exists in the run directory, and `FixedRound` the highest round whose fixer report exists, each 0 for none, through `ResolveRound` and the file names this package owns; `TestReviewRound` is tier0 over zero, one and three review files and fixer reports.

`RowDeps` carries the row name, `RowKind`, the run directory, `Reap func(row string, point ReapPoint) ([]StrayEntry, error)`, `Models func(round int) (review, fix string)`, `Head func() (string, error)`, `Commits func(from string) ([]string, error)` and the `ArchiveDeps`.
`WrapRow` returns a producer whose `Call` fixes its round once, at its start, and uses it for every file of the call: `FixedRound` plus one for a review row, so a review row resumed after it wrote its review keeps that review's round, and `ReviewRound` for a fix row. Then it:
1. Reaps at `ReapStart` and appends the entries to the round's stray report, `harden-strays-<round>.json`.
2. On a review row, writes `harden-models-<round>.json`, the round's reviewer and fixer models from `Models`; on a fix row, reads `Head`.
3. Calls the wrapped producer.
4. On a fix row whose call returned no error, writes `harden-commits-<round>.json`, the commits `Commits` lists past the head read in step 2.
5. Reaps at `ReapEnd`, appending to the stray report, whatever the wrapped call returned.
6. On a fix row whose call returned no error, calls `ArchiveRound`.
It returns the wrapped producer's outcome and pointer.
A reap, file or archive error is joined to the wrapped call's error, never dropped, and names its way forward: "run lym harden sandbox clear in the task worktree, then lym loom resume" for a reap error, "lym loom resume" for any other.
A resumed fix row runs its round again and archives it again; `ArchiveRound` overwrites the round's directory and commits only what changed, so a second archive of one round is harmless.

`ArchiveDeps` carries the run directory, the worktree root and `Commit func(relPaths []string, message string) error`, which card 14 wires to `fabricengine.CommitAnchoredPaths` (PATTERN-fabric-boundary).
`ArchiveRound` copies the round's files from the run directory, every `round-<n>-*` file (the review and the fixer report) and every `harden-*-<n>.json` file, with the latest bouncer ledger, into `_lym/harden/round-<n>/`, named through `lymdirs.HardenDirName`, and commits that directory.
`ArchiveFinalLedger` copies the latest ledger into `_lym/harden/` and commits it; card 11 runs it as the `harden` commit seam on Harden-Bouncer's approved settle.

`TestWrapRow` is tier0 with a fake inner producer and fake seams:
- a review row reaps at start and end and writes the models file, both reaps into the same round's stray report;
- a review row whose round's review file already exists, its fixer report not, keeps that round;
- a fix row writes the commits past its start head, reaps, and archives;
- a stray left by the inner call appears in the round's stray report with its row and the end point;
- a start reap's entries are marked interrupted;
- an inner error still reaps at the end and archives nothing;
- a failing reap and a failing archive each come back joined to the call's result with their way forward.
`TestArchiveRound` is tier0: the round's files land under `_lym/harden/round-<n>/` and the commit seam is called with that path; a second archive of the same round succeeds; the final ledger lands under `_lym/harden/`.

**ImpactSummary:** Two exported reads in shedadapters, one lymdirs name and new hardenshed code; no existing producer changes.
