# Card 7 — drift-records-guard

**Extends:**
- `internal/driftengine#ScopeRecord`
- `internal/driftengine#Paths`
- `internal/driftengine#Deps`
- `internal/driftengine/doc.go`

**Edit:**
- `internal/driftengine#NewPaths`
- `internal/loomcli#driftDeps`
- `internal/testkit/envkit#driftDeps`
- `internal/battencli#intentionallyNil`
- `internal/loomcli/doc.go`

**Create:**
- `plan:internal/driftengine#SessionMark` -> `type SessionMark struct`
- `plan:internal/driftengine#Amendment` -> `type Amendment struct`
- `plan:internal/driftengine#CheckResult` -> `type CheckResult struct`
- `plan:internal/driftengine#JudgedRecord` -> `type JudgedRecord struct`
- `plan:internal/driftengine#JudgedTuple` -> `type JudgedTuple struct`
- `plan:internal/driftengine#JudgedRange` -> `type JudgedRange struct`
- `plan:internal/driftengine#JudgedCase` -> `type JudgedCase struct`
- `plan:internal/driftengine#Records` -> `type Records struct`
- `plan:internal/driftengine#RecordsTrailerKey` -> `const RecordsTrailerKey = "Drift-Records"`
- `plan:internal/driftengine#RecordsDigest` -> `func RecordsDigest(scope, judged []byte, scopeFound, judgedFound bool) string`
- `plan:internal/driftengine#ReadRecords` -> `func ReadRecords(d Deps) (Records, error)`
- `plan:internal/driftengine#WriteRecords` -> `func WriteRecords(d Deps, change func(scope *ScopeRecord, judged *JudgedRecord) error, extra ...fabricengine.Trailer) error`
- `plan:internal/driftengine#RestoreRecords` -> `func RestoreRecords(d Deps) error`
- `plan:internal/driftengine#TestRecords` -> `func TestRecords(t *testing.T)`
- `plan:internal/driftengine#commitStore` -> `type commitStore struct`
- `plan:internal/driftengine#newCommitStore` -> `func newCommitStore(t *testing.T) *commitStore`
- `plan:internal/driftengine#commitStore.deps` -> `func (s *commitStore) deps(p Paths) Deps`

**Uses:**
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `plan:internal/fabricengine#CommitAnchoredPathsWithTrailers`
- `plan:internal/fabricengine#FindTrailerCommit`
- `plan:internal/fabricengine#AnchoredFileAt`
- `internal/driftengine#UpdateScope`

**Intent:** Add D6's guard over the drift records, `scope.json` and the new `judged.json`, and the record fields every later card writes; no row uses them yet.

- `Paths` gains `Judged` (`_lym/drift/judged.json`), `Expected` (`_lym/drift/expected.yaml`) and `Judgement` (`_lym/drift/judgement.yaml`, the file the judge writes, outside the records guard), declared in `NewPaths` beside the others.
- `ScopeRecord` gains, all `omitempty` and tolerant (`PATTERN-tolerant-records`): `ScopeSource` (the config key that set `off`, D3), `Diff` and `KeyPatterns` (recorded once, D3), `ExpectSession` and `JudgeSession` (`SessionMark`: `Open`, `OpenedAt`, `SpawnID`, D6, D9), `Amendments` (`Amendment`: `By`, `At`, `Kind` `entries`, `restore` or `discard`, and `Entries`, the amended YAML, D11), `LastCheck` (`CheckResult`: the warp `Head` checked, `Passed`, `At`, `Mode`, the counts and lists D13 renders, the compare's exit and log under `command`), `LastPassHead` and `LastPassAmendments` (the head of the last passing check and the amendment count then, D4), `Unsealed` (D6) and `SnapshotLost` (D9).
  The old fields stay readable.
- `JudgedRecord` holds `Tuples` (`JudgedTuple`: case, identity, kind, old, new, ruling, reason, strand, at, D10), `Ranges` (`JudgedRange`: case, range, the covering entry's `why`, checked, strand, at, replaced by a later listing of the same range, so the verdict renders the `why` without reading the expectation) and `Cases` (`JudgedCase`: warp head, case, ruling, reason, strand, at, for `command` mode, D2).
- `Deps` gains three seams: `CommitWithTrailers func(relPaths []string, trailers []fabricengine.Trailer) error`, `TrailerCommit func(key string, newest bool) (fabricengine.TrailerCommit, bool, error)` and `FileAt func(rev, relPath string) ([]byte, bool, error)`.
  `loomcli`'s `driftDeps` wires them to `CommitAnchoredPathsWithTrailers` (message `loom: drift records for <slug>`), `FindTrailerCommit` and `AnchoredFileAt` over the worktree's location; `envkit`'s `driftDeps` fills fakes, and `battencli`'s `intentionallyNil` lists the three as loom- and darn-only seams.
- `ReadRecords` finds the newest weft commit carrying `Drift-Records` and returns both files' copies at that commit, through `FileAt`, with `Committed` true; it sets `Mismatch` when the current files' `RecordsDigest` differs from that commit's trailer value.
  With no such commit it returns the files as they are, `Committed` false and `Mismatch` false, so a run seeded before this lands reads its records unguarded until its first write (D6).
- `WriteRecords` starts from `ReadRecords`' copies, never the current files, applies `change`, writes both files through `internal/state`, and commits both paths at once through `CommitWithTrailers` with `Drift-Records: <RecordsDigest>` and any extra trailers, such as `Drift-Baseline` (card 8).
  A write interrupted before its commit leaves a mismatch, which only `--restore` clears (D6).
- `RestoreRecords` writes both files back from the newest `Drift-Records` commit, removing a file that commit lacks, and leaves the commit history unchanged.
- `RecordsDigest` is a sha256 over each file's presence and content, so a deleted file differs from an empty one.

`commitStore` is the package's one fake in-memory weft for the three seams, shared by every later driftengine test that reads or writes the records or the seal: it keeps each commit's paths' bytes, its trailers and a settable warp HEAD, and `deps` returns a `Deps` over `p` whose `CommitWithTrailers`, `TrailerCommit` and `FileAt` read and write it; a test sets any other seam on the returned value.
`TestRecords` (tier0, on `commitStore`, `t.Parallel`) covers: a write makes one commit whose `Drift-Records` value equals the digest; a read returns the committed copies; a hand edit adding an amendment to `scope.json` sets `Mismatch` and the read still returns the committed copy; a hand-added tuple in `judged.json` sets `Mismatch`; `RestoreRecords` writes both files back and removes a file the commit lacks; with no commit the files are read as they are; an extra trailer rides on the commit; an unknown key in `scope.json` survives a write.
The doc.go gains a `# Records` section; loomcli's doc.go names the three wired seams.

**ImpactSummary:** Adds fields and seams the existing rows leave unused; three wiring sites (loomcli, envkit, battencli's nil list) gain the new seams; the beam seam test still covers `Deps`.
