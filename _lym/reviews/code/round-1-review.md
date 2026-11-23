---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/parentreview.go:408
    summary: No test pins that `parent-review ask` passes the session's own strand as the claim; replacing `strand` with an empty string brings issue #200 back and every test still passes
    fix: "Take the `askPR` closure out of `parentReviewState.preRun` into a named function of `internal/loomcli/parentreview.go` that takes the target location and the strand claim, for example `parentReviewPRAsker(target *lymcwd.Location, strandClaim string) func(pullrequest.Record, string) (burlerengine.ParentAsk, error)`, and have `preRun` call it with `os.Getenv(agentname.StrandNameEnv)`. Then make `TestPRReviewAsker_AskReachesTheParentSignedAsTheCaller`'s session row drive that function with the session's strand (set the claim through `t.Setenv` only if the function reads the env itself, which would drop `t.Parallel` for that test and needs a comment saying why), so the row fails if `preRun` stops passing the session's strand."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Merge danger
    summary: "The two-way merge-danger call does not name the record change the diff makes: a new `undelivered` value in `.lym/parent-review/asks.json`, which an older binary reads; an older Drift-Verify does not halt on it under autonomy, since its `endedUnanswered` lists only timeout, not-sent and no-parent"
    fix: "In `_lym/landing/summary.md`'s `## Merge danger` section, name the change to the ask record: the new `undelivered` state written to `.lym/parent-review/asks.json`. State what an older binary does with it: `ParentAskState.Ended` counts it ended, the PR-Review standing reads ask-again, the review seat does not re-ask, and an older Drift-Verify does not halt on it under autonomy and stores the rulings without the parent. Either keep `two-way` with that justification written beside it or change the call to `one-way`, and widen `Blast radius` to name the drift judge's ask and the review rounds' parent seat beside PR-Review."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/landing/landing-block-pr-review-autonomous.md:26
    summary: The stencil says an undelivered end means creel refused the ask, but every seat ends any failed post undelivered, a daemon or client failure included
    fix: "In `contracts/stencils/landing/landing-block-pr-review-autonomous.md`, change the line `An undelivered end means creel refused the ask, not that your parent is down: its reason names the refusal.` (the word reason in backticks) to `An undelivered end means creel refused or failed to deliver the ask, not that your parent is down: its reason names why.`, keeping reason in backticks. In the `ask` verb's Long text in `internal/loomcli/parentreview.go`, change `An ask creel refuses ends undelivered, with the refusal as its reason.` to `An ask creel refuses or fails to deliver ends undelivered, with the error as its reason.`"
---

### [MEDIUM] The session's strand claim in `parent-review ask` is untested

**Location:** `internal/loomcli/parentreview.go:408`

**Issue:** The fix for issue #200 turns on one argument: `preRun`'s `askPR` closure passes `strand`, read from `LYM_STRAND_NAME`, to `prReviewAsker`.
`TestParentReviewAskVerb` replaces `askPR` with a stub, and the reach report shows only `cmd/lym.TestCLITree_EveryCommand` reaching `preRun`, which checks the command tree.
`TestPRReviewAsker_AskReachesTheParentSignedAsTheCaller` calls `prReviewAsker` directly with the claim, so it pins the asker but not that the verb hands it the session's strand.
Passing `""` there would sign the session's ask as `LOOM` from its pane, the refusal the issue reports, and no test would fail.

### [MEDIUM] Merge danger leaves out the new ask-record state

**Location:** `_lym/landing/summary.md#Merge danger`

**Issue:** The summary calls the door `two-way` with blast radius `parent-review`.
The diff writes a new state value, `undelivered`, into the ask record at `.lym/parent-review/asks.json`, which the run's long-lived driver reads and which an older binary may read after a rebuild or a rollback.
The rubric makes a `two-way` call a finding when the diff changes a record format another binary reads.
The summary's design notes cover only `Ended`.
An older `DriftVerify.endedUnanswered` lists timeout, not-sent and no-parent and nothing else (`internal/driftengine/verify.go:887` before this diff), so under autonomy it does not halt on an undelivered judge ask, and it stores the rulings with no parent answer.
The blast radius also leaves out the drift judge's ask and the review rounds' parent seat, which the diff changes too.

### [LOW] The stencil narrows undelivered to a creel refusal

**Location:** `contracts/stencils/landing/landing-block-pr-review-autonomous.md:26`

**Issue:** `ParentAskUndelivered`'s doc says "creel refused or failed", and every seat ends any `Post` error undelivered, which covers a client-build failure or a daemon error that `noticeError` wraps (`internal/orchcli/notify.go`).
The stencil tells the session the end always means creel refused the ask.
The instruction that follows, sending the reason to the parent, still fits either case, so the session is not misled into a wrong action.
The `ask` verb's Long text narrows the same way.

## Rejected

- Commit 878f0bc14's message says the stencil has the session send the reason "rather than wait for mail", while the stencil has it wait for the next mail after sending (from lens:code-affected-set). The rubric's commit-message checks cover trailers and claims of agreement or improvement, not this kind of wording slip. `_lym/landing/summary.md` states the behaviour correctly, and a fixer could change the message only by rewriting history.
- Mutation survivors at `internal/driftengine/judgeparent.go:207` and `internal/shedadapters/parentseat.go:226` negate `endErr != nil`, which only switches a `logger.Warn` on or off. The undelivered state and its reason are asserted in `TestParentAgreementGate` and `TestBurlerReviewProducer_Call_ParentAskThatDoesNotGoOut`, as the focus asked.
- `prreviewask_integration_test.go` against `TestPRReviewAsker_Ask`: the unit test fakes `Post` and pins state transitions, while the integration test pins who signs the post through creel's real sender resolution. They do not cover the same behaviour, so there is no redundancy finding.
