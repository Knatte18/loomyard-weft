---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

## Changes

- F1: `contracts/orchrole/role-autonomy.md` now reads "When a run's latest PR-Review ask ended no-parent or undelivered, such as one its PR-Review session mails you, answer that ask the same way:". Commit c2815c8de.
- F2: `_lym/landing/summary.md` now says `AnswerParentAskOrUnreached` replaces `AnswerParentAsk` and every other caller passes no tag. The file sits under `_lym`, so it is edited on disk and not committed.
- F3: added the `//testtiming:keep` marker to `TestBouncer_Call_RefusesARecoveredConvergedOverParentFindings`. `lym loom fmt` placed it after a bare `//` line in the doc comment. Commit 0535eeecb.
- F4: renamed `AnswerParentAsk()` to `AnswerParentAskOrUnreached()` in every failure message the review lists, and fixed the `TestReadParentReport` doc comment. Test function names are unchanged. Commit 0535eeecb.
- F5: added the row "a no-parent ask with no tag is refused when the call names no unreached tag" to `TestAnswerParentAsk`. Commit 0535eeecb.

`lym gate test` over burlerengine, shedadapters, driftengine and loomcli passed with 0 failures.

F3, F4 and F5 share files, so they form one commit; F1 touches no shared file and has its own.

## Self-check

All four lens forks returned a report and none found anything.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
