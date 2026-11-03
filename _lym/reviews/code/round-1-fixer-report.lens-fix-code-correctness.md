```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code for correctness through the fix-code-correctness lens.

Findings:

1. **LOW** — `internal/seatengine/table_test.go`, new row "a stencil that does not parse" (body `Lead [{{if}}].`). The row asserts only `seat "multi"` and `"seat-test-malformed"`.
   - Both strings also appear in any error from `readStencilSet`, which runs before the `stencil.TopLevelMarkers` call in `CheckMarkers`.
   - If `readStencilSet` already rejects the unparsable body, the row passes with the `TopLevelMarkers` parse-error return deleted. F6 asked for exactly that return to be pinned. I did not confirm which path fires; the tool call below would settle it.
   - The row should assert `seatengine: seat "multi": stencil "seat-test-malformed":`, the exact prefix of the `Errorf` at `prompt.go:135`. It should also use a body that `readStencilSet` accepts but `TopLevelMarkers` rejects.

2. **LOW** — `internal/loomcli/darnwiring.go` `PreflightStencils` closure, F3 wrap.
   - The wrapped error still reaches `preflightshed.stencilsFailure`, which prepends `seat stencils cannot be filled: `.
   - The reason therefore still reads "seat stencils cannot be filled: build the darn writer table: …". A table-build fault, which is config or model-spec, is still labelled as an unfillable stencil.
   - The wrap adds the way forward and names the build, which is what F3's `fix` key asked for. The misnamed prefix is only partly resolved. Splitting it would need a typed error or a separate `Checks` field, which is outside the review's proposal.

No other defects.
- `Table.fills` runs correctly on these inputs:
  - A blank seat value over a filled table value gives `false`.
  - A blank value on an Optional marker gives `true`.
  - A reserved marker gives `true`.
  - An absent seat key falls back to the table value.
  - This matches the `seatValues` merge, where the seat overrides the table. `CheckMarkers` is its only caller.
- The F2 rows (blank table value, blank seat value over a filled table value) fail under the old key-presence `fills`.
- The dedup row would read `topic, topic that` without dedup, so it fails with the defect in place.
- The "two unfilled seats" row fails if `CheckMarkers` collects every seat's markers instead of stopping at the first.
- The F5 table runs the lead and advisor rows correctly: the loop variable is per iteration on Go 1.22 or later, and the shared `table`, `geom` and `names` are read-only.
- The F1 test fails if the closure body is `return nil`.
- The F3 test fails without the wrap.
- The F4 test fails if `preflightEntry` drops `env.PreflightStencils`.
- The gate was green for the four changed packages.

Tool calls: none used. I did not read `readStencilSet`, so finding 1 stays unconfirmed; `prompt.go:127-135` is in context, but the body of `readStencilSet` is not.
