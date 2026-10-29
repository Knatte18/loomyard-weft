```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. Findings list, by function or branch:

- `vocabScanRootFiles` (`internal/lymcwd/enforcement_test.go:559`). Case run: a walk over the tree with no root `CLAUDE.md`. The map is only read as a lookup by relative path, at lines 618 and 621. It is never iterated to open each key as a file, so a missing `CLAUDE.md` raises no error. `README.md` is still scanned. Correct.
- `bareScope` branch (`enforcement_test.go:618`). Case run: `rel == "CLAUDE.md"`. This only matters if some `CLAUDE.md` reappears, for example a nested one under `internal/`. A `.md` under `internal/` is covered by `inInternal`, so that case is unchanged. A new root `CLAUDE.md` would fall outside the scan, which is the intended result of D7. Correct.
- `merriamFixedContext` (`internal/websterengine/merriambase.go:23`). Case run: the constant is still `20600 - 392*12`. The base for a tree without the file counts 59 fewer lines and the fixed term does not change, so the computed base shrinks by the file's lines as the comment says. The comment's arithmetic holds: 59 + 229 + 104 = 392. Correct.
- Comment-only edits (`cmd/lym/crosscompile_test.go:18`, `internal/fabricengine/doc.go:1642`, `tools/mdreflow/reflow.go:2`, `internal/editdirective/doc.go`). These have no runtime effect. The claims they now make match the files they cite: `README.md:220` states the cgo requirement, and `PATTERN-fs-links` exists in the PATTERN list. Correct.
- Stencil and markdown edits (`edit-directive.md`, `darn-template-write.md`, `drift-template-baseline.md`, `orch-template-role.md`). These are static prose. `edit-directive.md` stays marker-free, since the added lines carry no template markers, so the render path is unaffected. Correct.

No findings.

Tool calls:
1. `grep -n "vocabScanRootFiles" -r internal/lymcwd/`: my context had the map's two edits but not how the map is consumed, so I checked that a missing root file cannot error.
