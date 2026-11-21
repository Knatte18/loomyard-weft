---
verdict: BLOCKING
findings:
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/runplan/postbatch.go:301
    summary: "Malformed added-decision headings are silently skipped in internal/discussionparser/append.go, so PostBatchChecks never reaches its new record-path and way-forward error wrapper."
    fix: "In internal/discussionparser/append.go, validate added-decision headings before DroppedCreates obtains spans filtered by isAddedDecisionHeading, using the existing parser-owned heading validation. Return malformed-heading errors through the existing PostBatchChecks wrapper. Extend TestPostBatchChecks_UnreadableDecisionRecordNamesTheWayForward with a record containing an invalid added-decision date and a Drops Create line; assert rejection, the record path, and the parent decision replace way forward."
    origin: lens:code-correctness
    carries: F2
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/burlerengine/roundcheck.go:339
    summary: "The amendment exemption still admits non-card Markdown documents inside the plan directory, including the overview."
    fix: "Restrict namedByAmendment to plan card paths before matching names, using the plan parser's card vocabulary rather than treating every Markdown file in the plan directory as a card. Exclude the overview and amendment log. Add TestCheckReviewRound rows showing that an entry naming a card and mentioning overview or amendments does not exempt those unchanged non-card files."
    origin: lens:code-correctness
    carries: F5
  - id: F9
    severity: MEDIUM
    class: design
    location: internal/shedrecipe/entries_burlerhalves_test.go:159
    summary: "Halt delivery is now tested, but the plan-review amendment-reader wiring remains untested."
    fix: "Extend TestBurlerReviewEntry_ProfileMapping with a completed earlier review round, a pending amendment record at Env.AmendmentPath, and its added decision entry. Drive the review producer and assert that the Plan-Burler-Review profile carries the entry in Round.AmendingTexts while another review row does not. Keep the existing halt-delivery assertions."
    origin: lens:code-tests
    carries: F9
prior:
  - id: F1
    status: resolved
    answer: "Binding now receives filtered card and group declarations; the two-handle test checks that only the retained handle reaches binding."
  - id: F2
    status: open
    answer: "The read-error wrapper is fixed, but malformed headings are excluded before DroppedCreates can return its promised parse error; the added test covers only an unreadable path."
  - id: F3
    status: resolved
    answer: "DroppableCreate distinguishes an unresolved Create from Rename-only findings, and the way-forward table covers both."
  - id: F4
    status: resolved
    answer: "Pending amendments from earlier snapshots are rendered and marked even when AddedTexts is empty."
  - id: F5
    status: open
    answer: "Whole-token matching and the directory restriction fix substring and outside-directory matches, but no check establishes that a Markdown file inside the directory is a card."
  - id: F6
    status: resolved
    answer: "Cluster, assistant-cluster and assistant consolidation preserve origin webster."
  - id: F7
    status: resolved
    answer: "The package documentation permits loom verbs as steps for the parent."
  - id: F8
    status: resolved
    answer: "The parent fold rule explicitly repeats the dropped targets through --drops-create."
  - id: F9
    status: open
    answer: "Spawn, probe, resume and plan-only halt delivery are covered; amendment-reader delivery through recipe wiring is not."
  - id: F10
    status: resolved
    answer: "The selective-drop row asserts that the retained unresolved target still blocks and only the dropped target produces a warning."
  - id: F11
    status: resolved
    answer: "RecordBatch and both Weaver recording paths now have dropped-Create cases."
  - id: F12
    status: resolved
    answer: "The empty-decision row again asserts none at the rendered marker position."
  - id: F13
    status: resolved
    answer: "Only one new Webster finding outside the changed files receives the exemption; the table rejects a second."
  - id: F14
    status: resolved
    answer: "TestParseAddedDecisions carries the requested keep marker."
  - id: F15
    status: resolved
    answer: "The help restores duplicate-title and duplicate-PATTERN restrictions, withdrawal refusal and the sampler's operator-only label."
  - id: F16
    status: resolved
    answer: "Validation derives the plan segment directory from the recipe's Bouncer row, consistent with existing wiring."
---

### [BLOCKING] Malformed decision headings never reach the new refusal

**Location:** internal/runplan/postbatch.go:301

**Issue:** A record with `### Added after Discussion (parent, bad-date): Drop it` produces no heading error.
`DroppedCreates` obtains spans from `addedDecisionSpans`,
which excludes headings rejected by `isAddedDecisionHeading` before the new validation branch runs.
The drop is silently lost, so an unresolved Create reports unfinished work instead of the damaged record.
The evidence is in `internal/discussionparser/append.go:252` and `internal/discussionparser/identity.go:94`.
The test at `internal/runplan/postbatch_test.go:194` covers a directory read failure but no malformed heading.
This retains the malformed-heading portion of F2 and combines the correctness and test lenses' reports.

### [MEDIUM] Amendment matching still includes the overview

**Location:** internal/burlerengine/roundcheck.go:339

**Issue:** An amending entry saying “Provide an overview of recovery in 14-card-slug” exempts `00-overview.md`.
The directory and extension pass; removing `00-` produces the whole token `overview`.
The overview is a separate plan document, as `internal/planparser/parse.go:31` defines.
Checking the directory and token boundaries does not establish card membership.
The correctness and affected-set lenses reported the same remaining portion of F5.

### [MEDIUM] Amendment wiring still lacks a reaching test

**Location:** internal/shedrecipe/entries_burlerhalves_test.go:159

**Issue:** The new profile-mapping row seeds a halt file and asserts only `Profile.HaltReason`.
It never seeds the amendment record or reaches a later round's `Round.AmendingTexts`.
Removing the amendment-reader assignment at `internal/shedrecipe/entries_burler.go:186` escapes these assertions.
The round-context tests supply their own reader directly, so they do not cover that assignment.
This is the amendment-reader portion of F9; its halt-delivery portion is resolved.

## Rejected

- The affected-set lens's file-path drop mismatch is outside this incremental round's changed hunks.
  The exact lookup predates the round-two fixes, as commit `1334c32` shows.
  Its report is preserved in the lens file; it is not admitted as a new finding in this round.
- The halt-file mutation survivors alter logging conditions, not the persisted outcome.
  The Create classification is exercised by the way-forward table through the real classification helper.
  The mutation report alone does not establish another coverage finding.

The focused gate invocation failed before running tests because its shared holder directory was read-only.
No test result is claimed.
The board lookup returned the invalid-JSON error captured in the decision record;
the parent's added scope entry supplied the review yardstick.
