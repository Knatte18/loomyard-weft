no findings

- Comment edit (F2): comment lines only, no behaviour or caller changes.
- Test with the defect in place: an unwired edge closure would fail in `newShed` at `prfindings.go:199`, the verb would print an error envelope, and the test would stop at its exit check. Not run against reverted code; rests on the review's focus note.
- Test with the fix in place: the verb reaches the edge, the edge returns true, and the verb answers `send_back: true`.
- Tier 1 passed for the package and the test ran.

Further tool calls: none.
