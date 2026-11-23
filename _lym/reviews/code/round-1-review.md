---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/orchrole/role-autonomy.md:48
    summary: The orch contract says the PR-Review session mails the parent an ask that ended no-parent, but the autonomous landing block has the session mail the parent only after an undelivered end, so the no-parent half of the line rests on a mail nothing sends.
    fix: "In contracts/orchrole/role-autonomy.md, replace the line \"When the run's PR-Review session mails you an ask that ended no-parent or undelivered, answer that ask the same way:\" with \"When a run's latest PR-Review ask ended no-parent or undelivered, such as one its PR-Review session mails you, answer that ask the same way:\" and keep the following line \"your answer stands at the head the ask named.\" unchanged. Leave the landing stencil as it is."
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md
    summary: "The change description says `AnswerParentAsk` keeps its behavior and delegates with no tag, but commit b696b4cca removed that function and every caller now calls `AnswerParentAskOrUnreached` directly."
    fix: "In _lym/landing/summary.md under \"## What changes\", replace the sentence \"`AnswerParentAsk` keeps its behavior and delegates with no tag, so review rounds and the drift judge are unchanged.\" with \"It replaces `AnswerParentAsk`. Every other caller passes no tag, so review rounds and the drift judge are unchanged.\""
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/shedadapters/bouncer_parent_test.go#TestBouncer_Call_RefusesARecoveredConvergedOverParentFindings
    summary: The redundancy report lists this test as contained by TestBouncer_JudgeCall_AsksTheParentBeforeJudging and TestBouncer_Reentry. It still pins the recovered-verdict refusal's reason text, the verdict's archive and the absence of any judge spawn or commit, none of which those tests assert, and it carries no keep marker.
    fix: "Add the line `//testtiming:keep pins the recovered CONVERGED verdict's refusal over parent findings: its reason, the verdict's archive, and that no judge or commit runs` directly above `func TestBouncer_Call_RefusesARecoveredConvergedOverParentFindings` in internal/shedadapters/bouncer_parent_test.go, replacing nothing."
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/burlerengine/parentreview_test.go:609
    summary: A test doc comment and the failure messages at the call sites the diff rewrote still name the removed AnswerParentAsk.
    fix: "Replace `AnswerParentAsk wrote` with `AnswerParentAskOrUnreached wrote` in the TestReadParentReport doc comment at internal/burlerengine/parentreview_test.go:609. In every t.Fatal, t.Fatalf and t.Errorf message that reads `AnswerParentAsk()`, change it to `AnswerParentAskOrUnreached()`: internal/burlerengine/parentreview_test.go lines 317, 325, 328, 361, 365, 424, 567 and 731; internal/driftengine/judgeparent_test.go lines 185 and 268; internal/driftengine/verify_test.go:2075; internal/loomcli/commitrecords_test.go:350; internal/loomcli/parentreview_test.go:637; internal/shedadapters/bouncer_parent_test.go lines 73 and 155; internal/shedadapters/parentseat_test.go:144. Keep the test function names."
  - id: F5
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/burlerengine/parentreview_test.go#TestAnswerParentAsk
    summary: "No row pins the `unreachedTag != \"\"` guard in AnswerParentAskOrUnreached. The no-tag row's ask carries the PR-Review tag, so the tag-equality check refuses it, and deleting the guard leaves every row passing."
    fix: "Add a row to TestAnswerParentAsk named \"a no-parent ask with no tag is refused when the call names no unreached tag\": asks `[]burlerengine.ParentAsk{{State: burlerengine.ParentAskNoParent}}` with an empty Tag, unreachedTag empty, number 1, now parentAskBase, wantRefusal burlerengine.ParentAnswerEnded, wantStates `[]burlerengine.ParentAskState{burlerengine.ParentAskNoParent}`."
---

### [MEDIUM] The orch contract names a no-parent mail the session never sends

**Location:** `contracts/orchrole/role-autonomy.md:48`

**Issue:** The new line reads "When the run's PR-Review session mails you an ask that ended no-parent or undelivered, answer that ask the same way".
The autonomous landing block (`contracts/stencils/landing/landing-block-pr-review-autonomous.md`, lines 24–31) has the session post the reason and ask number with `lym creel post --parent` only after an undelivered end.
After a no-parent end the session only ends its turn.
The contract therefore describes a no-parent mail that no step sends.
The code accepts an answer to a no-parent ask from the parent orch or the operator's shell, so the fix is to the contract's wording, not to the code.

### [LOW] The change description claims a wrapper the range deleted

**Location:** `_lym/landing/summary.md`

**Issue:** "What changes" says "`AnswerParentAsk` keeps its behavior and delegates with no tag".
Commit b696b4cca ("drop the test-only AnswerParentAsk wrapper") removed `AnswerParentAsk`.
At HEAD no function of that name exists, and every caller, tests included, calls `AnswerParentAskOrUnreached` with an empty tag.
The claimed outcome still holds, since review rounds and the drift judge are unchanged, but the named mechanism does not exist.

### [LOW] A contained bouncer test lacks its keep marker

**Location:** `internal/shedadapters/bouncer_parent_test.go#TestBouncer_Call_RefusesARecoveredConvergedOverParentFindings`

**Issue:** The redundancy report lists the test as contained by `TestBouncer_JudgeCall_AsksTheParentBeforeJudging` and `TestBouncer_Reentry`.
Its assertions cover what those tests do not: a CONVERGED verdict already on disk at Call entry, the reason "holds findings, which rule out CONVERGED", the archived verdict, and no shuttle spawn or commit.
The covering judge test runs the judge live, and the re-entry cases carry no parent ask.
The test stays, and it needs the marker that records what it pins.

### [NIT] Stale references to the removed AnswerParentAsk in tests

**Location:** `internal/burlerengine/parentreview_test.go:609`

**Issue:** The doc comment of `TestReadParentReport` says ReadParentReport returns "what AnswerParentAsk wrote".
The failure messages at each call site the diff rewrote still print `AnswerParentAsk()`.
Both name a function that no longer exists.

### [NIT] The empty-tag guard has no pinning row

**Location:** `internal/burlerengine/parentreview_test.go#TestAnswerParentAsk`

**Issue:** `unreached` requires `unreachedTag != ""` (`internal/burlerengine/parentreview.go:371`).
The row "a no-parent ask under the tag is refused when the call names no unreached tag" uses an ask tagged `[pr-review request]`.
`latest.Tag == unreachedTag` already fails for that ask, so the guard can be deleted without any row failing.
An untagged no-parent ask answered with an empty tag would then be answered.

## Rejected

- code-correctness NIT: a done ask answered through the unreached path keeps its no-parent or undelivered `Reason`. No reader shows `Reason` for a done ask (`parentReviewAwaitVerb` adds it only for not-sent and undelivered). The kept reason records why the ask never reached the parent, which is accurate history, so this is no defect.

## Focus verdicts

- `AnswerParentAskOrUnreached` admits only the run's latest, uncleared no-parent or undelivered ask under the named tag. The check and the write share one `state.UpdateJSON` under the record's lock, so a racing expiry, clear or newer ask leaves one end state. Not a defect.
- A satisfied answer sets the ask done with its `Digest` unchanged, so `ReadPRReviewStanding` reads satisfied at that head and `pr-continue` and the row's `judgeParentAnswer` proceed. A non-PR-Review ask that ended no-parent is refused `ended`, since its tag differs or the caller passes no tag. Not a defect.
- `parentReviewAnswerVerb` passes `landingshed.PRReviewAskLineTag` and keeps `parentAnswerRefusalText` for the cases still refused. The test "a cleared no-parent PR-Review ask is refused" asserts that text. The ended-ask path returns the same output shape as a live answer. The board entry asks for no distinct report, so this is not a defect.
- Scope: the entry's "the ask resolves the orch's current pane" needs no code change. The row's ask and the session's ask share `orchcli.ParentStrand`. Reed's `Status` builds liveness from the panes of every window (`presentPanes(st.Strands, live, windows)`). The issue's no-parent asks came only from the session, which is the #200 signing defect that the merge base 708cdadc5 fixed, where a refused post is now recorded undelivered. The omission is covered and licensed.
- The Root-Cause trailer on 4e863bbab names `AnswerParentAsk` refusing every ended ask. The diff changes exactly that, so the trailer holds.
- In the contract and stencil texts, the stencil's "await first" narrows when to ask again and drops no earlier rule. The contract's no-parent wording is F1.
- Test economy: the contained bouncer test pins what coverage cannot see; see F3. The edits to existing tests in driftengine, landingshed, shedadapters and loomcli only change the call and add `""`, so they loosen no assertion.
