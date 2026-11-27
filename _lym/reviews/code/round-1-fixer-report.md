---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Tied the listed words (EOF, END, EOT, PY) as proposed. Beyond the proposal, a delimiter word may not open with a digit or a dash, so the arithmetic-shift false denial the finding lists is gone, and the backslash-quoted word is read like the other quoted forms."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "Widened the closing word as proposed, but did not read two heredoc operators on one line: grep -E cannot tie one body and closing line to each operator, and admitting a second operator with untied closings would read the second body as commands and deny text that edits nothing. The shape is named as a pass in the row comment and doc.go, and its row pins it as let through, not denied."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

# Round 1 fixer-report

## Changes

- F1: `internal/shellguard/rows.go` replaces the single closed-heredoc pattern with `closedHeredocPattern()`, one alternative per word of `heredocDelimiterWords` plus one for any other word.
  `heredocBodyLineOtherThan` builds the body line that is not the word.
  `notWordFrom` takes the word-breaker class and the first-position exclusions, so it serves both the `gh` words and the delimiter word.
  The row comment and `doc.go` state the remaining false denials.
  Rows pin the skipped bodies, the unlisted-word false denial and the arithmetic shift.
- F2: `inPlaceEditAfterClosedHeredocPattern` reads all three scopes after a closed heredoc, with single- and double-quoted rows.
  The prefix is rendered once with an alternation of the three scopes, not once per scope as the `fix` key described, which keeps the pattern shorter.
- F3: the closing word of an unlisted delimiter is `[A-Za-z0-9_.-]+`; the unread shapes are named.
- F4: the empty-body row.
- F5: the exception now sits in the `doc.go` sentence that limits a row to one heredoc operator.
- F6: `_lym/landing/summary.md` states the blast radius, the false denial and the remaining passes.
  It is not committed, since `_lym` is the loop's.
- The Claude settings golden is regenerated.

Commits: `38c6e5b06` (F1 to F5), `23ba3306a` from the self-check (F1, F3), and a comment-only commit on top of them.
The last one answers the round's gate findings: it breaks the wrapped and over-long comment lines in `rows.go` and `doc.go` into one sentence per line.

A further comment-only commit answers a second gate finding by shortening one over-long line in `rows.go`.

Gate: after both comment changes, `lym gate test` on `internal/shellguard`, `claudeengine` and `codexengine` passed, and so did `--tier 1` on `internal/shellguard`.
The new rows were not run against the old pattern.
The larger pattern was not timed under `grep`; the review's timing was taken on the previous, shorter one.

## Self-check

- fix-code-correctness, LOW, tab between `<<` and the delimiter word passed (a regression of this round's first commit): fixed as proposed, with a row.
- fix-code-correctness, LOW, the body-line match was mostly unpinned: fixed as proposed, with a row.
- fix-consistency, MEDIUM, the stated shell -c gap was wider than the code's: fixed as proposed in `doc.go` and the row comment.
- fix-consistency, MEDIUM, the summary was stale: fixed as proposed (the pending F6 edit).
- fix-proposal, LOW, the same tab regression: fixed as proposed (merged with the first line).
- fix-affected-set, MEDIUM, F6 not applied: fixed as proposed (merged with the summary line).
- fix-affected-set, LOW, the same tab regression: fixed as proposed (merged with the first line).

Every lens returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
