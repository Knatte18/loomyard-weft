---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/02-burler-halves-engine.md (Fix half, the fixTable paragraph)
    summary: fixTable's field list omits the review segment color reviewTable carries, so the fixer seat spawns uncolored, breaking PATTERN-spawn-color for the fix half
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-split-burler-rows.md and _lyx/plan/10-retire-burler-round.md (Edit lists)
    summary: cards 4 and 10 list a bare test-file path while their Intent deletes named existing tests and fakes in that file, instead of listing those members as Delete glyphs
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/06-row-name-prose.md (Edit list and Intent)
    summary: card 6 claims no comment will name a retired Burler row, yet test comments in loomcli, landingshed, discussionparser and loomrecipe that name the old rows are in no card
---

### [BLOCKING] The fix half's seat table carries no segment color

**Location:** `_lyx/plan/02-burler-halves-engine.md`, the paragraph beginning "`fixTable` is the role prefix `fixRolePrefix`".

**Issue:**
The decision record's constraints require, for `PATTERN-spawn-color`, that each half's "color segment ride the launch spec".
Card 2 lists the review table's fields as "the role prefix `reviewRolePrefix`, the review segment color, `ReviewGateEntry(p.ReviewPath)` …", but lists `fixTable` as "the role prefix `fixRolePrefix`, `opts.Gate` with every entry wrapped by `repairReportBeforeGate` as the table gate, `opts.Round`, `opts.Timeout`, not interactive, and one chair seat", with no segment.
Today's `roundTable` (`internal/burlerengine/engine.go:122`) sets `Segment: segmentcolor.Review`, and `seatengine.Table.Segment` (`internal/seatengine/table.go:91`) has no default and no validation, so an implementation that follows the card spawns the fixer uncolored.
`fix_test.go`'s planned shape assertions (one chair, role, wrapped gate) do not pin the segment, so nothing would catch it.
Per this round's constraints section, a constraint violation is BLOCKING.

**Fix:**
In card 2's `fixTable` sentence, add "the review segment color (`segmentcolor.Review`)" after the role prefix, matching `reviewTable`.
Add the segment to `fix_test.go`'s fix-table shape assertion, and to `review_test.go`'s review-table shape assertion.

### [MEDIUM] Deleted tests and fakes listed as a bare file path

**Location:** `_lyx/plan/04-split-burler-rows.md` (`internal/loomcli/smoke_attachprobe_test.go` under `**Edit:**`) and `_lyx/plan/10-retire-burler-round.md` (`internal/shedadapters/burler_test.go` under `**Edit:**`).

**Issue:**
The rubric's member-glyph rule flags an `Edit` card that lists only a test-file path while its Intent changes a named existing test, fake or helper.
The plan template allows a plain test-file path only "for a test file the card adds tests to without changing an existing member".
Card 4's Intent deletes `probingBurlerRunner`, its three methods and `TestSmokeBurlerRound_AttachesToALiveRoundInsteadOfRespawning` from `smoke_attachprobe_test.go`, but lists only the file path.
Card 10's Intent says `burler_test.go` "loses its `BurlerProducer` tests and its compile-time assertion over the old fake", but lists only the file path.
The file holds `TestNewBurlerProducer_Validation` and sixteen `TestBurlerProducer_*` functions, from `TestBurlerProducer_RoundScan` through `TestBurlerProducer_Call_PreAttemptArchiveFailureHonoursCancellation`.
Card 10 does list as Delete glyphs the parallel `TestBurlerProducer_*` functions in `attachprobe_test.go`, so `burler_test.go` departs from the card's own convention.
`delete-target-gone` and the derived dependency edges see none of these deletions.

**Fix:**
On card 4, replace the `internal/loomcli/smoke_attachprobe_test.go` Edit path with `**Delete:**` glyphs `internal/loomcli#probingBurlerRunner`, `internal/loomcli#probingBurlerRunner.Run`, `internal/loomcli#probingBurlerRunner.ProbeRound`, `internal/loomcli#probingBurlerRunner.Resume` and `internal/loomcli#TestSmokeBurlerRound_AttachesToALiveRoundInsteadOfRespawning`.
On card 10, replace the `internal/shedadapters/burler_test.go` Edit path with `**Delete:**` glyphs for `internal/shedadapters#TestNewBurlerProducer_Validation` and each `internal/shedadapters#TestBurlerProducer_*` function in that file.
Add any kept helper the card changes there, such as `internal/shedadapters#simpleBurlerProfile`, as an `**Edit:**` glyph.
Drop each file path, since `redundant-file-target` refuses a file listed beside a member of it.

### [LOW] Card 6 misses test comments that name retired rows

**Location:** `_lyx/plan/06-row-name-prose.md`, the Intent's "this card brings every comment, help text, spec and background file that names a retired row up to date, so none points at a row that no longer exists".

**Issue:**
No card covers these comments, which name a retired row and are outside every card's targets:
`internal/loomcli/parity_test.go:292` ("Plan-Burler's own gate", in `planParityCase`'s doc),
`internal/loomcli/validate_test.go:260` ("Plan-Burler's own gate", in `glyphPlanFixture`'s doc),
`internal/discussionparser/validate_test.go:246` ("Discussion-Burler gates"),
`internal/landingshed/publish_verify_test.go:366` ("the Webster-Burler goto", whose way forward card 4 moves to `Webster-Burler-Review`),
and `internal/loomrecipe/overlay_seam_guard_test.go:7`, the file header ("the shipped Discussion-Burler row").
After the plan lands, each points at a row that no longer exists, which contradicts the card's own claim.

**Fix:**
Add to card 6's `**Edit:**` the glyphs `internal/loomcli#planParityCase`, `internal/loomcli#glyphPlanFixture`, `internal/discussionparser#TestValidate_DecisionRecordPathIsDirectory` and `internal/landingshed#TestPublishVerify_FailureRecord`, plus the path `internal/loomrecipe/overlay_seam_guard_test.go` for its file header, since card 6 lists no member of that file.
Name each in the Intent's covered list, renaming the old row to the fix row for gate sentences and to the review row for the goto.

## Focus verdicts

- Fidelity: D1 and D5 are carried by card 4, D2 by cards 2 and 3, D3 and D4 by cards 2 and 3, D6 and D9 by card 2, D7 by card 1, D8 by card 10, D10 by cards 8 and 9, and D11 by card 7.
  D9's `PATTERN-review-round` amendment (card 2), D5's fixer-report repair (card 2's `fixTable` wraps every entry with `repairReportBeforeGate`) and D6's lean fixer prompt (card 2's `composeFixPrompt`, with card 10 dropping the fixer part of `pattern-directive-review-fix`) each have a card.
  The decision record's constraints are each carried, except the spawn-color gap in F1.
  Shared Decision `changed-review-recovery` is not a defect.
  D4's own text contradicts itself, since Shed resumes a failed run at the failing row and that row is now the fix row.
  The plan keeps D4's hard error and archive set and names the goto as the way forward.
  `gotoAdmitted` (`internal/shedengine/goto.go`) admits the review row from a failed fix row, since the review row comes earlier in the list, so that way forward works.
- Card order: no card between 4 and 10 edits code naming a symbol card 10 deletes, apart from prose in doc files.
  Cards 7 and 9 depend only on earlier cards.
- Verify coverage: `go test ./...` and `go test -tags integration ./...` run every targeted package, and the section follows the template's order.
- Reachability: card 3's missing-acceptance refusal names its inputs and a `burlerfix_test.go` test.
  The changed-review error is reached by `fix_test.go` and card 3's tests.
  Card 8's negative-key refusal has a test row per key, and `LoadConfig` is the first check on that input.
- Attack surface: card 7's Intent states D11's bounds one for one, and card 2 states D4's empty-review bound.
- Test economy and granularity: card 3's row tests carry over `BurlerProducer`'s cases, which card 10 deletes, so the duplication is transitional under `additive-then-retire`.
  `TestBouncer_AutoAccept` rows with the flag true cover bounds no existing test covers.
  The private helpers on cards 2 and 3 sit on their parent symbol's card already.
- Edit-card glyphs: re-sign arrows are present where signatures change (`splitFrontmatter`, `renderCirclingDecision`, `Routing`, `reviewSegments`, `applyReviewBudget`, `reviewBudget`, `resultEnvelope`, `newLoomBurler`, `newEscalationShed`); F2 records the bare test-file paths.
