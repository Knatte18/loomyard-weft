---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/plan/01-shedrun-tree-records.md (Intent, doc.go and paths.go paragraph)
    summary: The narrowed doc.go and paths.go claim still reads false for SeedRel, StatusRel and DriveReportsRel, and the paths.go header's list of constructor kinds gains no tree-relative entry
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/04-shed-span-producers.md (Intent, readShedRecord paragraph)
    summary: readShedRecord is told to parse at and last_at as RFC 3339 without saying last_at is parsed only on a folded entry, so a literal reading aborts on every unfolded entry, whose last_at is omitted
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/plan/04-shed-span-producers.md (Intent, TreeRunFileRels run-id error)
    summary: The run-id validation error is propagated bare, while Decision 9 says a run id that fails validation aborts with an error naming the slug, the tag and the file
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/plan/05-run-reads-shed-status.md (Intent, resolveHub)
    summary: resolveHub compares the hub against the parent of the current directory, while Decision 8 bounds the weft default by the current worktree's parent; they differ when the tool runs from a subdirectory
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/03-archive-tag-dates.md (Tests)
    summary: archivedSlugs' skip of a line with no commit field is prescribed but no TestArchivedSlugs row reaches it
---

### [LOW] Card 1's narrowed constructor claim still leaves counterexamples

**Location:** `_lyx/plan/01-shedrun-tree-records.md`, Intent, the paragraph on `doc.go`'s closing claim and `paths.go`'s file header.

**Issue:** The focus asked whether the `doc.go` and `paths.go` claims stay true once `TreeRunFileRels` is added.
The card names `TreeRunFileRels` and `RunsRootRel` as the two constructors that break the claim that every constructor is a `filepath.Join` onto a told `Location`'s `AnchorPath()`.
But `SeedRel`, `StatusRel` and `DriveReportsRel` (`internal/shedrun/paths.go:113`, `:122`, `:134`) already break it too: they take a `*lyxcwd.Location` and read the disk through `runSegment`, but return a relative join that never touches `AnchorPath()`.
The replacement wording, "the on-disk constructors join onto a told `Location`'s `AnchorPath()`", does not say whether these Location-taking relative constructors count as on-disk.
A reader who counts them as on-disk still finds the claim false.
Separately, the first sentence of `paths.go`'s header lists the file's constructor kinds (durable, ephemeral, anchor-relative for fabric commit pathspecs, one hub-scoped lock).
`TreeRunFileRels` returns a tree-relative path for git reads, which fits none of those kinds, and the card does not update that sentence.
The header then fails the file-header contract of `scribe:code-quality`: it no longer says everything the file holds.

**Fix:** Word the narrowed claim by return shape, for example: every constructor that returns an absolute path joins onto a told `Location`'s `AnchorPath()`, and the `*Rel` constructors and `TreeRunFileRels` return relative paths.
Keep "no constructor resolves a cwd or spawns git".
Also add the tree-relative seed and status paths to the list of kinds in `paths.go`'s header.

### [LOW] Card 4 does not say that `last_at` is parsed only on a folded entry

**Location:** `_lyx/plan/04-shed-span-producers.md`, Intent: "parses `started_at`, `at` and `last_at` as RFC 3339", and "a timestamp in either that fails to parse ... aborts".

**Issue:** `shedengine.HistoryEntry.LastAt` is `json:"last_at,omitempty"` and empty unless the entry folded calls (`internal/shedengine/status.go:70-72`).
Every unfolded entry in a real archived status has no `last_at`, for example `archive/agent-command-wrapping/477ecc096e53`'s history.
The Intent parses `started_at` only when present ("zero when the seed or its field is absent"), but gives `last_at` no such condition.
Read literally, it aborts the report on the first unfolded entry of every run.
The test row "a folded entry (`repeats` > 0) whose `last_at` is not" hints at the condition, but the Intent never states it.
It also leaves open whether a folded entry's `at`, which `End` does not use, is parsed.

**Fix:** State in the Intent that `last_at` is parsed only when `Repeats` > 0.
Say whether `at` is parsed on every entry or only on unfolded ones.
In the first test row, name an unfolded entry with no `last_at` that reads without error.

### [LOW] Card 4's run-id error does not name the tag and the file Decision 9 asks for

**Location:** `_lyx/plan/04-shed-span-producers.md`, Intent: "That call's run-id error is propagated as a fail-loud wiring guard no test reaches".

**Issue:** Decision 9 says: "A run id that fails validation, or a seed or status that fails to decode, aborts the report with an error naming the slug, the tag and the file."
The card wraps the decode and timestamp errors that way, but propagates `TreeRunFileRels`'s `ValidateRunID` error unwrapped.
That error names only the run id (`internal/shedrun/runid.go`).
Calling the branch unreachable is sound: `archivedSlugs` keys by the segment between two slashes, and git ref-name rules exclude `.` and `..` components.
That exempts the branch from a test, not from the message the decision prescribes.

**Fix:** Wrap the run-id error with the slug, the tag's `Name` and the `shed` run directory, in the same shape as the decode errors.
Keep it a fail-loud guard with no test row.

### [LOW] `resolveHub` compares against the cwd's parent, not the worktree's parent

**Location:** `_lyx/plan/05-run-reads-shed-status.md`, Intent: "When the result differs from `filepath.Clean(filepath.Dir(wd))` and `weft` is empty it refuses".

**Issue:** Decision 8's bound is: "when `-hub` names a directory other than the current worktree's parent and `-weft` is empty, the tool refuses".
The weft default it guards is `fabricengine.RecordsWorktree` of `lyxcwd.Resolve(wd)`, which resolves the worktree root from any subdirectory.
`resolveHub` compares against `filepath.Dir(wd)`, the parent of the current directory, which differs from the worktree's parent whenever the tool runs below the worktree root.
In that case a `-hub` naming the correct hub is refused, and the refusal text calls the cwd's parent "the current worktree's hub".
The launchers `cd` to the repository root, and the existing `-hub` default already uses `filepath.Dir(wd)`, so the launcher path is unaffected.
The card still departs from the decision's wording without saying so.

**Fix:** Either compare against the parent of `lyxcwd.Resolve(wd)`'s worktree root, which `run` already resolves for the weft default, or state in the Intent that the tool assumes it runs from the worktree root, as the `-hub` default does, and word the refusal to match.

### [LOW] No test row reaches card 3's missing-commit skip

**Location:** `_lyx/plan/03-archive-tag-dates.md`, Intent: "A line that lacks the `refs/tags/archive/<slug>/<sha>` shape, a commit or a date that parses as RFC 3339 is skipped"; Tests: "the skipped malformed lines (no sha segment, another namespace, a missing date)".

**Issue:** The Intent prescribes three skip branches: bad ref shape, missing commit and unparseable date.
The rows reach the ref-shape branch (no sha segment, another namespace) and the date branch, but no row reaches a line with no commit field.
Under the rubric's reachability rule, a prescribed guard branch needs a named test input that reaches it.

**Fix:** Add a row to the `TestArchivedSlugs` table: a well-shaped ref with an empty object-name field and a valid date, skipped.
Alternatively, drop the missing-commit branch from the Intent if the parse needs no separate check for it.

## Focus verdicts

- Card 1, `TestParseSeed`: not a defect.
  A grep of every `_test.go` under `internal/`, `cmd/` and `tools/` finds no seed written with an unknown driver or an empty params key.
  The only unknown-driver test is `TestValidateDriver`, which calls `ValidateDriver` directly, not the decoder.
  So neither planned row duplicates a test that reaches the behavior through `ReadSeed`.
  The card's list of behaviors left to the existing tests is correct: `started_at` (`TestWriteSeed_ReadSeed_RoundTrip`), the retired `parent` key and an unknown key (`TestSeed_LegacyParentKey`), and an absent driver (`TestReadSeed_AbsentDriverDefaultsToGo`).
  The claim that no test matches `decode seed for run` or `params key must not be empty` holds: the grep finds only `seed.go`.
- Card 4, abort and absence branches: every branch has a named input.
  The rows cover a bad `started_at`, a bad `at`, a bad `last_at` on a folded entry, a tag with `status.json` and no `seed.json`, no tags, a tag without `status.json`, and an empty history.
  The remaining gap, whether `last_at` is parsed only on folded entries, is F2.
- Card 2, "All runs": not a defect.
  The last row uses two runs whose working intervals overlap in wall-clock time, and asserts per-role and idle sums and the span-sum denominator.
  Under a union across runs those overlapping intervals would count once, so the row distinguishes summing from a union.
  The same row asserts that the explanatory paragraph follows the table.
- Card 1, `doc.go` and `paths.go` claims: see F1.
