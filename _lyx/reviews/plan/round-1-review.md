---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/07-review-amend-key.md and _lyx/plan/05-shedverbs-pause-label.md (target lists)
    summary: Two cards add a key that existing exact-set pins and key-list docs enumerate, but list none of them, so internal/configreg and internal/loomcli tests fail and three docs go stale
    fix: "Card 7: add `internal/configreg#TestModules_EveryTemplateKeyClassified` and `internal/darnengine/doc.go` to its Edit list; in its Intent, add review_amend_max_bounces to darn's wantLocalKeys list in that test (sorted position, after driver_kind), extend darnengine/doc.go's darn.yaml key list and its 'The two budget keys come from internal/reviewbudget' sentence to the three keys, and extend the key list in internal/loomengine/config.go's file header, which the card's existing loomengine#Config and #LoadConfig members already reach. Card 5: add `internal/loomcli#TestStatusCmd_EnvelopeKeySet` to its Edit list and say in its Tests paragraph that its wantKeys set gains pause_reason. Card 20: add to its docs/overview.md sentence that the darn and loom config key descriptions (around lines 468 and 552) name review_amend_max_bounces. Validated together with lyx loom validate-plan --overlay: no new finding."
    origin: lens:affected-set
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/11-bouncer-reentry.md (Intent, Writes paragraph and the Subject seam paragraph)
    summary: Card 11 writes the generation record unconditionally, at every seed, judge spawn and approveWithoutReview, so a Bouncer with a nil Subject seam, or one answering false, reaches case 3 instead of the case 4 and today's paths it promises
    fix: In card 11's Intent, state that every settle-record and generation-record write (recordSettle, markSettled, the seedCall and approveWithoutReview generation-record writes, judgeCall's JudgeSubject write) runs only when the Subject seam is told and answers true; with a nil seam or a false answer no record is written, so re-entry over a settled verdict takes case 4. Add a TestBouncer_Reentry row asserting that a Bouncer with a nil Subject leaves no generation-record.json after a seed and a judge.
    origin: lens:guard-reach
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/14-loomshed-rework-records.md (Intent, KeepReviewRecords paragraph, and Tests)
    summary: KeepReviewRecords is called only for subdirectories this round's archive moved, but the archive writes record.json before recreateReviewRunDirs runs, so a re-step after a seam failure takes the in-flight branch, moves nothing and carries no records
    fix: "In card 14's Intent, define the carried set independently of this Call's moves: recreateReviewRunDirs calls KeepReviewRecords on every Call, for each ReviewRunSubdirs entry that exists under LatestArchivedReviewsDir(reworkDir), copying into the recreated directory (CarryRecords overwrites, so a repeat is harmless). Add a TestPRRework_FirstRoundMovesGenerationZero row: KeepReviewRecords fails on the first Call, and the re-step, taking the in-flight round, carries the records."
    origin: lens:way-forward
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-settle-records.md (Intent, Subject.Unchanged paragraph)
    summary: Subject.Unchanged admits a moved merge base with a merge-in-only history and an equal patch-id as unchanged, an escape hatch that lets an interacting parent change through unreviewed, and card 10 states no bound for it
    fix: "Add a Bound sentence to card 10's Intent after the Unchanged paragraph: a moved merge base with a merge-in-only history and an equal patch-id reads as unchanged, which lets through, unreviewed by this segment, a parent change that interacts with the task's change without sharing a line with it; the verify tiers bound it, at Publish in loom and at darn's Merge-In, and in loom the Merge-Review segment, unless its merge-overlap skip seam or a merge budget of 0 settles it with no round."
    origin: handler
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/12-bouncer-amendment.md (Intent, startGeneration paragraph)
    summary: Case 3 with a changed subject calls startGeneration, which keeps the reject path only when the reject is later than the settle record's SettledAt, but case 3 has no settle record, so a zero time makes every earlier PR-Gate reject read as later
    fix: In card 12's Intent, name the time case 3 compares a reject against, the judge-spawn time judgeCall records in the generation record beside JudgeSubject (add a JudgeAt field to generationRecord in card 12's wording), and add a TestBouncer_Amendment row in which case 3 with a changed subject and a reject older than the judge spawn names no reject file in the note.
    origin: lens:guard-reach
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/16-loom-goto-guard.md (Intent, settledSegmentGuard refusal text)
    summary: The refusal says a changed subject then starts an amendment generation for every segment, but for Merge-Review the writer is Merge-In and D6 starts a full generation under the perch budget there
    fix: "In card 16's refusal text, make the trailing clause segment-neutral: replace 'and a changed subject then starts an amendment generation' with 'and a changed subject then starts a new generation', and add a TestSettledSegmentGuard row asserting that a Merge-Review review row's refusal names Merge-Bouncer and Merge-In."
    origin: lens:way-forward
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/17-budget-way-forward-texts.md (Intent, batten's told clause)
    summary: Batten's told budget clause names the literal placeholder lyx batten run <slug>, where today's budget reason renders the run's real run-id, so the halt's way forward regresses from a runnable command to a template
    fix: In card 17's Intent, have battenCLI.wire build the clause with the run's resolved run-id, rendering run "lyx batten run <run-id>" with the actual value, as gotoCommand does today, and say so beside the loom clause.
    origin: lens:way-forward
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/plan/16-loom-goto-guard.md (Intent, settledSegmentGuard refusal text)
    summary: The way forward names only the goto onto the Bouncer as re-settling at no review cost, but a goto leaves the run paused, and only the following lyx loom resume runs the Bouncer
    fix: In card 16's refusal text, append to the way forward ', then "lyx loom resume" in the task worktree runs it'.
    origin: lens:way-forward
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/plan/17-budget-way-forward-texts.md (Create line for plan:internal/loomcli#loomBudgetWayForward)
    summary: The Create signature declares the placeholder value x while the Intent gives the constant's real text, so the declared signature disagrees with the card
    fix: "Replace the signature with the Intent's text: const loomBudgetWayForward = \"run \\\"lyx loom resume\\\" in the task worktree: the resume grants the row one more budget; a goto keeps the spent budget and grants none\". Validated with lyx loom validate-plan --overlay: no new finding."
    origin: handler
---

### [BLOCKING] New keys break exact-set pins and key-list docs the cards do not list

**Location:** `_lyx/plan/07-review-amend-key.md`, `_lyx/plan/05-shedverbs-pause-label.md`

**Issue:** The method that collected each card's forced edits missed the tests and docs that pin a key set exactly.

- Card 7's new template key `review_amend_max_bounces` is classified `CategoryNone`, so it joins darn's derived `LocalKeys`.
  `TestModules_EveryTemplateKeyClassified` (`internal/configreg/configreg_test.go:74`) pins that list with `slices.Equal` and lacks the key, so it fails.
- Three docs enumerate the key family and go stale against CLAUDE.md's same-commit doc rule:
  - `internal/darnengine/doc.go`'s darn.yaml key list, and its "The two budget keys come from `internal/reviewbudget`" sentence;
  - `internal/loomengine/config.go`'s file header;
  - `docs/overview.md`'s config key descriptions.
- Card 5 adds `pause_reason` to the status envelope's generic core.
  `TestStatusCmd_EnvelopeKeySet` (`internal/loomcli/status_test.go:90-115`) refuses every key outside its `wantKeys` set, so it fails.

### [MEDIUM] A Subject-less Bouncer still writes the generation record and so never reaches case 4

**Location:** `_lyx/plan/11-bouncer-reentry.md`

**Issue:** The Writes paragraph has `seedCall`, `judgeCall` and `approveWithoutReview` write the generation record with no condition on the `Subject` seam.
The card still promises:
- a nil seam keeps every existing test on today's paths;
- a seam answering false writes no record and clears as case 4.

D5 (`decision-record.md:111`) says such a segment writes neither record.
Once a generation record exists, a re-entry over a settled verdict matches case 3 (no settle record, a generation record, a settled verdict), not case 4.
The card never says which wins.

### [MEDIUM] The PR-Rework record carry is skipped on a re-step after a seam failure

**Location:** `_lyx/plan/14-loomshed-rework-records.md`

**Issue:** `prRework.Call` (`internal/loomshed/prrework.go:158-172`) archives first, and `archive` writes `record.json` last, before `recreateReviewRunDirs` runs.
When `KeepReviewRecords` fails, the re-step finds the round in flight and runs no archive.
Read as "subdirectories this round's archive moved", the seam then has nothing to carry.
The next Plan-Bouncer re-entry falls to case 4, a full re-review, instead of an amendment.
No test row covers the retry.

### [MEDIUM] The merge-in patch-id admission carries no stated bound

**Location:** `_lyx/plan/10-settle-records.md`

**Issue:** `Subject.Unchanged` reads a moved merge base with a merge-in-only history and an equal patch-id as unchanged.
That re-settles the segment with no review over a parent change that may interact with the task's change.
The rubric's attack-surface rule requires the introducing card to state what this lets through and what bounds it.
D5 states it: the verify tiers, and loom's Merge-Review.
Card 10 does not, and card 15's Bound sentence covers only `harnessOwnFiles`.

### [LOW] Case 3's reject comparison has no settle time to compare against

**Location:** `_lyx/plan/12-bouncer-amendment.md`

**Issue:** `startGeneration` keeps the reject path when the reject is later than the settle record's `SettledAt`.
Card 12 also calls it for case 3 with a changed subject, and case 3 has no settle record (`decision-record.md:134`).
The zero time then admits every older reject `LatestRejectReview` finds into an amendment it does not belong to.

### [LOW] The guard's refusal promises an amendment where Merge-Review starts a full generation

**Location:** `_lyx/plan/16-loom-goto-guard.md`

**Issue:** The writer for Merge-Review is `Merge-In`, and D6 (`decision-record.md:165`) gives that segment a full generation under the perch's own budget, never an amendment.
The refusal text says "starts an amendment generation" for every segment.

### [LOW] Batten's budget clause names a placeholder instead of the run

**Location:** `_lyx/plan/17-budget-way-forward-texts.md`

**Issue:** Today's budget reason renders the addressed run through `gotoCommand(s.RunID, …)` (`internal/shedengine/run.go:375`).
Batten's verb takes `[<run-id>]` (`internal/battencli/cli.go:127`).
The card's told clause `lyx batten run <slug>` is a template the reader must fill in.

### [NIT] The refusal's way forward stops at the goto

**Location:** `_lyx/plan/16-loom-goto-guard.md`

**Issue:** `Goto` writes `paused`, so the Bouncer re-settles only after a resume.
The way forward names the goto as if it re-settled by itself.

### [NIT] Placeholder signature on the told constant

**Location:** `_lyx/plan/17-budget-way-forward-texts.md`

**Issue:** The Create line declares `const loomBudgetWayForward = "x"`, while the Intent gives the constant's actual text.

## Rejected

- Verify coverage of loomcli's `tmux` smoke tests and burlerengine's `llm` tests: neither tier is hermetic, so the rule does not require running them. The hub lists no verify tiers, and `go test -tags integration ./...` runs every untagged and integration test of every targeted package, the `!integration` TestMain files included.
- The consume-time guard refusals of cards 6 and 16 are reachable only when a driver reuses an attempt validated without the guard. D2 asks for that arm, so it is not a defect.
- Card 16's interrupted-record refusal row is reachable at unit level only, since `CheckGotoTarget` already refuses a goto past a Bouncer that failed. The guard has no separate branch for it, so the row only pins the shared branch.
- Card 4's guard-error way forward, "re-run goto", cannot progress over an undecodable record. D2 prescribes this way forward, so the plan is not at fault.

Fidelity: D1–D9 and every Constraint are carried by a card. Cards 16, 17, 18 and 19 introduce no scope the decision record does not license.
Card order: no card uses a symbol a later card builds; cards 11–16 consume only what cards 2–6 and 10 introduce.
Edit glyphs: the re-signed persist helpers (card 3) and `reflectFriction` (card 18) carry the arrow. No other member named in cards 2, 6, 11, 12 or 19 changes its signature.
Test economy: cards 3, 5 and 6 extend existing tests, and no card plans a per-symbol test or an assertion on static stencil prose.
