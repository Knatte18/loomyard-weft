```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: no findings.

- **F1 (writable-root sites):** `internal/shuttleengine/codexengine/codexengine.go:117-123` is the only non-test consumer of `Dirs.Records` that reaches a launch line. The `.Records` search over `internal` and `cmd` found no other site that builds a writable root. Resolving it in `Prepare` therefore covers the whole set.
- **F1 (builders):** `ConfigDirs` at `internal/fabricengine/junctionnames.go:131` is the only non-test code that sets `Records:`. No other builder tells the checkout unresolved.
- **F2 (weft stat sites):** `ConfigDirs` was the only place a weft stat fed `Dirs.Records`. Other weft stats such as `unwire.go:103`, `warpclean.go:29` and `origin.go:202` serve other purposes and are outside this finding's premise.
- **F2 (message dependents):** the strings `Bolt checkout not statable` and `weft checkout not statable` occur only in `junctionnames.go`. No test or other code pins the Bolt message, so moving it into the `existingDirOrLog` helper breaks nothing.

Tool calls:
1. grep for `.Records` and `Records:` in non-test Go code: to enumerate every consumer and builder of `Dirs.Records`, which my context lacked.
2. grep for the warning strings, `os.Stat(weft` and `RecordsWorktree(`: to find dependents of the moved Bolt message and any other weft stat site under F2.
