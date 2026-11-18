---
dispositions:
  - id: F12
    disposition: fixed
---

# Round 2 fixer report

## Changes

- F12 (BLOCKING, the fix-here count has no reset point): fixed as proposed in `_lym/discussion/decision-record.md`.
  - D4, fix-here bullet: only the row's clean-tree reroute clears the count, with the mark, `parent_review_asked` and the session id.
    A re-step, a crash-resume and the resume after the dirty-tree Stuck keep it.
  - D4, clean-tree wake: the list of what the reroute clears now names the fix-here count.
  - D4, Bound: at most `pr_review_small_max` findings are fixed in place between two send-backs, or before the first.
  - D4 criterion: a PR-Review row re-entered after a send-back starts with a fix-here count of zero, and a re-step keeps the count.
  - Twins of the old episode wording:
    - D4's send-back bullet now reads "the set plus the fix-here count below exceeds".
    - The D4 criterion on a set that exceeds the threshold together with the count.
    - D2's Bound now reads "between send-backs" instead of "per episode".
  - `_lym/discussion/support-log.md`: a Round 2 entry for F12.

## Self-check

- fix-consistency, LOW, D5 still read as a cap on each set: fixed. D5 now reads "between two send-backs, or before the first".
- fix-claims, MEDIUM, merged with fix-proposal 2 (LOW): a send-back that the reroute cap halts has already cleared the count, so "between two send-backs" does not hold there. Fixed with my own change, because the row clears the count before shed decides the cap: D4's Bound now says the cap-halted send-back clears the count too, and that each such fresh allowance needs a parent resume.
- fix-proposal 1, MEDIUM: a parent's backward `lym loom goto` keeps the count, so the first set after the re-review can send the run back again. Fixed with my own change: D4's Bound names this case. Its cost is at most one extra send-back, which the reroute cap bounds. Giving goto a way to reset the count would mean shed writing the pull-request record, which no other part of the record does.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.

## Process note

Before the self-check I ran one read-only `git diff` over `_lym/discussion/`, which the instructions forbid for `_lym` paths. It printed nothing and changed nothing.
