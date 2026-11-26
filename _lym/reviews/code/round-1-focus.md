---
round: 1
focus:
  - 'continuesItem in internal/discussionparser/decisionids.go: does its first-character exclusion list agree with where markdown ends a lazy continuation? Check both directions: block starts it may absorb (thematic break, setext underline, table row, fence, ordered or other list marker) and paragraph text it may cut (a line opening with an issue number, an autolink or an angle-bracket placeholder).'
  - 'SectionBullets and bulletLines: do the two readers now end a bullet at the same places? Trace each loop for fenced blocks, carry-over blocks, whitespace-only lines, indented bullet markers and section boundaries, including what happens to the line after a fence or a block closes.'
  - 'Every consumer of SectionBullets and bulletLines (raddle gates, SplicePart, decision-id coverage): does absorbing unindented lines change what an existing record yields, for example prose or a comment directly under the last bullet of a section, and does a spliced or carried bullet round-trip to the same text that criterion matching compares?'
  - 'The board entry reports the raddle-propose gate refusing the whole bullet joined to one line: after the diff, which text does that gate accept for a two-line criterion, and does any other reader of record bullets in the repository keep the indented-only rule?'
  - 'The doc comments of SectionBullets, bulletLines and continuesItem, and the new sentences in contracts/specs/loom-discussion-spec.md: does each state the ending conditions the code has, no more and no fewer?'
  - '_lym/landing/summary.md: check its claims against the diff, including that a spliced part record reads back the same text, what ends a bullet, and whether the Merge danger door and blast radius cover everything that reads these bullets.'
  - 'Tests in partsplice_test.go and validate_test.go: which excluded openers and ending conditions that the doc comment of continuesItem states have no case that would fail if the exclusion were removed?'
---

Seed focus from the board entry, the commit messages, the change description and the diff.
No round has been judged yet.
