# Card 20 — drift-verdict

**Edit:**
- `internal/driftengine#Verdict` -> `func Verdict(d Deps) (string, error)`
- `internal/driftengine#renderVerdict` -> `func renderVerdict(rec Records) string`
- `internal/driftengine#scopeText`
- `internal/driftengine#compareText`
- `internal/driftengine#TestVerdict`
- `internal/driftengine/doc.go`
- `internal/loomcli#landingDeps`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestLandingDeps_DriftVerdict_RendersTheScopeRecord`

**Delete:**
- `internal/driftengine#acceptedByText`

**Uses:**
- `internal/driftengine#ReadRecords`
- `internal/fabricengine#FindTrailerCommit`
- `internal/fabricengine#AnchoredFileAt`
- `internal/driftengine#commitStore`
- `internal/driftengine#commitStore.deps`

**Intent:** D13's verdict: the `## Regression check` section of the pull request and landing commit, rendered from the copies of `scope.json` and `judged.json` in the newest `Drift-Records` commit through `ReadRecords`, or from the files as they are on a run with no such commit, never running a capture.

`Verdict` takes the `Deps` whose `Paths`, `TrailerCommit` and `FileAt` it reads, and still returns "" with no scope record, creating no directory.
`landingDeps` builds them over the landing location with `FindTrailerCommit` and `AnchoredFileAt`, and so does `wire`'s Describe-table closure, the other production caller, over the worktree's location.
`renderVerdict` lists:
- the scope, and for `off` its source; a deferred capture, saying no compare ran;
- for a record whose scope says `partial`, the old scope and the cases its skip list left out;
- the diff mode, and under `command` that containment was the judge's;
- from `LastCheck`: changed cells, expected cells, cells judged as matching, cells reused from stored rulings, unexpected cells;
- every expectation entry that covered no cell;
- every open range with its `why` and the judge's sentence on what it checked under it, both from `judged.json`'s `JudgedRange`;
- every amendment with who and its entries, restore or discard.
A record with no `LastCheck` but with the old compare, acceptance or waiver fields renders those fields, each acceptance with who accepted it, under a line saying the check ran under the old Drift; a waiver is rendered as before.

`TestVerdict` (tier0, on `commitStore`) is table-driven: scope `off` with its source; a deferral; a full check's counts, uncovered entries, open ranges with `why` and sentence, and amendments; a `command`-mode check saying containment was the judge's; an old `partial` record naming its left-out cases; an old record with compare and acceptance fields rendered under the old-Drift line; a hand-edited `scope.json` after a records commit renders the committed copy; no scope record renders "".
`TestLandingDeps_DriftVerdict_RendersTheScopeRecord` seeds its record through a records commit.

**ImpactSummary:** Re-signs `Verdict`, whose two production callers, `landingDeps` and `wire`'s Describe-table closure, and the landing test change on this card; the rendered section's content changes for every landing that ran Drift.
