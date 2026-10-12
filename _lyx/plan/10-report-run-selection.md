# Card 10 — report-run-selection

**Create:**
- `plan:internal/reportengine#Geometry` -> `type Geometry struct`
- `plan:internal/reportengine#Reader` -> `type Reader interface`
- `plan:internal/reportengine#Sources` -> `type Sources struct`
- `plan:internal/reportengine#Open` -> `func Open(g Geometry) (Sources, error)`
- `plan:internal/reportengine#Options` -> `type Options struct`
- `plan:internal/reportengine#Report` -> `type Report struct`
- `plan:internal/reportengine#Selection` -> `type Selection struct`
- `plan:internal/reportengine#RunRow` -> `type RunRow struct`
- `plan:internal/reportengine#PartialRun` -> `type PartialRun struct`
- `plan:internal/reportengine#Finding` -> `type Finding struct`
- `plan:internal/reportengine#Compute` -> `func Compute(src Sources, opts Options) (Report, error)`
- `//internal/reportengine/doc.go`
- `//internal/reportengine/fake_test.go`
- `//internal/reportengine/runs_test.go`
- `//internal/reportengine/seam_enforcement_test.go`
- `//internal/reportengine/testmain_test.go`

**Edit:**
- `//pattern/PATTERN-told-geometry.md`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#ParseSeedTolerant`
- `plan:internal/shedrun#TreeRunRels`
- `plan:internal/shedengine#ParseStatusTolerant`
- `plan:internal/fabricengine#ArchiveTagSlug`
- `plan:internal/fabricengine#ArchiveTagPrefix`
- `plan:internal/gitrepo#Repo.Log`
- `plan:internal/gitrepo#Repo.TagRefs`
- `internal/shedrun#RunsRootRel`

**Intent:**
The engine of `lyx report convergence` (D1) starts here: told geometry, one ref snapshot, the run unit, the batten span and the absent-is-unknown rule (D2, D3, D4, D5).
`internal/reportengine` never imports `internal/lyxcwd`, takes every path from `Geometry`, writes nothing, takes no lock and makes no commit.
No production file in it spells `_lyx`, `.lyx` or a fabric side token: record paths come from `shedrun`, `loomengine` and the other record packages, and fabric names come through `fabricengine`'s neutral exports (`ArchiveTagPrefix`, `ArchiveTagSlug`, `TaskCommitOf`).

Surface, in `source.go`:
- `Geometry` holds `CodeRepo`, `RecordsRepo` and `BoardRepo` (absolute checkout paths) and `ParentBranch`, `RecordsBranch` and `BoardBranch`.
- `Reader` is the read set the engine uses, which `*gitrepo.Repo` satisfies: `FileAtRevision`, `FilesInDirAtRevision`, `SubdirsAtRevision`, `Log`, `PreImageLines` and `BlameLines`.
- `Sources` holds a `Reader` per repository (`Code`, `Records`, `Board`), the three branch names, the tips they resolved to (`ParentTip`, `RecordsTip`, `BoardTip`) and `Tags`, the archive tags.
- `Open` builds a `gitrepo.Repo` per path, resolves each `refs/heads/<branch>` once through `RefSHA` and lists the archive tags once through `TagRefs(fabricengine.ArchiveTagPrefix)`, so every later read sees one snapshot while runs land concurrently (D3); an unresolvable branch is an error naming it.
  `Open` is wiring: card 15's hubforge test runs it, and `Compute` is tested over a fake `Reader`.

Surface, in `report.go` and `runs.go`:
- `Options` holds `Since` (a UTC date; the CLI defaults it) and `FreshDays`.
- `Compute` returns a `Report`: `Selection` (the branches, tips, `Since` and `FreshDays`), `Runs []RunRow`, `Partial []PartialRun` and `Findings []Finding`; later cards add fields.
  A `Finding` names the run's tag, the tree-relative file and the error.
- Every tag whose slug `ArchiveTagSlug` parses is one run, so a reused slug yields one run per tag (D4); a tag it cannot parse is a finding and is skipped.
  A tag committed before `Since` is left out entirely.
- At the tag's commit the engine reads the loom seed and status at `shedrun.TreeRunRels(slug)` through `ParseSeedTolerant` and `ParseStatusTolerant`, never `ParseSeed` or `ReadSeed` (D5).
  The run is full when the seed's recipe is `shedrun.RecipeLoom` and the status history holds a `loomshed.NameFinalize` entry, the rule `tools/tokencount` applies; otherwise it is a `PartialRun` (tag, slug, week, reason) that enters no per-run figure.
  A seed or status that exists but does not decode is a finding naming the tag and the file, and the tag is partial.
- A run's time is its tag's committer date in UTC, and `RunRow.Week` is its ISO week (`time.ISOWeek`, Monday start) spelled `2026-W41`.
- `RunRow` carries `Tag`, `Slug`, `Commit`, `Committed`, `Week` and `Sources`, a map from source name to whether the run holds it; a source a run lacks is reported absent, never as zero (D5), and later cards add their sources and figures.

The batten source (D4), in `batten.go`:
one `Log(RecordsTip, LogQuery{PathPrefix: <slash form of shedrun.RunsRootRel()>})` walk lists every records commit touching a slug's `status.json`, and the engine reads the file at each, a commit that deleted it holding nothing.
A snapshot is a fresh seed when its history's first entry is `battenrecipe.NameWorktreeCreate` and the slug's previous commit touching the file holds no file or a longer history.
The run's own fresh seed is the newest fresh seed for the slug committed after the slug's previous tag's commit and at or before this tag's commit; its span runs up to the slug's next fresh seed, or to `RecordsTip` when none follows.
The run's batten record is the newest snapshot in the span holding the file, so a run directory `batten cleanup` removed still reads from the last commit that held it, and the run keeps every snapshot in its span, oldest first, for card 11's classifier.
No fresh seed in the interval, or no snapshot in the span holding the file, leaves the batten source absent; a snapshot that does not decode is a finding naming the tag and the file and is skipped.

Tests:
`fake_test.go` holds `fakeRepo`, an in-memory `Reader`: files per commit SHA, a commit list per tip that `Log` filters by `FirstParent`, `Since`, `PathPrefix` (computing `Paths` from the per-commit files against the first parent) and `Limit`, directory listings derived from the file maps, and maps standing in for `PreImageLines` and `BlameLines`; plus builders for a tag, a loom seed and status, and a batten snapshot commit.
`TestCompute_SelectsRuns` (in `runs_test.go`) is table-driven over fake sources, each row asserting the exact runs, partial runs, batten presence and snapshot sequence, and findings:
- no archive tags yields a report with no runs, no partial runs and no findings, and no error;
- a slug with two tags yields two runs, each with its own batten snapshots, when the second run's batten commits precede its tag;
- a slug whose first tag came from a batten run and whose second came from a run batten never seeded reports the batten source absent for the second tag, and the first tag's snapshots appear on the first run only;
- a slug whose `status.json` a later commit deleted reads its batten record from the last commit holding it;
- a tag whose loom history has no Finalize entry (parked after Discussion-Write) and a tag whose seed names `darn` are each partial and appear in no run;
- a loom `status.json` that does not decode yields a finding naming the tag and the status path, the tag is partial, and `Compute` returns no error; a seed and a status each carrying an unknown key read, and the run is full;
- a tag dated before `Since` is in neither runs nor partial runs;
- tags at Sunday 23:59:59 UTC and at Monday 00:00:00 UTC fall in consecutive ISO weeks, and one at Monday 00:30 +02:00 falls in the earlier week.
`seam_enforcement_test.go` holds `TestToldGeometryInvariant_AllowlistOnly`: `scankit.AssertImportAllowlistNoStale` over `internal/reportengine` with the allowlist `gitrepo`, `shedrun`, `shedengine`, `loomshed`, `battenrecipe` and `fabricengine`, which every later engine card extends with what it imports.
`testmain_test.go` runs the package under `tmuxkit.Main`.

`pattern/PATTERN-told-geometry.md`'s bound-packages list gains `reportengine` (D2).
`doc.go` states the package's purpose, its told geometry, the single ref snapshot and the run unit; each later engine card adds its part.
`cmd/lyx/testdata/test-budget.yaml` gains `internal/reportengine: 2`.

**ImpactSummary:** A new package with no importer yet; the pattern file gains one package name.

**Commit:** `10: report engine: run selection and batten span`
