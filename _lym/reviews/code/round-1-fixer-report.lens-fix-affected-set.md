```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

All three set premises checked out:
- **F5:** the other mentions of the replayer (`doc.go:37`, `replayer.go:1`, `replayer.go:15`) already say it claims `HEDDLE.md` or are neutral. `cmd/lym/main.go:161` was the only stale comment.
- **F2:** the third `addDerived` caller, `Show` at `render.go:361`, returns text and no warning by design. `Render` and `RenderOverview` are the only callers that surface `DerivedWarning`, and both now do.
- **F6:** `PATTERN-heddle-instructions.md` has no other heading missing the blank line before it.

Calls made, each for something the context lacked:
1. grep for other comments describing the heddle replayer, to check F5 for sibling comments the review didn't name.
2. Read `render.go:345-375`, to see the `addDerived` caller in `Show` that the context didn't hold.
3. grep of headings with one line before each in the pattern file, to check F6 for other lost blank lines.
