# Card 8 — Expose the fabric-reference rule without a worktree location

**Edit:**
- `internal/fabricengine/refscanner.go`
- `internal/fabricengine/refscanner_test.go`
- `internal/fabricengine/doc.go`

**Intent:**
Make the implementer audit's location-free patterns callable on their own, so the plan check (cards 9 and 10) shares one rule with the audit instead of a copy (D13).

In `refscanner.go`, add an exported type for the location-free rule, built by a constructor that takes no `lyxcwd.Location`, with two methods:
- the path rule: on the text with heredoc bodies stripped, the separator-prefixed name ending in the suffix runs with quotes intact, then the bare word ending in the suffix runs on a copy with quoted spans blanked;
- the spelling rule: `spellingPattern` on the heredoc-stripped, quote-blanked copy.
Each returns the matched text, taken from the copy its pattern ran on and trimmed of the surrounding separator, and whether it matched.
`RefScanner` holds that rule and keeps only its exact-path branch (the worktree's own sibling path) on top, so `Matches` answers exactly as today; the separator-prefixed and bare patterns are compiled in one place.
The suffix still comes from `weftname`, which only this side may import.
`doc.go` names the new type beside `RefScanner`.

Tests: `refscanner_test.go` keeps `TestRefScanner_Matches` unchanged as the proof the audit's matches do not move, since `Matches` runs on this rule and its rows already cover the hits and misses of both rules.
The new table covers only what that test cannot see, the matched text and the two rules being callable apart: a quoted separator path, whose matched text is read from the text with its quotes intact, a bare word and a spelling, whose matched text is read from the quote-blanked copy, a command that only the path rule matches and one that only the spelling rule matches, and a heredoc body that neither returns.

**ImpactSummary:** fabricengine's reference scanner is refactored around a new location-free rule type; the audit's own match results are unchanged.
