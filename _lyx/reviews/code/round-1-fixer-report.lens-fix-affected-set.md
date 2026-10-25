```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the sets behind F1 (stale "last" claims about the dobby start) and F3 (inline joins of the adopt scratch path).

No findings.

- F1: a search of `.go` and `.md` files under `internal`, `docs`, `PATTERN.md` and `pattern` for "last" next to "dobby" in adopt-related lines found no other stale claim.
- F3: `lyxdirs.AdoptDirName` is now used only at its definition and in `fabricengine.AdoptScratchDir` (`internal/fabricengine/junctionnames.go:174`). No other non-test code joins that path inline. Test files were excluded from this search.
- F2 is a single coverage gap rather than a set, so I ran no search for it.

Tool calls:
1. `grep` for "last" next to "dobby" in `.go` and `.md` files, to find other stale "last" claims. My context only held `doc.go`.
2. `grep` for `AdoptDirName` and `"adopt"` in non-test `.go` files, to find other inline joins of the path. My context only held `deps.go`.
