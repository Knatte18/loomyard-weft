```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1:** `pairs` is read and written only in `resolveChildPair` (`overview.go:228`, `:235`) and deleted at `:290` and `:307`. The new delete covers every non-Run-Shed read. The other slug-keyed caches (`seeds`, `childRecipes`, `childCaches`) hold content that is write-once or mtime-keyed, not the anchor, so the anchor move does not make them stale. The new test pins the premise itself: a moved marker is re-read in the next Run-Shed spell.
- **F2:** `runsWindowTitle` has one writer, `cli.go:197`, and one reader, `runs.go:66`. `commandOf` is called only from `Command()` and the new tier1 test, so no other caller of `Command()` is affected. The new test covers both the prime and the beam, the two values `beam` can take at the call site.

Calls made:
1. `grep` over `internal` for `r.pairs` and `.pairs[`: to list every use of the F1 cache, which my context lacked outside `overview.go`.
2. `grep` over `internal` for `orchCLI{` and `runsWindowTitle`: to list every writer and reader of the F2 field and every constructor of the CLI receiver.
