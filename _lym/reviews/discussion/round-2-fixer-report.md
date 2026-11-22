---
dispositions:
  - id: F22
    disposition: fixed
  - id: F23
    disposition: fixed
  - id: F24
    disposition: fixed
  - id: F25
    disposition: fixed
  - id: F26
    disposition: fixed-differently
    reason: "The fix key's takeover, stopping the waiting run with a pause before the goto, cannot be carried out: `lym tender pause` makes PR-Review hold inside its call (internal/landingshed/prreview.go holdWhileCovered), so the run never reaches the paused state, and Goto refuses a run whose driver holds the run lock (internal/shedengine/goto.go). D4 instead has the operator take a waiting review over by answering its asks from their own unnamed shell, which parentAnswerCallerRefusal admits. The goto clear of the autonomy and session fields is kept for a run halted at PR-Review or later."
  - id: F27
    disposition: fixed
  - id: F28
    disposition: fixed
---

# Round 2 fixer-report

## Changes

All changes are in `_lym/discussion/decision-record.md`, with a Round 2 entry and one rejected alternative added to `_lym/discussion/support-log.md`.

- F22: D4 keeps PR-Review asks in burlerengine's run-wide ask record, the fix key's first option.
  Each ask carries the `[pr-review request]` tag, a seed of its own, and the head it was sent at in `Digest`.
  Its deadline comes from `pr_review_parent_timeout_min`, a new loom key classified in Constraints.
  The answer verb records as it does today.
  The record now says why the latest ask during a review is always a PR-Review ask, and that `pr-continue` and the row read it with `LatestTagged`.
  An expired ask is set timeout and the session asks again.
  New acceptance bullet: under autonomy, with an earlier Code-Review or judge ask in the run, a satisfied answer to the newest PR-Review ask is recorded and `pr-continue` ends the review.
  The rejected option, a PR-Review branch of the answer verb, is logged in the support log.
- F23: D4 gains a bullet: the row's `judge` ends on the continue flag only when the newest PR-Review ask is done satisfied at the gated head.
  Otherwise it clears the flag.
  The Bound states both checks, and a new acceptance bullet covers a commit made after a passing `pr-continue`.
- F24: under autonomy `pr-continue` is refused on a closed pull request, and the row never records `ReviewEnded` there, so it reaches its closed-without-merge halt.
  The orch relays that halt as class 4, and a new acceptance bullet covers it.
  The PATTERN-refusal-way-forward constraint names the new refusal.
- F25: D5 counts class 5 from halt entries, each counting as one halt plus its `repeats`.
  The Bound and the Review D5 acceptance bullet say so too.
- F26 (fixed differently; see the frontmatter):
  - D4 records the autonomy once, when the record holds none, and every later `ensureSession` builds the spec from the recorded value.
  - `lym loom goto` onto PR-Review clears the autonomy and the session fields.
  - D2's exception now reads "once per review".
  - The toggle and goto acceptance bullets match, and a new bullet covers the operator answering from their own shell.
- F27: D3's Bound extends the autonomy stop to not-sent and no-parent judge asks, with the resume re-asking and a persistent cause reaching class 5.
  The D3 acceptance bullet covers all three end states.
- F28: D9 appends the autonomy line beside the presence step, whatever the presence, since `WithPresence` appends nothing outside a slack presence.
  The D9 acceptance bullet adds the terminal-presence case.

## Self-check

All four forks returned a report.

- fix-consistency 1, MEDIUM (D4 says "ask 1" but numbering is run-wide), with fix-claims 2, MEDIUM, merged: fixed. D4 says the review's first ask carries the run's next number.
- fix-consistency 2, LOW (the shell-guard constraint omits `parent-review await`): fixed. The constraint names it.
- fix-consistency 3, LOW (the stencil acceptance bullet omits the await): fixed.
- fix-consistency 4, LOW (the support log's takeover trade-off is stale): fixed. It names the shell answer and the goto of a halted run.
- fix-consistency 5, LOW (the answer verb's help names no PR-Review ask): fixed. D4 has the group and `answer` help name it.
- fix-claims 1, MEDIUM, with fix-way-forward 1, MEDIUM, merged (`lym tender pause` never leaves PR-Review paused, so the goto is refused): fixed differently.
  The pause step is gone, and the takeover of a waiting review is the operator's own answer.
  Goto is named only for a run halted at PR-Review or later.
- fix-claims 3, LOW, with fix-proposal 1, MEDIUM, merged ("Finalize never reopens" is too broad, and a pull request closed after the review ended is still reopened): fixed.
  Finalize takes its closed-without-merge halt on any closed pull request whose record holds autonomy, whatever `ReviewEnded` or merge-at-once says.
  The acceptance bullet covers both closes.
- fix-proposal 2, MEDIUM (`repeats` also counts folded bounces that are not halts): fixed differently from the implied "entries plus repeats" count.
  Only halt entries count: a Stuck with no `routed_to`.
  A bounce names its row in `routed_to` (internal/shedengine/status.go), and `foldsInto` requires equal `RoutedTo`, so a bounce never folds into a halt.
- fix-proposal 3, MEDIUM (the Bound's "only" ignores GitHub approval and merge): fixed. The Bound scopes the check to the continue-flag ending and names GitHub's own approval and merge, and a new acceptance bullet pins the approval.
- fix-way-forward 2, MEDIUM (an ended PR-Review ask's refusal names `lym loom decision add`, which ends no review): fixed. That refusal names the session's next request, and the acceptance bullet says so.
- fix-way-forward 3, MEDIUM (nothing cues the session after the row clears the continue flag): fixed. The row mails the session to ask again, and the acceptance bullet says so.

## Disputed

None.

## Deferred

None.

## Rulings

None.
