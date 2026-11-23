```yaml
lens: fix-consistency
outcome: reported
```

**Findings**

1. **LOW**: `internal/loomshed/doc.go:112-114` and `internal/loomshed/recordwrite.go:136` are not wrong, but they sit beside a reworded `errorEnvelopeBodyReason`.
   - Both still say only that the entry body is Stuck. Neither is false, and no sibling carries the old single-route way forward.
   - Evidence: `grep` for `batten cancel <slug>` near Record-Write or body text found no stale copy of the old reason outside the test and the constant.
   - I found no change needed here; it is listed only because I checked it.

2. **LOW**: The F2 fence skip applies to `--body-file` too, through `internal/boardcli/bodyfile.go:39`, and its sibling text did not change.
   - The `applyBodyFile` doc comment (`bodyfile.go:20-21`) says it refuses "a file holding a CLI error envelope line" and does not say that a fenced line is exempt.
   - The new help lines in `cli.go` (upsert flag, merge `--body-file`) repeat that unqualified wording.
   - A reader would conclude a fenced envelope is refused, which is now false.
   - Evidence: `ErrorEnvelopeLine`'s doc comment now says "A line inside a fenced code block is never matched".
   - Suggested edit: add "outside a fenced code block" to those three places.

3. **LOW**: The `errorEnvelopeBodyReason` doc comment and the test cover the beam route, but the guard's reach has no test.
   - `recordwrite_test.go` pins only the stuck case with an unfenced envelope under a heading.
   - It has no row that a body quoting an envelope in a fence reaches Done, so the Record-Write side of F2 is untested.
   - `output_test.go` pins the fence behaviour in the helper, so this is a gap in coverage rather than an inconsistency.

No BLOCKING or MEDIUM findings.

**Checks that agreed**
- The reason text now names `lym batten run <slug>`. That matches `internal/battencli/arm.go:868`, which uses it for a fresh start after a cancel.
- `--remove` matches `cancelcmd.go:60`, and `lym loom resume` matches the missing-entry reason in `recordwrite.go:178`.
- The updated test `want` list matches the new constant text exactly, with no stray `<slug>` placeholder mismatch.

**Tool calls**
- `grep` for `ErrorEnvelopeLine` and "error envelope" across `.go`, `.md` and `.yaml` files: to find twins and docs my context lacked.
- `sed` on `internal/loomshed/doc.go:108-120` and a `grep` for stale `batten cancel <slug>` text: to read the doc.go sibling, which my context lacked.
- `Read` of `internal/boardcli/bodyfile.go:14-43`: to read the `--body-file` guard's doc, which my context lacked.
