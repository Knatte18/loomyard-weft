---
verdict: BLOCKING
findings:
  - id: F15
    severity: BLOCKING
    class: design
    location: internal/shedadapters/bouncer.go#parentFindingsRefusal
    carries: F1
    summary: The parent-findings refusal runs only on judgeCall's fresh harvest (bouncer.go:2067), so the Bouncer's recovery paths in Call (the live-judge attach harvest at bouncer.go:453, entryHarvest at bouncer.go:477 and the recorded-verdict replay at bouncer.go:528) settle a CONVERGED verdict through settleUnlessRefused without reading the parent's report, and the segment is approved over the parent's findings after a driver crash, pause or resume while the judge waits for the parent.
    fix: "In internal/shedadapters/bouncer.go, move the parent-findings check out of judgeCall into settleUnlessRefused, run beside judgmentRefusal after routeDecisions, so every path that settles a judgment applies it. Resolve the round's ask there without sending one: add a parentAsker method in internal/shedadapters/parentseat.go that, for a seat with AskBeforeJudge, returns the number of the ask the round holds under its generation seed (record.RoundAsk(seed, round)) and 0 otherwise; the existing heldAsk returns 0 for such a seat and cannot be reused as is. On those paths settle the ask with asker.settle(round, number, reportPath) before the check, as judgeCall does now, and drop judgeCall's separate check so the check lives in one place. Add a row to TestBouncer_JudgeCall_AsksTheParentBeforeJudging, or a sibling test in bouncer_parent_test.go, where round 1's CONVERGED verdict and ledger are already on disk at Call entry and the round holds a done parent ask whose report holds findings: Call must return Stuck with the refusal reason and archive the verdict file."
  - id: F16
    severity: BLOCKING
    class: design
    location: internal/shedadapters/parentseat.go#judgeParentBlock
    origin: assist:assist
    summary: judgeParentBlock is Bouncer judge instruction text embedded as a Go constant and appended to the judge's prompt in judgeCall, which breaks PATTERN-stencil-ownership's rule that a producer prompt is read at call time through stencilstore and never embedded; the Discussion chair's matching parent-review instructions already live in the stencil contracts/stencils/burler/burler-parent-consolidation.md.
    fix: "Move the text of judgeParentBlock into a new stencil fragment, contracts/stencils/bouncer/bouncer-fragment-parent-review.md, registered as the other bouncer-fragment-*.md files are. In judgeCall in internal/shedadapters/bouncer.go, when parentAsk is not 0, read that fragment through stencilstore for the judge's engine (the engine RenderJudgePrompt is called with) and append the rendered text to the prompt in place of judgeParentBlock; a read failure degrades through the same renderFailureReason path RenderJudgePrompt's error takes. Delete the judgeParentBlock constant, and point TestBouncer_JudgeCall_AsksTheParentBeforeJudging's prompt check at the fragment's heading."
  - id: F17
    severity: MEDIUM
    class: design
    location: internal/driftengine/verify.go#parentAgreement
    carries: F5
    origin: assist:assist
    summary: judgeStartedAt is set only in DriftVerify.judge (verify.go:456); the diff-command path, passCommand, reaches runAgent at verify.go:1071 without setting it, so judgeAskLimit gets the zero time, the parent ask's deadline is not capped, and a silent parent can still run the judge into its 60-minute timeout and a Stuck in command mode.
    fix: "In internal/driftengine/verify.go, move the assignment p.judgeStartedAt = p.now() from judge to the top of runAgent, which both judge and passCommand call before the table, and with it the parent agreement entry, is built. Add a test in internal/driftengine/judgeparent_test.go that builds the judge's table through the production path with an injected clock after the judge run starts (for example through runAgent with a fake Seats, or by asserting the limit the built entry's ask records), and asserts that the recorded ask's deadline is the run's start plus judgeTimeout less judgeParentMargin when judge_parent_timeout_min would run past it."
prior:
  - id: F1
    status: open
    answer: The ask now goes out from the Bouncer's judge call once the round's review and fixer report exist, names that round's own files and is capped by run_timeout_min, but the CONVERGED refusal over parent findings is skipped on the Bouncer's recovery paths; carried as F15.
  - id: F2
    status: resolved
    answer: The ask records a digest of the parsed rulings, and agreementGate sends a new ask when the current judgement's digest differs from the latest ask of the spawn's seed, after a satisfied or a findings answer and after a respawn under the same spawn id.
  - id: F3
    status: resolved
    answer: "Settled by the added entry \"Opt-in parent-ask keys stay outside Guard\"; PATTERN-config-layers carries the written exception and the four keys stay open to run_settings."
  - id: F4
    status: resolved
    answer: "Settled by the added entry \"Drift key names stand\"; judge_ask_parent and judge_parent_timeout_min stay."
  - id: F5
    status: open
    answer: The cap now holds for line mode, but passCommand never sets judgeStartedAt; carried as F17.
  - id: F6
    status: resolved
    answer: TestParentSeats_BuildEachSeatFromItsOwnPerch asserts each seat's tag, subject, AskBeforeJudge, RecordWrittenAt, setting key and wait.
  - id: F7
    status: resolved
    answer: TestDriftVerify_ParentAgreementCarriedWhileAsking asserts the table's gate entries in order through tableWithParent.
  - id: F8
    status: resolved
    answer: The three keep markers are in place with the requested reasons.
  - id: F9
    status: resolved
    answer: TestParentAgreementGate has the pending-post, resolve-error and answer-before-timeout rows.
  - id: F10
    status: resolved
    answer: With the ask moved to the Bouncer, every round names its own review and fixer report, and TestParentAsker_BeforeJudgeAsk_NamesTheRoundsOwnReviewAndFixerReport pins rounds 1 and 2.
  - id: F11
    status: resolved
    answer: TestDriftDeps_BuildTheJudgesParentSeat asserts the anchor-based ask files, the slug and both routing seams.
  - id: F12
    status: resolved
    answer: The not-latest refusal names no decision record and points at what the newer ask names.
  - id: F13
    status: resolved
    answer: The ParentAsk, Round and Seed docs cover both review generations and the drift judge's spawn seeds.
  - id: F14
    status: resolved
    answer: due judges against the seat's own latest tagged ask, reads untagged asks as the discussion seat's, and two asking-rule rows pin it.
---

### [BLOCKING] Recovered Bouncer verdicts skip the parent-findings refusal

**Location:** internal/shedadapters/bouncer.go#parentFindingsRefusal
**Issue:** `parentFindingsRefusal` is called only in `judgeCall`, after its fresh harvest.
`Call` has three other paths that settle a verdict found on disk: the attach to a live judge at the entry probe (bouncer.go:453), the `entryHarvest` replay (bouncer.go:477) and the recorded-verdict replay (bouncer.go:528).
Each calls `settleUnlessRefused`, whose `judgmentRefusal` does not read the parent's report, and none settles the round's ask.
The code-review judge now waits for the parent inside its session for up to `code_review_parent_timeout_min`, so a driver restart or pause while it waits is a plausible path, not a corner case.
On it, a judge that wrote `CONVERGED` despite the parent's findings gets the segment approved, against the judge prompt's own "lym refuses that verdict", orch-role.md's "such a round is not converged", and the doc.go contract.
This is the part of F1's fix ("the parent's findings must reach the Bouncer's judgement") that is left undone.

### [BLOCKING] The Bouncer judge's parent-review instructions are embedded in Go

**Location:** internal/shedadapters/parentseat.go#judgeParentBlock
**Issue:** `judgeParentBlock` holds a whole prompt section: a heading, the await procedure, the rule that a parent finding stands, and the focus-file instruction.
`judgeCall` appends it to the rendered judge stencil.
PATTERN-stencil-ownership: "Reading prompt or spec: at call time via `stencilstore`, never embedded, bar the orch role".
The Bouncer is not the orch role, and no `Pattern-Departure:` trailer in the range names this entry.
The repository already keeps the Discussion chair's equivalent instructions in `contracts/stencils/burler/burler-parent-consolidation.md`, and the Bouncer's own fragments in `contracts/stencils/bouncer/bouncer-fragment-*.md`.

### [MEDIUM] Command-mode drift judge has no cap on the parent wait

**Location:** internal/driftengine/verify.go#parentAgreement
**Issue:** `parentAgreement` passes `judgeAskLimit(p.judgeStartedAt)`, and `judgeAskLimit` treats the zero time as no limit.
Only `judge` (verify.go:456) sets `judgeStartedAt`.
`passCommand`, the `diff: command` path, calls `runAgent` (verify.go:1071) without setting it, so in command mode the parent ask's deadline is `judge_parent_timeout_min` from the ask, uncapped.
A judge that rules 40 minutes into its 60-minute session and asks with the default 30 minutes then times out while waiting, which is the Stuck F5 named.
The mutation report's survivor at verify.go:456 is the same gap from the test side: no test drives the production path that sets the start.

## Focus verdicts

- Code-Review ask timing (parentseat.go, burlerreview.go, bouncer.go): the ask goes out from `judgeCall` after the round's review exists and before the judge spawns, and names that round's own review and fixer report.
  The review producer of such a seat sends and settles nothing.
  The parent's findings reach the judge through the prompt block and the CONVERGED refusal, but only on the fresh-harvest path (F15).
- `agreementGate` and `agreement`: a done answer passes or holds only while the parsed rulings' digest equals the one the latest ask of the spawn's seed recorded.
  A respawn under the same spawn id finds that same ask, so a changed judgement gets a new ask, and an unchanged one keeps its answer.
  Not a defect.
- `parentAgreement`'s cap: capped for line mode, from the start of the judge run of the pass; not set for command mode (F17).
- `due`: a Discussion-Review seat judges against its own latest tagged ask, with untagged asks read as its own, so a later Code-Review or drift-judge ask no longer re-triggers it.
  Not a defect.
- Tests: the new tests each pin their own rows.
  `TestParentAsker_BeforeJudgeAsk_NamesTheRoundsOwnReviewAndFixerReport` overlaps its covers, but its round-2 line naming round 2's own files is asserted nowhere else.
  `TestDriftDeps_BuildTheJudgesParentSeat` asserts the seat's files and slug, which `TestWire_ReviewKeysReachTheEnv` never reads, so it would take a keep marker rather than a fold.
  The always-ask branch of `due` for a seat asking before the judge (`|| a.seat.AskBeforeJudge`) is reached by no row with a prior ask of the same seed.
  None of these test-economy points reaches the BLOCKING bar this incremental round sets for a new finding, so none is raised.

## Rejected

- The drift entry's `awaitParentFindings` and rewrite messages are gate findings text, the same kind of Go-built re-prompt the judgement entry already returns, not a prompt read; not raised under PATTERN-stencil-ownership.
- `_lym/landing/summary.md` still describes the round-1 design: a round-start ask naming the previous round's files, an uncapped drift deadline, and an `AskEachRound` test.
  It is no changed hunk of this incremental round, so it is not raised here; its fixer should still refresh it before landing.
- Mutation survivor judgeparent.go:203 (negate the post's error check): the "a spawn with no ask mails the rulings and holds the judge" row expects an open ask and a held entry, which the mutant breaks, so no missing check is shown.
- Mutation survivor judgeparent.go:106 (the attempts constant): a tuning value; asserting the literal would add no behavior.
