# Card 3 — output-round

**Create:**
- `plan:internal/shedadapters#OutputRound` -> `func OutputRound(path string) (int, bool)`

**Edit:**
- `internal/shedadapters#ReviewRounds`
- `internal/shedadapters#TestReviewRounds`

**Intent:** D2: the checklist maps a review-row history entry's output to its round "through `shedadapters`, the package that owns the round file names, never by a second parse of its own".
Add `OutputRound` in `internal/shedadapters/round.go`: it reads the base name of path and returns N and true when the name is `round-<N>-<kind>` with N a positive integer, and 0 and false otherwise (an empty path, a name without the `round-` prefix, a non-numeric or non-positive N).
`ReviewRounds` takes its round number and kind from the same parse, a private helper both call, so the two cannot disagree on what a round file name is.

`TestReviewRounds` gains `OutputRound` rows over the names its cases already hold and over absolute paths: `/x/round-3-review.md` and `round-3-fixer-report.md` give 3, `round-0-review.md`, `round-x-review.md`, `notes.md` and "" give false.

**ImpactSummary:** ReviewRounds keeps its answer for reportengine's two callers; one new exported reader in shedadapters' round file.
