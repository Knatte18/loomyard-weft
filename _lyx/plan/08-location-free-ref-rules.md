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

Tests: `refscanner_test.go` keeps `TestRefScanner_Matches` unchanged as the proof the audit's matches do not move, and adds a table over the location-free rule: the path rule's hits and misses (quoted separator path, bare word outside quotes, string literal inside quotes, heredoc body, slug merely containing the suffix) with the matched text, and the spelling rule's hits and misses (command position, behind a separator, inside a quoted search pattern) with the matched text.

**ImpactSummary:** fabricengine's reference scanner is refactored around a new location-free rule type; the audit's own match results are unchanged.
