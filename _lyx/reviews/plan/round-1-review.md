---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/01-head-reconcile-helpers.md
    summary: reconcileReportHead's walk rule checks equality with reportHead only at a non-merge commit, so a reportHead that is itself a merge commit is walked past and refused; no test covers it
  - id: F2
    severity: LOW
    location: _lyx/plan/06-merge-in-warns-mid-run.md
    summary: errConflictsWithRecord's new warnings input has no specified shape, and its untagged callers in envelope_test.go are not among card 6's targets
  - id: F3
    severity: LOW
    location: _lyx/plan/03-recover-batch-tolerates-merge.md
    summary: RecoverResult.Warnings doc comment still says substrate-cleanup failures only once it carries the moved-HEAD notice; card 3 lists RecoverResult under Uses, not Edit
  - id: F4
    severity: LOW
    location: _lyx/plan/02-record-batch-tolerates-merge.md
    summary: record-batch's behaviour changes in card 2's commit but the package-doc description of its head rule lands only in card 3's commit
  - id: F5
    severity: LOW
    location: _lyx/plan/05-run-in-flight-predicate.md
    summary: D5's unreadable outcome.yaml acceptance row has no test case; the table covers only malformed content
  - id: F6
    severity: NIT
    location: _lyx/plan/06-merge-in-warns-mid-run.md
    summary: the recipe-blind constraint names the whole word loom but omits the other recipe name, batten
  - id: F7
    severity: NIT
    location: _lyx/plan/06-merge-in-warns-mid-run.md
    summary: test seeds webster state under Dir(h.PrimeWorktree()) while the verb reads Dir(loc().AnchorPath()); the two coincide only at the dot anchor
  - id: F8
    severity: NIT
    location: _lyx/plan/04-parent-merge-wedge-scenario.md
    summary: the RecordDeps the card builds names State, Plan, Batches and Geom but not the other seams RecordBatch reads (RefMatcher, Sleeper, OutcomePath, SummaryPath)
---

### [MEDIUM] reconcileReportHead walks past a reportHead that is a merge commit

**Location:** `_lyx/plan/01-head-reconcile-helpers.md`, Intent paragraph on `reconcileReportHead` and its test list.

**Issue:** The Intent says each walked commit "must have two or more parents" and that the walk "stops at the first commit with fewer than two parents ... which is refused, unless that commit is `reportHead` itself".
Read literally, the only equality check against `reportHead` happens at a commit with fewer than two parents.
A `reportHead` that is itself a merge commit passes the parent-count test, gets collected as a walked merge, and the walk continues past it until it hits a non-merge commit, which is then refused.
That case is realistic: card 4's own scenario ends with batch 2's `StartSHA` at the parent merge commit, and a batch-2 fork that commits nothing reports `head_sha` equal to that `StartSHA`.
A second parent merge-in before batch 2's `record-batch` then wedges the run exactly as today, which D1 ("walking HEAD's first-parent chain reaches the report's `head_sha`") forbids.
The test list has no case with a merge-commit `reportHead`, so the misreading would pass the card's own tests.

**Fix:** State the walk order explicitly: at each step compare the current commit with `reportHead` first (equal accepts), and only then require two or more parents before stepping to the first parent.
Add a test: a first merge commit as the report head, a second `--no-ff` merge on top, accepted with the warning naming only the second merge's SHA.

### [LOW] errConflictsWithRecord's signature change is unspecified and its test callers are untargeted

**Location:** `_lyx/plan/06-merge-in-warns-mid-run.md`, Intent paragraph on `setMergeExit`/`errConflictsWithRecord`.

**Issue:** The card says `errConflictsWithRecord` "gains the matching optional field" without saying how.
Go has no optional parameters: a positional parameter breaks the three untagged call sites in `internal/fabriccli/envelope_test.go` (`TestErrConflictsWithRecord_ConflictEnvelopeShape` and `TestErrConflictsWithRecord_ReservedKeysAreAlwaysTheHelperOwnValues`), which the verify section compiles, and that file is not among the card's targets.
The card also leaves implicit that the `warnings` key must be absent when the slice is empty, which the "envelopes are unchanged" claim depends on.

**Fix:** Pin the shape: `errConflictsWithRecord` takes a `warnings []string` parameter and adds a `warnings` key only when it is non-empty; `envelope_test.go`'s existing calls pass `nil`.
Add `internal/fabriccli/envelope_test.go` to the card's Edit list.

### [LOW] RecoverResult's Warnings doc comment goes stale

**Location:** `_lyx/plan/03-recover-batch-tolerates-merge.md`, Uses list and Intent.

**Issue:** `RecoverResult`'s doc comment (`internal/websterengine/recoverbatch.go`) describes `Warnings` as "non-fatal substrate-cleanup failures".
Card 3 routes the moved-HEAD notice into that field, but lists `RecoverResult` only under Uses and never mentions its comment.
Card 2 updates the matching `RecordResult.Warnings` comment for record-batch, so the two verbs end up documented unevenly.

**Fix:** Move `internal/websterengine#RecoverResult` to card 3's Edit list and have the Intent update its comment to list the moved-HEAD notice alongside the substrate-cleanup warnings.

### [LOW] Package-doc description of record-batch's head rule lands a commit late

**Location:** `_lyx/plan/02-record-batch-tolerates-merge.md` and `_lyx/plan/03-recover-batch-tolerates-merge.md`.

**Issue:** The decision record's Scope names "websterengine's package doc (the record-batch head cross-check)" under "Docs in the same commit", and its Constraints repeat that docs named in Scope land in the same commit.
Card 2 is the commit where record-batch starts accepting merge-only movement, recording at the report's head and refusing mid-merge, yet `doc.go` is edited only by card 3.
Card 2's commit therefore ships the behaviour change with a package doc that does not describe it.

**Fix:** Card 2 edits `internal/websterengine/doc.go` to add the merge-only head rule for record-batch to the fork-return contract section (acceptance, recording at the report's `head_sha`, the warning, the refusals, and the bisect limit).
Card 3 then extends that paragraph to say recover-batch follows the same rule.

### [LOW] RunInFlight's unreadable-outcome case is untested

**Location:** `_lyx/plan/05-run-in-flight-predicate.md`, test list.

**Issue:** D5's acceptance row is "unreadable/malformed `outcome.yaml` → error".
The table covers two malformed forms (an unknown outcome value, unparseable YAML) but no unreadable file, so the read-error path through `parseOutcome`'s `os.ReadFile` goes untested.

**Fix:** Add a row where `outcome.yaml` exists but cannot be read, portably by creating a directory at `OutcomePath(Dir(anchor))`, and expect an error.

### [NIT] Recipe-blind constraint omits batten

**Location:** `_lyx/plan/06-merge-in-warns-mid-run.md`, the ly-drive paragraph.

**Issue:** `cmd/lyx/lydriveskill_test.go` forbids every name in `shedrun.RecipeNames()` as a whole word, which is both `loom` and `batten`, plus the three literals.
The card lists only `loom` as a forbidden whole word, and the triggering run is named `batten-hub-ready`, a string an implementer could plausibly cite.

**Fix:** Say the rule may not contain either recipe name (`loom`, `batten`) as a whole word, nor the literals `lyx loom`, `lyx batten` or `.lyx/`.

### [NIT] Test seeds state at PrimeWorktree rather than the anchor path the verb reads

**Location:** `_lyx/plan/06-merge-in-warns-mid-run.md`, test paragraph.

**Issue:** The verb reads `websterengine.Dir(loc().AnchorPath())`, where `AnchorPath` is `WorktreePath()` joined with `AnchorRel`.
The test seeds under `websterengine.Dir(h.PrimeWorktree())`, which is `WorktreePath()` alone.
They coincide at `hubforge.NewHub`'s default `.` anchor, which is what the merge-in tests use, so the test works; at a `backend` anchor the verb would read a directory the test never seeded.
hubforge's own `PrimeWeft`/`WeftBase` docs warn about this exact divergence.

**Fix:** Seed under `websterengine.Dir(h.Location.AnchorPath())`, the same value the verb reads.

### [NIT] Card 4's RecordDeps wiring is under-specified

**Location:** `_lyx/plan/04-parent-merge-wedge-scenario.md`, Intent.

**Issue:** The card builds `RecordDeps` "from the same `State`, `Plan`, `Batches` and `Geom`" plus a `recordFakeEngine`.
`RecordBatch` also reads `RefMatcher` (consulted by `CheckParent`/`CheckFork`, documented as never nil), `Sleeper`, `OutcomePath` and `SummaryPath`, and `newRecordFixture` sets all four.

**Fix:** Name the remaining seams: `RefMatcher: websterengine.NeverMatches{}`, a `recordFakeSleeper`, and `OutcomePath`/`SummaryPath` in a temp contract dir, as `newRecordFixture` does.

## Focus verdicts with no finding

- Card 2, D2 fidelity: passing `report.HeadSHA` as `postBatchInputs.HeadSHA` covers both the delta (`planglyph.Delta(worktree, StartSHA, HeadSHA)`) and `DetectDrift`'s triggering SHA, since `postBatchChecks` threads `in.HeadSHA` into both; the card says so explicitly.
  Placing `refuseMidMerge` before the dirty check is not a defect: with no merge in progress it returns nil and the equal-HEAD path is unchanged, and a mid-merge refusal at equal HEAD is the new refusal D4 licenses.
- Card 3, refusal point: both refusals sit after the `DigestStatusRunning` return and before the strand/run-dir removal switch, and the digest is persisted only later by `PersistRecoveryTerminal` in the CLI, so the batch stays non-terminal and retryable, as D4 requires.
- Card 4 granularity: a test-only card pinning the begin → record → begin scenario across two verbs is separately reviewable and matches its own acceptance row; its `Uses` of `RecordBatch` derives the edge to card 2.
  `newBeginFixture` and `recordFakeEngine` are both in package `websterengine_test` under the integration tag and can share one `State`; `BeginBatch` sets `Kind: "fork"` and `SessionID`, which `RecordBatch` needs.
- Card 5 against `parseOutcome`: `parseOutcome` rejects unreadable files, undecodable YAML (including unknown fields), unknown outcome values and a `stuck` outcome with no reason; the card's error cases and the `stuck with a reason` row are consistent with that.
  `archiveStaleOutcome` runs at run start before Master spawns, so a previous run's `done` outcome cannot mask a new run.
- Card 6 anchor: `merge-in` runs in `h.PrimeWorktree()` in the existing helpers, `<worktree>/_lyx` is the weft junction, and at the `.` anchor `AnchorPath()` equals `PrimeWorktree()`, so the seeded state is visible (see F7 for the anchor-general form).
- Card 6 help: `cmd/lyx/helptree_test.go` checks subcommand presence and `Short`, so an added `Long` paragraph does not affect it.
