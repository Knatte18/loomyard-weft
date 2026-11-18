---
verdict: BLOCKING
findings:
  - id: F9
    severity: BLOCKING
    class: scope
    location: _lym/plan/03-drift-verify-loosened-checks.md#Edit
    summary: Card 3 makes Drift.ChangedFiles and Drift.CommitMessages required seams that batten leaves nil, and internal/battencli/wire_seams_test.go intentionallyNil lists every nil Drift seam, so TestWire_EverySeamFilled fails at card 3's gate; card 9 does the same with Landing.PullRequest.ShedStatus. Neither card edits intentionallyNil, they list only the test under Uses.
    fix: In _lym/plan/03-drift-verify-loosened-checks.md add `internal/battencli#intentionallyNil` under **Edit:** and say in the Intent that it gains the entries Drift.ChangedFiles and Drift.CommitMessages with the reason "loom- and darn-only seam, batten has no Drift row", while Drift.ReviewBase needs none because envkit's nilLegal skips it. In _lym/plan/09-pr-review-send-back-row.md add `internal/battencli#intentionallyNil` under **Edit:** and say in the Intent that it gains the entry Landing.PullRequest.ShedStatus with the reason "loom-only seam, batten lands no branch". Keep `internal/battencli#TestWire_EverySeamFilled` under **Uses:** in both cards. A combined lym loom validate-plan --overlay run with both additions reports no blocking finding beyond the baseline.
  - id: F10
    severity: BLOCKING
    class: scope
    location: _lym/plan/07-bouncer-review-inputs.md#Intent
    summary: Card 7 gives captureReview a new optional step parameter, but TestBurlerHalfEntries_EnvFallback and TestBurlerHalfEntries_ColorSegment store captureReview in a table field of the fixed type func(t, name, cfg, env), so the changed signature stops those two tests compiling at card 7's gate; the card lists neither and its glyph carries no re-sign arrow.
    fix: In _lym/plan/07-bouncer-review-inputs.md remove `internal/shedrecipe#captureReview` from **Edit:** and reword the TestBurlerReviewEntry_RubricStencil row so that it builds the row with burlerReviewEntry itself, writes the full generation record naming a pr-findings-1.md path, calls the producer, and reads the rubric from the BurlerHalfRunner fake's GotReviewProfiles, leaving captureReview untouched. The alternative is to keep the changed helper, mark its glyph with the re-sign arrow, and add `internal/shedrecipe#TestBurlerHalfEntries_EnvFallback` and `internal/shedrecipe#TestBurlerHalfEntries_ColorSegment` under **Edit:**. A combined lym loom validate-plan --overlay run with the removal and with the additions reports no blocking finding beyond the baseline.
prior:
  - id: F1
    status: resolved
    answer: Card 3 now lists internal/shedrecipe#driftTestEnv and internal/shedrecipe#TestDriftEntries under Edit, and its Intent says driftTestEnv fills the two seams and TestDriftEntries gains the two nil-seam rows.
  - id: F2
    status: resolved
    answer: Card 10 lists internal/loomrecipe#TestRouting_ProgressAt under Edit and its Intent says the tapestry-tail subtest wants PR-Review's reroute to be Code-Bouncer. The darn and sampler recipe tests do not pin PR-Review's reroute.
  - id: F3
    status: resolved
    answer: Card 7 now renders the rubric at each round through BurlerHalfDeps.Rubric, called from Call before the round's profile is built, and a TestBurlerReviewEntry_RubricStencil row writes the generation record after the build and asserts the reviewers' rubric names the send-back path. The test seam it uses is a new finding, F10.
  - id: F4
    status: resolved
    answer: The disposition is accepted. Fabric excludes the worktree's .scratch link through the repository's info/exclude, so DirtyPaths never counts a file under it. A TestPRFindingsVerb row over the faked dirty seam could not reach git's ignore rules. The dirty-refusal row reaches the message that names .scratch.
  - id: F5
    status: resolved
    answer: Card 7 lists //contracts/specs/shed-recipe-spec.md under Edit and its Intent states the Bouncer passage's new wording.
  - id: F6
    status: resolved
    answer: Card 10 lists //internal/loomrecipe/doc.go under Edit and its Intent rewords the claim that PR-Review's Stuck always halts, in the doc and in the header and PR-Review row comments of all three recipes.
  - id: F7
    status: resolved
    answer: Card 9 lists internal/landingshed#PRReview under Edit and its Intent says the doc comment names the send-back reroute among the ways the row ends.
  - id: F8
    status: resolved
    answer: Card 3's Intent now carries a Bound line naming the trailer's cause as the committer's attestation and the verdict's Loosened checks lines as its bound.
---

### [BLOCKING] Card 3 and card 9 leave batten's intentionallyNil table unedited

**Location:** `_lym/plan/03-drift-verify-loosened-checks.md#Edit`

**Issue:** `internal/battencli/wire_seams_test.go` holds `intentionallyNil`, a table of every `shedrecipe.Env` seam that batten's wiring leaves nil, and `TestWire_EverySeamFilled` fails both for a nil seam the table omits and for a table entry the wiring now fills.
The table lists every `Drift.*` seam and every `Landing.PullRequest.*` seam today.
Card 3 adds the required seams `Drift.ChangedFiles` and `Drift.CommitMessages`, which `driftVerifyEntry` requires, and batten wires neither.
Card 9 adds `PullRequestDeps.ShedStatus`, which `checkWiring` requires, and batten leaves `Landing.PullRequest` unwired.
Each of these paths is a nil seam missing from the table, so the test fails at the card's own gate.
Both cards list `internal/battencli#TestWire_EverySeamFilled` only under **Uses:**, and neither lists the table.

### [BLOCKING] Card 7's captureReview change breaks two tests the card does not list

**Location:** `_lym/plan/07-bouncer-review-inputs.md#Intent`

**Issue:** The card's new `TestBurlerReviewEntry_RubricStencil` row runs through "captureReview's new optional step", so `captureReview` gains a parameter.
`TestBurlerHalfEntries_EnvFallback` and `TestBurlerHalfEntries_ColorSegment` in `internal/shedrecipe/entries_burlerhalves_test.go` store `captureReview` in a table field typed `func(t *testing.T, name string, cfg Config, env Env) (burlerengine.Profile, burlerengine.RunOpts)`, beside `captureFix`.
A function with an extra or variadic parameter is not assignable to that field, so both tests stop compiling at card 7's gate.
The card lists neither test, and its `captureReview` glyph has no re-sign arrow although the Intent changes the parameters.

## Settled focus questions

- Card 7, rubric of a generation the Bouncer starts in the same process: settled.
  `BurlerHalfDeps.Rubric` is read at each `Call` before the round's profile is built, so the note reflects the record the Bouncer wrote earlier in the process, and the named row reaches the rubric the reviewers receive.
  The only defect is F10.
- Cards 8 and 12, where the findings file goes: settled.
  Both name the task worktree's `.scratch/`, which the repository's `info/exclude` covers, so the file never trips `DirtyPaths`; the dirty-refusal message names it and a `TestPRFindingsVerb` row reaches that message.
- Cards 3 and 10, tests and test kits pinning a changed seam or routing value: card 10 is complete; card 3 misses `intentionallyNil`, F9.
- Cards 7, 9 and 10, prose the change falsifies: the shed-recipe spec's Bouncer passage, the `PRReview` doc comment and every "PR-Review's Stuck is always a Halt" site are each listed and reworded; nothing further found.
