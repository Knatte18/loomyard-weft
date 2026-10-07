# Card 6 — Report cost and quality per batcher profile

**Create:**
- `tools/tokencount/profile.go`
- `tools/tokencount/profile_test.go`

**Edit:**
- `tools/tokencount/main.go`
- `tools/tokencount/calibrate_integration_test.go`
- `internal/shedadapters/burler.go`

**Uses:**
- `internal/websterengine/state.go`
- `internal/burlerengine/verdict.go`
- `internal/loomengine/config.go`

**Intent:**
Implement D11: `-calibrate` adds a per-batch cost table and a per-profile summary, read from the one repository card 3's flag names, through `gitrepo`'s read side only (`PATTERN-fabric-git`, `PATTERN-gitrepo-client-boundary`).

Reading a run, in `profile.go`:
- The run's webster records are the commits whose subject is `loom: webster run record for <slug>` (declare the prefix as a constant beside `planCommitSubjectPrefix`); the last record is the newest.
  Read `state.json` under `websterengine.DirRel()` at each record and decode it into `websterengine.State` with `encoding/json`'s default, lenient decoding, so a state with an older breakdown shape still reads.
- More than one distinct `RunGUID` across the records means the run was restarted with `--fresh`: list it in a marked row with that reason, and it enters no profile's summary line.
- A record whose state cannot be read or decoded keeps the run's row with the reason and the missing columns marked, never a silent zero, and the run enters no profile line, having no partition to name one.
- A batch's profile is its `PartitionBatch.Profile` in the last state; a run with no recorded partition counts as `identity`, one batch per card, with its cards counted from the run's plan commit (read through the plan-read function card 5 extracts from `calibrate.go`, not a second copy).
- A batch's weight and measured peak come from the run's forks whose transcript file name is among the batch record's `ForkTranscripts` (compare base names); failed batches and recoveries come from the batch records, attributed per batch: a record of kind `recovery` is one recovery, and a batch is failed when its record's terminal status is failed, dead or stuck or its kind is `recovery`.
- Webster-Review findings: list `round-<n>-review.md` files directly in the webster review directory (`loomengine.LoomReviewsDirRel()` joined with the `webster` run subdirectory, slash-separated) at the last record, recognised by `shedadapters.ParseRoundReviewName`, and count each file's findings through `burlerengine.ParseReview`, the sole reader of that format; `prior-generation/` is a subdirectory and is not listed.
  No such file marks the run's findings not read, and the run contributes none; a file that does not parse marks them not read with the error.

Report:
- The per-batch table: run, batch, profile, cards, measured peak, weight.
- One row per run: its profiles, batches, cards, failed batches, recoveries, findings and findings per card, or the marked reason.
- One summary line per profile: runs, batches, cards, weight per card, failed batches, recoveries and Webster-Review findings per card.
  Per-batch figures go to each batch's own profile, so a run whose batches name several profiles contributes to each.
  A run contributes findings only when its batches name one profile, it was not restarted with `--fresh` and its review files were read; a run whose batches name several profiles shows its findings per card in its own row only.
  A profile line's findings per card is the findings over the cards of the contributing runs alone, and the line states how many runs those are; with none it shows not available, never zero.
- The tool never edits `batcher.yaml`.

`internal/shedadapters/burler.go`: export `parseRoundReviewName` as `ParseRoundReviewName`, its one in-package caller following, so tokencount does not restate the round file name shape.
`main.go` writes the new sections after the calibration section, and its package comment describes them, the profile attribution rule, the `--fresh` and unreadable marks and where findings come from.

Tests:
- `profile_test.go`, over an in-memory history: a two-profile run counted per batch in each line with its findings in its own row only; a run without a partition counted as `identity` with cards from its plan commit; a run whose records carry two run guids marked and left out; an unreadable state marked with its reason and left out; a run with no review files marked not read; a profile line with no contributing run showing not available; weight per card, failed batches and recoveries from batch records and fork transcripts.
  Comments and strings in the test must not name the fabric side (`PATTERN-fabric-vocabulary`).
- `calibrate_integration_test.go`: the git fixture also commits a webster record and one round review file, and the git-backed profile report equals the in-memory one.

**ImpactSummary:** tokencount gains a report section; `shedadapters` exports one existing name-parsing helper without changing it.
