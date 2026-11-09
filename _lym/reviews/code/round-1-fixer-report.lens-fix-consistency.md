```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set (tests and docs) for sibling text or twins left stale.

No findings.

- **Recipe lists:** the batten recipe list and the `Run-Shed` list in `docs/overview.md` (lines 1529 and 1799) now name arras. The twin lists at lines 335 and 1034 already did, so no recipe list is left stale. The `Board task's recipe must be tapestry, darn or empty` sentence is a different constraint and correctly unchanged.
- **Config keys:** the new `split_*` paragraphs agree with `config.go`. `split_max_bounces` is `*int` (null takes the shared value), the split fans are guard keys (closed to the local layer), and an empty `split_fixer_model` takes `raddle`. The discussion/code-only sentences just above them still read as accurate for those two perches.
- **Tests:** the `TestArrasRecipe` header still fits, because the rows are still read and the routing is still pinned. The deleted assertions each have a surviving twin test, as the review named.

Further tool calls:
1. Ran one grep across `*.md`, `*.go` and `*.yaml` for other recipe lists and `code_auto_accept`, to find twin lists my context lacked.
