**LOW: `internal/loomcli/parentseatwiring.go:2`.**
- The reworded header line is about 130 characters. The prose rule caps a line at 119 characters of text.
- The line reads "The closures read the key, the history and the routing on a call, never at wire time, since wire() also runs for status and pause."
- The fix is to split it before "since", which opens an independent clause.

**LOW: `internal/burlerengine/parentseat_test.go:13-15`.**
- The doc comment of `TestParentRulesBlock` still says the part "tags a kept finding `origin: parent` and tells the chair to await the parent".
- The F3 fix deleted the loop that asserted those strings, so the comment now describes an assertion the test no longer makes.
- It should be cut down to what remains: no ask renders the plain step, and an ask adds a blank-line-led part to it.

I found nothing else that disagrees:
- The gate-test doc comment now names the missing-review and missing-report rows, matching the new rows.
- The `"read review file"` row matches the finding text at `parentseat.go:88`.
- The `strings` import in `parentseat_test.go` is still used.
- The `gate[1]` assertion in `review_test.go` matches the entry's `Attempts` and `Gate` fields.

Tool calls: none. My context already held every changed file.
