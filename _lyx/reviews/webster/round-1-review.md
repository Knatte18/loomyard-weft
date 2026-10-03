---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: internal/testkit/envkit/envkit.go:30
    summary: The new nil-legal Env.SegmentBounces seam is missing from envkit's nilLegal set, so go test ./... fails in internal/testkit/envkit (three tests) and internal/battencli (TestWire_EverySeamFilled).
  - id: F2
    severity: MEDIUM
    location: internal/loomcli/wiring.go:666
    summary: gofmt -l reports internal/loomcli/wiring.go (ReviewModel in the Env literal) and internal/loomengine/config.go (Selfreport in Config); both were gofmt-clean at the merge base.
  - id: F3
    severity: MEDIUM
    location: internal/shedadapters/bouncer.go:117
    summary: New and changed Go comments across the diff are fixed-column wrapped mid-sentence, with several sentences per line, against CONSTRAINTS.md's Comment Line-Break Convention.
  - id: F4
    severity: LOW
    location: internal/loomcli/circling.go:136
    summary: The circling status reader treats a missing lock directory as a missing status file, so a run whose committed status.json exists but whose untracked .lyx scratch directory does not is refused with the false no-status-file message.
  - id: F5
    severity: NIT
    location: CONSTRAINTS.md:333
    summary: The two new Review Round Invariant bullets are separated by a blank line, making a loose list where every other CONSTRAINTS.md section uses a tight one.
---

Review range: `git diff $(git merge-base main HEAD)..HEAD` (merge base `5805c58fb`).
`_lyx/shed/review-finding-class/status.json` gives `product.parent` as `main`, and no `_lyx/loom/rework/round-<N>/` exists.

The plan's `verify:` block passes in full (build, unit, integration, smoke vet).
A full `go test ./...` does not, which is F1.

### [BLOCKING] New Env seam breaks the envkit and battencli seam checks

**Location:** `internal/testkit/envkit/envkit.go:30` (the `nilLegal` map); failing tests in `internal/testkit/envkit/envkit_test.go` and `internal/battencli/wire_seams_test.go:63`.

**Issue:** Card 9 adds `shedrecipe.Env.SegmentBounces`, documented as optional ("Nil is the absent value and leaves the Reason without the budget sentence").
`envkit.NilSeams` reports every nil func field not listed in `nilLegal`, so `TestFullEnv_FillsEverySeam`, `TestNilSeams_ReportsExactPaths` and `TestNilSeams_SkipsNilLegalPaths` fail with `SegmentBounces` in their output, and battencli's `TestWire_EverySeamFilled` fails because batten's `wire` leaves the seam nil.
These packages sit outside the plan's `verify:` list, so the plan's own verify passes while the repository's test suite is red.

**Fix:** Add `"SegmentBounces": true` to `nilLegal`, since its nil is a documented default, and re-run `go test ./...`.

### [MEDIUM] gofmt drift in two changed files

**Location:** `internal/loomcli/wiring.go:666`, `internal/loomengine/config.go:361`.

**Issue:** `gofmt -l` over the changed Go files lists both files.
In `wiring.go` the `ReviewModel:` key lost its alignment when the `Slug`/`SegmentBounces` block was inserted above it; in `config.go` the `Selfreport` field lost one column of alignment when `Judge` was added.
Both files are gofmt-clean at the merge base.

**Fix:** Run `gofmt -w` on both files.

### [MEDIUM] New Go comments break the semantic line-break convention

**Location:** `internal/shedadapters/bouncer.go` (the `ArtifactPaths` field doc, `NewBouncer`'s budget rule, `Call`'s clear, circling and pointer rules, the clear branch's inline comment, `settledGeneration`, `retireLegacyVerdict`, `settle`, `settleCircling`, `circlingReason`, the CONTINUE replay and output-list inline comments, the `judgeCall` facts comment); `internal/shedadapters/bouncerfacts.go` (file header, `computeRoundFacts`, `recurringKeys`, `renderRoundFacts`, `writeRoundFacts`); `internal/shedadapters/bouncerfiles.go` (`parseVerdict`, `parseRecordedVerdict`, the `recordedVerdict` addition, `harvestedVerdict`, `readJudgment`); `internal/shedadapters/circling.go` (file header, `RecordCirclingDecision`, `readCirclingDecision`); `internal/shedadapters/round.go` (`factsPath`); `internal/shedadapters/doc.go` (Outcome-mapping opening and the Bouncer bullet); `internal/burlerengine/doc.go` (Finding class section) and `verdict.go` (`Class`); `internal/loomengine/review.go` (`JudgeSettings`, `ResolveJudge`); `internal/shedrecipe/recipe.go` (`JudgeModel`, the `Slug` doc addition) and `entries_bouncer.go` (the Slug/SegmentBounces inline comment).

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line on every new or changed Go comment, with breaks only at sentence or independent-clause boundaries.
These comments are wrapped at a fixed column, breaking mid-phrase (`... that left a verdict and ledger` / `only the legacy reading accepts`, `... (a legacy` / `APPROVED reads as CONVERGED here), this producer` / `has already settled ...`), and several rewritten lines carry two sentences (`Nth blocks the run if it comes back CONTINUE. The seed call's ...`, `... live judge, since the render is deterministic. A write failure degrades ...`).
`burlerengine/doc.go` breaks a single list sentence mid-enumeration across four lines.
The rule outranks the legacy wrapping of the surrounding code.

**Fix:** Rewrite each listed comment one sentence per line, breaking long sentences only at an independent-clause boundary.
Leave untouched legacy lines alone, except where a sentence straddles a changed line, in which case reflow that sentence.

### [LOW] Circling verbs misreport a present status file as absent

**Location:** `internal/loomcli/circling.go:136` (the `readStatus` closure in `circlingState.preRun`).

**Issue:** The closure returns not-found whenever the status lock's directory (`.lyx/shed/<slug>/`, untracked scratch) is missing, without looking at the status file (`_lyx/shed/<slug>/status.json`, committed).
A task worktree whose committed status says `awaiting` but whose scratch directory is gone (a fresh checkout of the branch, a `git clean`) is refused with "the run has no status file", which is false and names the wrong way forward (`lyx loom start` begins the run).

**Fix:** Decide absence from the status file itself: when it does not exist, report not-found; when it exists and the lock directory does not, create the lock directory, then read through `state.ReadJSONStrict`.

### [NIT] Loose list in the Review Round Invariant

**Location:** `CONSTRAINTS.md:333`.

**Issue:** The two added bullets are separated by a blank line, which renders as a loose list; every other bulleted section in CONSTRAINTS.md uses a tight list.

**Fix:** Drop the blank line between the two bullets.

## Focus-directive verdicts

- Bouncer `Call` entry and settle: every on-disk state routes as card 4 and card 8 require.
  CONVERGED and legacy APPROVED clear through `settledGeneration`; legacy BLOCKING replays as Stuck with the ledger pointer; an undecided CIRCLING replays as Awaiting without a spawn; a continue is Stuck; a pending accept settles its record before Approve; a settled accept clears at entry, so a resume after a failed Commit re-seeds without a second Approve; a malformed decision is not a settled one and degrades in `settle`.
  Done is reachable only through `settle` on a round `ResolveRound` found a review for (or the pre-existing `Skip` seam).
  Not a defect.
- `retireLegacyVerdict`: after the archive `judged(n)` reads false; the attach branch falls through to `judgeCall` when the attached judge left neither a strict nor a legacy judgment.
  Not a defect.
- `RecordCirclingDecision` resolves through `round-<N>-review.md`, the name `bouncerEntry` pins for every Bouncer row's `ReportName`; the O_EXCL guard, the settled-only-on-accept rule and the pending-only `settleCirclingAccept` hold.
  Not a defect.
- `filepath.Join(LoomReviewsDir(target), subdir)` equals `resolveUnderRoot(env.RunRoot, run_subdir)` because loom wires `RunRoot` to `LoomReviewsDir`; the awaiting check reads `State` and `CurrentProducer`; every circling refusal has its refusal-spec row, and the parent pre-run skips the group.
  Not a defect, apart from F4.
- `bouncerfacts.go` matches card 5 on gating, recurring keys, reopened marking, the parse-error row and deterministic rendering.
  Not a defect.
- `segmentBounces` takes the status read lock only for the read; `stepLocked` holds the separate run lock during a producer `Call` and takes the status lock only inside its own read and persist, so there is no contention or deadlock.
  The seam's count is `episodeStuckCount` over the pre-append history, and shed blocks the Bouncer's next Stuck when that same count is at or over the budget, so the "continue will block" sentence is true when it appears.
- The judge stencil carries D6 as the decision record writes it, its markers match `judgeValues`, `{{.artifacts}}` is gone and `facts_path` is present.
  Not a defect.
- Comment convention: F3.
- Formatting: F2.
- Per-card mechanical checks: card 11's deleted doc has no remaining link outside `_lyx`; card 10's `resolveReviewTarget` signature change reaches both call sites; card 3's `bouncerEntry` falls back to `Judge*` while `entries_burler.go` still reads `Review*`.
  Not a defect.
