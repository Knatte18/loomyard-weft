---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/01-shedrun-tree-records.md (Tests, TestParseSeed)
    summary: TestParseSeed plans four rows (started_at, retired parent key, absent driver, unknown key) that existing ReadSeed tests already cover once ReadSeed delegates to ParseSeed, violating PATTERN-test-economy
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/04-shed-span-producers.md (Tests, TestShedRecordSpanAndProducers)
    summary: the timestamp-parse abort for started_at, at and last_at has no named test input, and the absent-seed branch has none either
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/02-turn-time-by-role.md (Tests, TestRunTimeByRole)
    summary: no planned row pins that All runs sums per-run role times, idle and spans with no union across runs, nor the report's explanatory paragraph
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/01-shedrun-tree-records.md (Intent, doc.go paragraph)
    summary: card 1 adds TreeRunFileRels, a path.Join with no Location, but leaves the shedrun package doc and paths.go header claiming every constructor is a filepath.Join onto a told Location's AnchorPath
---

### [BLOCKING] TestParseSeed repeats behavior existing tests already cover

**Location:** `_lyx/plan/01-shedrun-tree-records.md`, the `TestParseSeed` bullet under Tests.

**Issue:** Card 1 makes `ReadSeed` call `ParseSeed` for all decoding, defaulting and validation, so every existing `ReadSeed` test of those behaviors reaches `ParseSeed`'s code.
Four of the six planned `TestParseSeed` rows repeat a test that already exists in `internal/shedrun/seed_test.go`:

- a seed carrying `started_at` decodes with it: `TestWriteSeed_ReadSeed_RoundTrip`'s row "a given start stamp is kept verbatim";
- a seed carrying the retired `parent` key decodes and drops it: `TestSeed_LegacyParentKey`'s first subtest;
- an absent driver decodes as `DriverGo`: `TestReadSeed_AbsentDriverDefaultsToGo`;
- an unknown key returns an error: `TestSeed_LegacyParentKey`'s subtest writing `"bogus":"x"`.

The remaining two rows, an unknown driver and an empty params key each returning an error, are new: a grep of `internal/shedrun/*_test.go` finds no test reaching either branch of `ReadSeed` today.
`PATTERN-test-economy` is a listed constraint, so the four repeated rows are a blocking violation even though they would pass.

**Fix:** Keep `TestParseSeed` as a table with only the unknown-driver and empty-params-key rows, plus at most one accepting row that pins `ParseSeed`'s happy-path return.
Drop the `started_at`, `parent`, absent-driver and unknown-key rows.
The budget raise of one top-level test stays.

### [MEDIUM] Card 4's timestamp-parse abort and absent-seed branch have no named test

**Location:** `_lyx/plan/04-shed-span-producers.md`, the `readShedRecord` paragraph and the Tests list.

**Issue:** The Intent says `readShedRecord` aborts with an error naming the slug, the tag and the file when "a timestamp in either that fails to parse", meaning `started_at` in the seed and `at` or `last_at` in a history entry.
The Tests list names only "a status and a seed that fail to decode", which reaches the JSON-decode branch and not the RFC 3339 parse branch: valid JSON with `"at":"yesterday"` decodes cleanly into `shedengine.Status` and fails only at the parse.
None of a bad `started_at`, a bad `at` or a bad `last_at` on a folded entry is a named input.
The Intent also gives `StartedAt` as zero "when the seed or its field is absent".
The listed case "a seed without `started_at`" reaches the absent-field branch, but no row has a tag holding `status.json` and no `seed.json`.
On that branch, `FileAtRevision`'s `ErrPathNotAtRevision` for the seed must be treated as absence, not as an abort.

**Fix:** Add rows to `TestShedRecordSpanAndProducers`: a seed whose `started_at` is not RFC 3339, a status entry whose `at` is not RFC 3339, and a folded entry (`repeats` > 0) whose `last_at` is not.
Each aborts with an error naming the slug, the tag's `Name` and the file.
Also add a row with a tag holding `status.json` and no `seed.json`, whose span starts at the earliest transcript line.

### [LOW] TestRunTimeByRole does not pin the "All runs" sums or the explanatory paragraph

**Location:** `_lyx/plan/02-turn-time-by-role.md`, the `TestRunTimeByRole` bullets.

**Issue:** Decision 7 and its acceptance criterion require "All runs" to sum each run's role times, idle and span, with no union across runs.
The card's Intent carries that, but every planned row is about one run, so nothing tells a correct sum from a union over two runs whose working intervals overlap in wall-clock time.
The row would also catch time shares divided by one run's span instead of the summed spans.
The card's explanatory paragraph below the "All runs" table, which holds Decision 4's and Decision 7's required statements, is also asserted by no row.

**Fix:** Add a row with two runs whose working intervals overlap in wall-clock time.
It asserts that "All runs" gives each role the sum of the per-run times, gives idle the sum of the per-run idle, and divides time share by the sum of the spans.
Assert the explanatory paragraph's presence in the same `WriteMarkdown` output.

### [LOW] shedrun's package doc and paths.go header are left stating a claim TreeRunFileRels breaks

**Location:** `_lyx/plan/01-shedrun-tree-records.md`, the paragraph on `doc.go`.

**Issue:** `internal/shedrun/doc.go` ends with "Every constructor in this package is a plain filepath.Join onto a told *lyxcwd.Location's AnchorPath()".
The `internal/shedrun/paths.go` header says "Every constructor is a plain filepath.Join onto the given *lyxcwd.Location's AnchorPath()".
`TreeRunFileRels` takes no `Location` and joins with `path.Join`.
The card adds one sentence about it to `doc.go` but does not amend either claim, and it does not name the file `TreeRunFileRels` goes in.
So the package doc would describe the new constructor and, a few lines later, say no constructor like it exists.
`RunsRootRel` already breaks the claim, which the card's edit is the moment to fix.

**Fix:** In card 1's Intent, have the `doc.go` edit narrow the closing claim to what holds: no constructor resolves a cwd or spawns git, and the on-disk constructors join onto a told `Location`'s `AnchorPath()`.
Name the file that holds `TreeRunFileRels`; if it is `paths.go`, list `internal/shedrun/paths.go` under `Edit:` for its header.

## Focus verdicts

- **Card 1, test economy:** settled in F1. The `started_at`, `parent`, absent-driver and unknown-key rows repeat existing tests; the unknown-driver and empty-params-key rows are new behavior.
  The `TestRunsRootRel` extension for `TreeRunFileRels` extends an existing test and is not a finding.
- **Card 2, granularity:** not a defect.
  `turnPoint` is a private type serving `workingIntervals`; `endsTurn` and `workingIntervals` are helpers serving `countFile`; `tableTime` is a private type serving `writeTable` and `WriteMarkdown`.
  All four already sit in the card of the surface they serve, so no unit is split across cards.
  The granularity rule binds card boundaries, and listing each created symbol as its own `Create` glyph within one card is the format's own shape.
- **Card 4, reachability:** decode aborts, `ErrPathNotAtRevision` for `status.json`, empty history and no tags are each reached by a named row.
  The timestamp-parse abort is not; see F2.
  The `TreeRunFileRels` run-id error is a stated fail-loud wiring guard: `archivedSlugs` keys by the segment between two slashes, so no tagged slug holds a separator, and it needs no row.
- **Card 5, reachability and upstream rejection:** not a defect.
  `lyxcwd.Getwd` is `os.Getwd` and succeeds in the test binary.
  `resolveHub` runs right after it, before `lyxcwd.Resolve` and `RecordsWorktree`, so `TestRunRefuses`' temporary-directory hub with an empty weft flag reaches the refusal with no earlier check rejecting it, and `TestResolveHub` reaches it directly.
  `tools/tokencount` has no section in `contracts/specs/refusal-spec.md`.
  The `os.Stat` non-not-exist abort is a propagated raw I/O error of the kind that file groups into one transient row with no per-row test, so I hold it to that exemption's spirit and raise no finding.
- **Fidelity:** each of Decisions 1 to 10 and each acceptance criterion is carried by a card's Intent.
  Decisions 1, 2, 4 and 7 are in card 2; Decisions 3 and 8's newest tag in cards 3 and 4; Decisions 5, 6 and 9 in cards 1 and 4; Decisions 8 and 10 in card 5; the Scope's docs in card 6.
  The test-side gaps are F2 and F3.
  The in-flight check for picked slugs and the empty-history fallback are Shared Decisions that fill gaps the record leaves open, each with its rationale tied to Decision 8 or 3, and are licensed.
  Card 5's calibration reuse of the one `gitrepo.Repo` is wiring inside `run`, which card 5 already edits, adds no behavior, and is not scope creep.
- **Card order and verify:** no card uses a later card's symbol.
  Card 4 uses card 1's shedrun symbols, card 2's `covered` and card 3's `ArchiveTag`; card 5 uses card 3's and card 4's.
  `tools/tokencount` and `internal/shedrun` are in the root module, so `go test ./...` and `go test -tags integration ./...` cover both and `time_integration_test.go`.
  The section's order matches the plan template's.
- **Test budget:** `internal/shedrun` has 22 top-level tests against a row of 22, and `tools/tokencount` has 12 (`TestMain` included) against 12.
  Card 1 adds one (`TestParseSeed`), card 2 one (`TestRunTimeByRole`), card 4 one (`TestShedRecordSpanAndProducers`), and card 5 two (`TestResolveHub`, `TestRunReadsArchivedShedStatus`; `TestRunRefuses` renames an existing test).
  Each card's stated raise matches.
  Card 3 reworks `TestArchivedSlugs` and `TestRecentRuns` in place, adds no top-level test, and rightly does not list the budget file.

Also checked and found sound: real archive tags in the hub's weft are lightweight (`objecttype` is `commit`) with an iso-strict committer date and `_lyx/shed/<slug>/{seed,status}.json` in their tree.
That bears out cards 3 and 4's premises.
Card 1's grep claim is also true: no test asserts the old `decode seed for run` or `params key must not be empty` wording.
