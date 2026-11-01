---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lym/plan/07-loomshed-part-checks.md:19
    summary: >-
      Card 7 lists contracts/specs/loom-plan-spec.md under Extends although it rewrites a sentence and three check rows already in that spec.
    fix: >-
      In _lym/plan/07-loomshed-part-checks.md move the bullet `contracts/specs/loom-plan-spec.md` from the **Extends:** group to the **Edit:** group (append it after `internal/loomshed#NewLensReviewPlanGate`), leaving `internal/loomshed/doc.go` as the only Extends bullet.
      The card keeps its **ImpactSummary:** line, which an Edit card needs.
      Checked with `lym loom validate-plan --overlay` together with F2 and F3 against a baseline run of `{}`; the combined run reports no blocking finding.
  - id: F2
    severity: LOW
    class: scope
    location: _lym/plan/05-planglyph-part-scope.md:18
    summary: >-
      Card 5 changes two resolve-backed rules for a part (glyph-not-found excludes Part.Paths, caller-uncovered is not raised) but no card carries either rule into loom-plan-spec.md, whose row 35 and glyph-not-found text describe the checks.
    fix: >-
      In _lym/plan/05-planglyph-part-scope.md add a **Extends:** group after **Create:** holding `contracts/specs/loom-plan-spec.md`, and add one sentence to the Intent saying the spec's part-scope note (the one card 4 adds) also states that under a part scope `glyph-not-found` leaves out a target path the part's external exports hold and `caller-uncovered` is left to the joined plan's plan gate.
      It is an addition beside card 4's note, so Extends is the right label; the overlay run named in F1 includes this change and is clean.
  - id: F3
    severity: LOW
    class: scope
    location: _lym/plan/09-seat-writer-kind.md:11
    summary: >-
      Card 9 rewords the advisor-only seat engine comments but misses stepEnder, stepEnder.end and stepEnder.fired in internal/seatengine/engine.go, whose comments still say the first required advisor ends the step.
    fix: >-
      In _lym/plan/09-seat-writer-kind.md add `internal/seatengine#stepEnder`, `internal/seatengine#stepEnder.end` and `internal/seatengine#stepEnder.fired` to the **Edit:** group after `internal/seatengine#seatSpec` (all three resolve with `lym quarry resolve`), and add a sentence to the Intent paragraph that rewords the other advisor-only comments saying their comments name the seat that ends the step, a required advisor or a writer.
      The overlay run named in F1 includes this change and is clean.
  - id: F4
    severity: LOW
    class: scope
    location: _lym/plan/18-validate-parts-verbs.md:41
    summary: >-
      validate-plan's help text says every mode but --require-approved also runs pattern-changes-mismatch, which --part does not; card 18 only says the mode list gains --part.
    fix: >-
      In _lym/plan/18-validate-parts-verbs.md replace the sentence "The help text's mode list gains `--part`, and ..." with one that also rewords the help paragraph "Every mode but --require-approved also runs rename-tool-missing, pattern-changes-mismatch, ..." so it says --part runs the context checks without the decision record and leaves pattern-changes-mismatch and pattern-line-unprescribed to the joined plan's plan gate, and that "five modes" and "The four flags are mutually exclusive" count --part.
  - id: F5
    severity: LOW
    class: decision
    location: _lym/plan/07-loomshed-part-checks.md:47
    summary: >-
      The part check also drops pattern-line-unprescribed, a third difference from the no-flag set that D7 does not list and no Added-after-Discussion entry records.
    fix: >-
      The fixer posts to its parent (lym creel post --parent --expect-reply) asking it to record with `lym loom decision add` an Added-after-Discussion entry that extends D7's second difference with `pattern-line-unprescribed`, whose check runs only with a decision record path (internal/loomshed/patternplan.go), the same input that leaves pattern-changes-mismatch to the joined plan.
      Once the parent answers, card 7's Intent line that clears pc's DecisionRecordPath names that entry as the licence; the card's text otherwise stays.
  - id: F6
    severity: LOW
    class: design
    location: _lym/plan/14-parts-plan-write-table.md:42
    summary: >-
      TestPartsPlanTable asserts each seat's composed prompt holds the text of the four plan-format blocks, which is an assertion on a stencil's static prose; include resolution is already covered by the seat engine's validation and prompt tests.
    fix: >-
      In _lym/plan/14-parts-plan-write-table.md drop "and the text of the four plan-format blocks" from the TestPartsPlanTable clause "each seat's composed prompt from seatengine.Prompts holding its part name, its plan directory ...", keeping the part name, the plan directory and "with no `{{`".
      Include resolution is covered by internal/seatengine#TestSeatPrompts_ValuesPromptsAndSpecs and Table.Validate's validateStencil rows, and the block markers by TestStencils_TopLevelMarkers.
  - id: F7
    severity: LOW
    class: design
    location: _lym/plan/09-seat-writer-kind.md:59
    summary: >-
      Card 9 adds a writer variant of each of the seven ends already in TestEngine_Run_RequiredAdvisorEndsTheStep, though every end reaches the same Seat.endsStep predicate and only two code sites differ.
    fix: >-
      In _lym/plan/09-seat-writer-kind.md replace "it gains a writer variant of each end of a fresh run" with two writer rows, a failed gate (the seatRun.watch site) and a failed start (the Engine.Run site), each asserting EndedStep, Writer and Outputs on the writer's result and nothing queued; the other five ends are covered by the existing advisor rows through the same predicate.
      Card 10's resume variants stay, since Engine.Resume is a third site.
  - id: F8
    severity: LOW
    class: design
    location: _lym/plan/18-validate-parts-verbs.md:34
    summary: >-
      Two of TestValidatePartsCmd's rows and one of TestValidatePlanCmd_Part's repeat behavior other rows already pin; the verb adds only the envelope.
    fix: >-
      In _lym/plan/18-validate-parts-verbs.md drop from TestValidatePartsCmd the row "an absent parts file failing with parts-missing" (partsparser TestValidate in card 3 covers the finding, and the "file with a check finding" row pins the findings envelope) and the row "the same findings as loomshed.NewPartsGate reports for the same file" (both call partsparser.Validate; card 7's TestNewPartsGate covers the gate), keeping the passing row, the finding row and the invalid parts_max row.
      From TestValidatePlanCmd_Part drop "an absent parts file refused with its way forward", since the unknown-part row pins the error envelope of a ValidatePart error and card 7's TestValidatePart covers the absent-file refusal.
  - id: F9
    severity: NIT
    class: consistency
    location: _lym/plan/12-plan-format-block.md:42
    summary: >-
      Card 12 promises Plan-Write's prompt line for line but its Step 3 recipe omits the blank lines the template has after the Step 3 heading (line 73) and before the format block (line 77).
    fix: >-
      In _lym/plan/12-plan-format-block.md change "Step 3 keeps its heading and its three opening lines, then holds" to say Step 3 keeps its heading, the blank line after it, its three opening lines and the blank line after them, then holds `{{template "loom-block-plan-format"}}`.
---

### [MEDIUM] Card 7 extends a spec it rewrites

**Location:** `_lym/plan/07-loomshed-part-checks.md:19-21`

**Issue:** The Intent says the spec's sentence "… in every mode but `--require-approved` …" gains `--part`, the `plan-part` gate and the writer seat's Accept, and that the `test-tier-unstated`, `seam-test-unlisted` and `doc-go-unlisted` rows add `loomshed.ValidatePart`.
I confirmed the sentence at `contracts/specs/loom-plan-spec.md:414`.
Changing text already there is a rewrite, and `Extends` is for adding to a ref without changing anything in it, so the ref belongs under `**Edit:**`.

### [LOW] Card 5's part-scope rules for the resolve checks reach no spec

**Location:** `_lym/plan/05-planglyph-part-scope.md:18-21`

**Issue:** `resolvePass` treats `Part.Paths` as forthcoming and `planGatePass` skips `caller-uncovered` under a part scope.
Card 4 extends the spec with the part scope's effect on `handle-dangling` and `path-missing` only, and card 5 lists `internal/planglyph/doc.go` but no spec.
Row 35 (`caller-uncovered`) and the `glyph-not-found` text of `contracts/specs/loom-plan-spec.md` therefore never say that `--part` behaves differently, and D7 states that difference.
Card 4 cannot carry it, since the behavior arrives in card 5.

### [LOW] Card 9 leaves the step-ender comments stale

**Location:** `_lym/plan/09-seat-writer-kind.md:11-28`

**Issue:** `internal/seatengine/engine.go` documents `stepEnder` as recording "the first required advisor whose end ends the step", and `stepEnder.fired` as reporting "whether a required advisor has ended the step".
After the card a writer ends the step through the same type.
The card enumerates the other advisor-only comments it rewords and names `Engine.Run`, `seatRun.watch` and `SeatResult`, but not these three symbols.

### [LOW] Card 18 leaves validate-plan's help claim about the context checks false for `--part`

**Location:** `_lym/plan/18-validate-parts-verbs.md:41`

**Issue:** `lym loom validate-plan --help` today says "Every mode but --require-approved also runs rename-tool-missing, pattern-changes-mismatch, …" and counts five modes with four exclusive flags.
Card 7 makes `--part` run the context checks without the decision record, so `pattern-changes-mismatch` does not run there.
Card 18 lists `loomCLI.validatePlanCmd` under `Edit` but states only that "the mode list gains `--part`".

### [LOW] The part check drops a third check D7 does not name

**Location:** `_lym/plan/07-loomshed-part-checks.md:47`

**Issue:** D7 lists two checks left to the joined plan, `caller-uncovered` and `pattern-changes-mismatch`.
Card 7 calls `planContextFindings` with `DecisionRecordPath` cleared, and `internal/loomshed/patternplan.go` runs `pattern-line-unprescribed` only with a record path, so that check is dropped too.
The omission is sound, because a part's cards would flag every entry whose fenced line sits in another card, but the decision record carries no entry for it.
Fidelity to the answer key asks for the entry, and a fixer cannot write one, so the fix goes through the parent.

### [LOW] TestPartsPlanTable asserts the blocks' static prose

**Location:** `_lym/plan/14-parts-plan-write-table.md:42`

**Issue:** The clause "the text of the four plan-format blocks" compares prompt text with block text that is a stencil's static prose, which `loom-template-plan.md` itself says is not behavior a test covers.
`TestPartsTable` in card 13 asserts a depth fragment's text, but that is a selection by config and stays.

### [LOW] Card 9's writer variants repeat the required-advisor rows

**Location:** `_lym/plan/09-seat-writer-kind.md:59`

**Issue:** `TestEngine_Run_RequiredAdvisorEndsTheStep` already has one row per end of a required advisor (died, timeout, wait error, failed gate, Accept error, failed start, not found at resume).
Card 9 changes the decision at two call sites, `seatRun.watch` and the start failure in `Engine.Run`, through one predicate, `Seat.endsStep`; the five other ends reach the same `watch` site.
Seven writer rows add no behavior beyond the two sites and the `Writer` and `Outputs` fields.

### [LOW] Card 18's verb tests repeat rows other tests pin

**Location:** `_lym/plan/18-validate-parts-verbs.md:34`

**Issue:** `partsparser.Validate` is the single function behind the verb and the gate, so a verb row for `parts-missing` and one comparing the verb's findings with `NewPartsGate`'s repeat the finding row's envelope assertion and card 3's `TestValidate`.
The same holds for the absent-parts-file row of `TestValidatePlanCmd_Part` against the unknown-part row and card 7's `TestValidatePart`.

### [NIT] Card 12's recipe for Step 3 omits two blank lines

**Location:** `_lym/plan/12-plan-format-block.md:42`

**Issue:** `contracts/stencils/loom/loom-template-plan.md` has blank lines at 73 and 77.
I checked the card's ranges (54–68, 78–110, 113–165, 167–279), its three kept lines (69, 111, 166), the markers each range holds and the absence of any `Step` reference in the moved text against the template; they are exact, and with the two blank lines kept the composed prompt is unchanged line for line.
The card's one-time before-and-after comparison would catch an omission, so this is only a gap in the stated recipe.

## Rejected

- Card 16 and `parts_max`: the gate reads it by key with `env.Setting("tapestry", "parts_max")`, and the four new `Env` fields are seams (table sources and commits), the kind `PlanTable` and `CommitPlan` already are; no config value becomes an `Env` field, so `PATTERN-shed-recipe-registry` holds.
- Card 12 and Plan-Write's prompt order: no moved line changes position, and the three lines that do not fit every including stencil stay in the template at their original places.
- Cards 7 and 14 and the parts file: no card lets the chair edit it. Card 14's chair stencil says it never changes it, and `part-export-missing` is licensed by the parent's third added decision.
- Card 14's chair stencil Step 4 does direct the chair to rewrite each carried decision's `Applies to` line to the joined card numbers, matched by slug.
- Card 6, 7 and 19 refusals: each has a named input and a named test through its own path. Card 6's unmet-export refusal is a backstop behind card 7's `part-export-missing`, but `planparser.JoinParts` is an exported write path and card 6's test calls it directly. Card 19's expected chair name matches the name the seat engine gives it (`internal/seatengine/prompt.go` forms seat names with `agentname.Format`).
- Card 18's `--part` refusal by the recipe's rows: the Constraints name `--part` among the verbs that refuse by rows. The branch is unreachable from an armed run on today's recipes, which the card says, and its test builds the command without arming it.
- No `Plan-Write` prompt, shared-decision format or entry-gate grammar problem: `tapestry.plan_split != true` is a form `shedbuild.parseEntryCheck` accepts, and `## Shared Decisions` is raw body text, so card 6's `### Part <name>` sections parse.

## Process note

While listing the plan directory I printed the first lines of `_lym/plan/lens-review.md`, a file inside the target directory that summarizes an earlier round's lens review, before my review was saved.
No finding above is taken from it; each rests on the cards, the template and the repository code I read.
