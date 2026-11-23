---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/loomcli/parentreview.go:330
    summary: "With a satisfied answer standing at the head and a later ask at that head still open, ReadPRReviewStanding now returns satisfied, so parentReviewAskVerb skips its awaiting refusal and records and posts a second ask while the first still awaits its answer; this breaks the verb's documented contract (parentreview.go doc comment and loomcli/doc.go) and leaves the earlier ask open, since the parent's answer to it is then refused as not-latest. No TestParentReviewAskVerb row pins the case."
    fix: "In parentReviewAskVerb, after reading the standing, refuse when it is landingshed.PRReviewSatisfied, writing nothing, with a way forward per PATTERN-refusal-way-forward: a satisfied answer from your parent stands at this head; way forward: run lym loom pr-continue. Keep the PRReviewAwaiting refusal as it is. Add the sentence 'It refuses an ask at a head where a satisfied answer stands.' to the verb's doc comment and the matching clause to the ask verb's line in internal/loomcli/doc.go. Add a TestParentReviewAskVerb row whose ask record holds a satisfied PR-Review ask at the head followed by an open one, asserting the refusal text names lym loom pr-continue and that no new ask is recorded."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/landing/landing-block-pr-review-autonomous.md:23
    summary: "The await-outcome rule still tells the session to ask again whenever await reports timeout, no-parent or not-sent, but await reports only the latest ask, so with a satisfied answer standing at the head and a later ask ended timeout the stencil sends the session to ask again instead of to pr-continue, contradicting the new line 20 and keeping the ask loop the board entry reports."
    fix: "Open the rule at line 23 with the exception: 'When a satisfied answer from your parent already stands at your head, run lym loom pr-continue instead of asking again, whatever await reports of a later ask.' Keep the rest of the paragraph as it is."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/landing/landing-block-pr-review-autonomous.md:20
    summary: "The new line sits between the pr-continue rule and 'It ends the review', so 'It' now reads as the satisfied answer rather than lym loom pr-continue."
    fix: "Move the new line 20 to after line 21 ('It ends the review, and the ordinary landing then merges...'), so 'It' follows the pr-continue sentence again."
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/landingshed/prreviewask.go:90
    summary: The PRReviewAwaiting doc describes an unanswered ask at the head before its deadline, a case the function now reads as satisfied when an earlier satisfied answer stands there.
    fix: "Change the comment to: PRReviewAwaiting is an ask sent at the head that the parent has not answered, before its deadline, with no satisfied answer standing at the head."
---

The diff makes the change item 1 of the board entry asks for: `ReadPRReviewStanding` takes the latest answered, uncleared PR-Review ask at the head first, so a later failed or pending ask no longer hides a satisfied answer.
Item 2 was delivered by 3e8a5eebf before the merge base, and item 3 leaves the run by the parent's added entry; the diff adds no `--head` flag.
The `Root-Cause:` trailer names the defect the diff removes.
With no answered ask at the head, the fallthrough gives the old result in every state.
The new test rows take literal expectations and cover the failed, open, findings-after-satisfied, open-after-findings and cleared cases.
The findings below concern the standing's other readers and the stencil around the new line.

### [MEDIUM] The ask verb posts a second ask while one at the head is still open

**Location:** internal/loomcli/parentreview.go:330

**Issue:** `parentReviewAskVerb` refuses only on `PRReviewAwaiting`.
Where an earlier satisfied answer stands, an open later ask at the head now reads as satisfied, so the verb records and posts another ask.
That contradicts the verb's own "It refuses an ask still awaiting its answer at the head" and leaves the open ask unanswerable, since `AnswerParentAskOrUnreached` answers only the latest ask (`internal/burlerengine/parentreview.go:375`).
Raised by the code-correctness, code-affected-set and code-tests lenses; consolidated here at the affected-set lens's severity, since a documented refusal contract breaks and an orphan ask results.

### [MEDIUM] The stencil's await rule still sends the session to ask again past a standing satisfied answer

**Location:** contracts/stencils/landing/landing-block-pr-review-autonomous.md:23

**Issue:** `parentReviewAwaitVerb` reports the latest ask's state alone.
After a satisfied answer and a later timed-out ask, `await` reports timeout, and line 23 tells the session to ask again, while line 20 says the satisfied answer stands.
The session follows the specific rule and keeps asking, the loop the board entry describes.

### [LOW] "It ends the review" lost its antecedent

**Location:** contracts/stencils/landing/landing-block-pr-review-autonomous.md:20

**Issue:** Line 21's "It" referred to `lym loom pr-continue`; the inserted line makes it read as the satisfied answer.

### [NIT] `PRReviewAwaiting`'s doc omits the satisfied precedence

**Location:** internal/landingshed/prreviewask.go:90

**Issue:** The constant's doc still reads as every unanswered ask at the head before its deadline.

## Rejected

- Code-tests lens, missing `TestParentReviewAskVerb` row: folded into F1, whose fix adds the row.
- Cleared PR-Review asks skipped by the loop letting an older satisfied answer stand: `ClearParentAsk` has only a driftengine caller, so no cleared PR-Review ask arises, and the doc comment states the behavior.
