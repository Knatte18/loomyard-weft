# Support log — status-overview-trim

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Shortening the output to its base name: still a long name such as `round-3-review.md` per line, and the board entry asks to drop the paths.
- Keeping the output on review visits only, to show the round: the round heading already shows the round number.
- Timing a gate line by the visit's start: every gate line under one visit would read the same time and say nothing about when that gate ran.

## Review rounds

_No rounds yet._

## Question ledger

- Which paths does the frame draw? Self-pick: only the visit lines' outputs; the state line, wait line and arbiter lines carry none.
- Which time opens a gate line? Self-pick: the first span start of that gate name, in UTC per the operator's decision.
- What does a gate line show with no parsable start? Self-pick: `--:--`, the frame's existing unknown-start mark.
