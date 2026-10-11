# Fixer report — discussion round 2

## Changes

- F1 (BLOCKING), fixed as proposed.
  Premise confirmed: `doneExit` in `internal/shedadapters/burler.go` archives the round on a gate failure without reverting the target, and D11 itself states the gate-failed edits stay, so a zero-findings re-review would skip every gate over them.
  D4's zero-findings bullet now runs the row's gates first: on a pass it writes the Go-authored report, on a failure it spawns the fixer with the gate's findings, held on the same gates.
  D4's `Bound:` line now limits the skip to a target whose row gates pass, and the "runs no verify gate, since nothing changed" line is replaced by the reason the gates still run.
  The zero-findings criterion is amended and the proposed gate-exhaustion re-review criterion added.
  Support log Q2 updated to match.
- F2 (LOW), fixed as proposed, with the first option.
  D4 gains a bullet: a fixer that died twice, timed out or ended with an unrecognized outcome is a hard error archiving the fixer report only, keeping the accepted review and acceptance record, so a resume re-spawns the fixer.
  This matches the restart criterion's existing killed-fixer case; the restart criterion gains the matching case.
- F3 (LOW), fixed as proposed.
  D7's current-round gate now also refuses any top-level frontmatter key other than `verdict` and `findings`, with prior-round hydration keeping the tolerant decode, and states the two checks together are what D4's skip rests on.
  Round 1 declined this option on the `harmless noise` comment in `internal/burlerengine/verdict.go`; the review's point holds that the skip makes a misnamed findings key no longer harmless, and an APPROVED review with no prose blocks slips past the one-for-one match alone.
  The misspelled-key criterion is extended to the no-prose-block case.
- F4 (LOW), fixed as proposed.
  The budget-of-1 criterion now also states that a fix gate exhausted in round 1 halts the run blocked for a human, with no accept or Done recorded.

## Disputed

None.

## Deferred

None.
