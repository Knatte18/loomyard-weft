```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer applied as proposed, under the fix-proposal lens.

Proposals challenged: F1, F2, F3, F4, F5, F6, F7, F8, F9, F10, F11, F12, F13, F14, F15, F16.

1. **MEDIUM — F6, D2's new sentence "Both callers run D1 and D2 even when `Validate` finds defects and append their findings after `Validate`'s".**
   Two cases go wrong when D1 and D2 run regardless of `Validate`'s result.
   - When the record or the support log is missing, `Validate` returns a `file-missing` finding, not an error (`internal/discussionparser/validate.go:66-79`). D1 and D2 then have no file to read. If they return that read error, it replaces `Validate`'s finding with an infrastructure fault.
   - When the record lacks `## Acceptance criteria`, `Validate` reports `section-missing` (`validate.go:82-88`). D2 then adds a `decision-not-in-acceptance` finding for every id, a cascade of findings caused by one defect.

   The proposal needs a carve-out: skip D1 and D2 on a missing record, and skip D2 when its section is missing. This is the same thing `ValidateDarnDiscussion` already does for the history check.
2. **MEDIUM — F3, D8's new sentence "a member glyph that quarry reports declared only in `_test.go` files, resolved through the plan gate's index".**
   - A test-only card that creates a new test function by member glyph names a glyph quarry cannot resolve yet. That glyph is then classed as non-test code, and the card is refused. That contradicts D8's Bound ("a test-only card ... not checked") and its acceptance line.
   - No `planindex.Index` method reports a member's declaring files. F1's new method is stated to resolve glyphs only, not to report where they are declared. So D8 rests on an index capability the record never decides, which is the same false-premise shape F1 fixed for D4.
3. **MEDIUM — F11, D7's new text "the tier tags are the hub's `gate.yaml` tier keys, which on a Go hub are the build tags the tier commands pass with `-tags`".**
   The proposal states a convention as if it were a fact. The review's own evidence contradicts it: `internal/gateslot/config_test.go` loads `tier1` with `-tags integration`. On a hub configured like that, D7 still demands `//go:build tier1`. The premise of F11 is unresolved, and the Bound names no such hole. Either the tags should be read from the tier commands' `-tags` values, or D7's Bound should state that it assumes keys and tags agree.
4. **LOW — F8, D3's rule "an entry git tracks at the worktree root".**
   D3's Bound sentence still says "a token under a root entry the worktree lacks ... is not tested". The rule now depends on what git tracks, so the Bound should say "an entry git does not track". Separately, the added "and `.git`" exclusion does nothing, because git never tracks `.git`.
5. **LOW — F7, the `decision-id-duplicate` way forward "renumber one of the decisions and the acceptance bullets that name it".**
   Renumbering also falsifies every other `D<n>` reference in the record: the Scope bullets ("D1, D2"), the Constraints ("(D10)") and the Auto-mode assumptions. The way forward should say to update every reference to the renumbered id.

No findings on F1, F2, F4, F5, F9, F10, F12, F13, F14, F15 or F16.

Further tool calls: one. I read `discussionparser.Validate`, to see how it handles a missing record and a missing section for item 1, and listed the loom discussion stencils, to confirm F12's writer stencil is `loom-template-discussion.md`. My context showed neither.
