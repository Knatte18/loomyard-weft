# Card 2 — bouncer-carry-over

**Create:**
- `internal/shedadapters/bouncercarryover.go`

**Edit:**
- `internal/shedadapters/bouncer.go`
- `internal/shedadapters/bouncer_commit_test.go`
- `internal/shedadapters/bouncer_circling_test.go`
- `internal/shedadapters/bouncer_skip_test.go`
- `internal/shedadapters/bouncer_config_test.go`
- `internal/shedadapters/bouncer_fixture_test.go`
- `internal/shedadapters/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/discussionparser/carryover.go`
- `internal/shedadapters/bouncerfiles.go`

**Intent:**
Give the Bouncer the carry-over seam of D2, called on every `Done` that settles a judged round.

`BouncerConfig` gains three told fields:

- `Segment string`, the review segment's name the entry is filed under (for example `Discussion-Review`); `Name` stays a log identity and is never used for this.
- `AnchorPath string`, the absolute anchor the entry's paths are made relative to.
- `CarryOver func(discussionparser.CarryOver) error`, the one seam that writes and commits the entry; nil is the absent value and keeps a Bouncer behaving exactly as before.
  `NewBouncer` refuses a non-nil `CarryOver` with an empty `Segment` or a non-absolute `AnchorPath`.

A new `bouncercarryover.go` builds the entry for a round from on-disk state only:

- every `status: open` entry of the round's ledger at MEDIUM or worse (MEDIUM or BLOCKING), plus every open entry missing its class or its severity, whose missing label is passed as `unlabelled`;
- each finding's rounds are the ledger rounds the key was open in, taken from `ledgerHistories` over ledgers 1..round as the round facts take them, never the judge-written `rounds` list;
- findings sorted by key;
- the review path (`RunDir` joined with `ReportName(round)`) and the fixer-report path (`roundFixerReportPath`), each made relative to `AnchorPath` and slash-separated; a path that does not lie under the anchor is an error.

Call sites in `bouncer.go`:

- `settle`'s `CONVERGED` branch calls the seam with `CarryOverConverged` before `Approve`.
- `settleUnconverged`'s pending-accept path calls it with `CarryOverAccepted` before `settleCirclingAccept`, so a failed write leaves the accept pending and its re-call retries the write.
- The seam is called with an empty `Findings` list when the round has none, so a record that holds the segment's entry loses it.
- `approveWithoutReview` (the skip seam) never calls it: no round ran, so the last reviewed generation's entry still describes the artifact.

A seam error is returned as settle's own error, wrapped as `shedadapters: <name> (bouncer): carry over round N's open findings: <err>`, with a way forward naming the file the error names, its `## Open risks` carry-over marker lines, and `lyx loom start` to re-run the settle; neither `Approve` nor `Commit` runs.
A failed `CONVERGED` settle is then cleared and re-seeded on re-entry as a failed `Approve` is today.
Add the refusal row (or rows, split by class) to the shed section of `contracts/specs/refusal-spec.md` in the same commit (`PATTERN-refusal-way-forward`).

Bound: the seam only writes the decision record's `## Open risks` entry for its own segment; it approves nothing, skips no seam, and a failure blocks the settle instead of passing it.

Tests, each in the file that already covers its path:

- `bouncer_commit_test.go`: a `CONVERGED` settle calls the seam before `Approve` and `Commit` with the expected entry (filtered findings, ledger-derived rounds, unlabelled entries, anchor-relative paths), calls it with no findings when the ledger has none, and a seam error returns an error with the way forward and runs neither `Approve` nor `Commit`.
- `bouncer_circling_test.go`: a pending accept calls the seam with `CarryOverAccepted` before the settle write, and a seam error leaves the decision unsettled so the re-call retries.
- `bouncer_skip_test.go`: a skipped review leaves the seam uncalled.
- `bouncer_config_test.go`: the two new validation rows.
- `bouncer_fixture_test.go`: a recording carry-over fake, if the tests need a shared one.

Update `doc.go`'s Bouncer section to describe the seam, its call sites and its bound.

**ImpactSummary:** Adds an optional seam to BouncerConfig; with it unset every existing Bouncer path behaves as before, and no caller sets it until the next card.
