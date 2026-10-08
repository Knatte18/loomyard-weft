# Webster round 1 — fixer report

Nothing fixed.
The working tree is unchanged and nothing was committed.

## Disputed

### F1 — Committed wording departs from the card's exact text

The finding assumes card 11's exact sentence, and the review's proposed `**Fix:**` line, can land in `PATTERN.md`.
They cannot, because the `internal/pattern` scans the plan's `## verify:` runs reject them.

- `internal/pattern/check.go:24` caps an entry line at `MaxEntryLineChars = 320`, and `TestOverview_RealFilePassesCheck` (`internal/pattern/overview_test.go:11`) runs `Check` over the real `PATTERN.md`.
- The original lead-in followed by the card's clause, which is the review's proposed fix, is 409 characters.
  Applying it and running `go test ./internal/pattern/ -run TestOverview_RealFilePassesCheck` fails with: `long-line PATTERN-shuttle-stop (line 66): entry line is 409 characters, over the 320 cap`.
  I reverted that trial edit, and `go test ./internal/pattern/...` passes again.
- No version that keeps the card's wording fits.
  The pre-commit line was already 309 characters, so keeping its lead-in leaves 11 characters for the new clause, and the card's clause is 208 characters on its own.
  Keeping the committed shorter lead-in and adding back the explicit "no other package removes one, except" wording still comes to 341 characters.
- The instruction says a constraint wins when it conflicts with the rest of the prompt.
  The test-enforced entry cap is that kind of constraint, and it rules out using the card's text verbatim.
  The defect is in the plan: card 11 asks for text that cannot fit the entry cap.
  Under the rubric's "Findings raised against the plan itself" rule, a plan defect is not a finding against the diff.

The finding also says the rewrite drops the "no other package removes a shuttle run's strand" prohibition.
It does not.
The committed lead-in already says a shuttle run's strand is stopped from Go outside `internal/shuttleengine` "only through shuttle's stop verb", and removing a strand is stopping it.
The two "are exempt" removals are the only exceptions to that rule, which is what the card's "except" clause says.
The review itself confirms the line names exactly the two exceptions, with the card's reason.

## Deferred

None.
