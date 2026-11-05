```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed `docsLinkIsResearchData` predicate and the new test row in `internal/lymcwd/docslink_test.go`.

I ran the predicate `len(parts) > 5 && parts[0]=="docs" && parts[1]=="research" && slices.Contains(parts[4:len(parts)-1], "data")` on each fixture path:

- `docs/research/topic/x/data/prompts/step.md`: 7 parts, `parts[4:6]` = `[data prompts]`, so true and skipped. This is correct.
- `docs/research/topic/series/x/data/output.md`: 7 parts, `parts[4:6]` = `[x data]`, so true and skipped. This is correct.
- `docs/research/topic/x/report.md`: 5 parts, so false and scanned. This is correct.
- `docs/research/data/x.md`: 4 parts, so false and scanned. This is correct.
- `docs/research/topic/data/x.md`: 5 parts, so false and scanned. The new row expects exactly this break.
- `docs/other/data/x.md`: `parts[1]` is not `research`, so false and scanned. This is correct.

The slice bounds are safe. `len > 5` guarantees `4 <= len-1`, so `parts[4:len-1]` cannot panic.

The new row fails with the defect in place. The old predicate `len > 4` with `parts[3:len-1]` returns true for `docs/research/topic/data/x.md`, because `parts[3:4]` is `[data]`. That file would be skipped, no break would be recorded, and the expected-breaks comparison would fail.

The expected list is in lexical file order: `docs/other/...`, `docs/research/data/...`, `docs/research/topic/data/...`, `docs/research/topic/x/report.md`. This matches the order of the three existing entries. I did not read `docsLinkScan`, so I can't confirm it orders breaks lexically. The existing three-entry expectation relies on the same order.

The only caller is `docslink_test.go:384`, and its signature is unchanged. The `find` run before the fix showed no topic-level `data` folder under `docs/research`; the only `data` folders are at study depth, which stays skipped. So the tighter predicate newly scans no existing file.

The changed doc comment now says a topic-level `data/` is scanned, which matches the code.

No findings.

Tool calls made: none. My context already held every changed span and the earlier `find` output.
