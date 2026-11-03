```yaml
lens: code-affected-set
outcome: reported
```

**Per-change list (lens code-affected-set)**

1. **`path-missing` `Detail` string now ends in an optional remedy suffix.**
   - Consumers found: `internal/planparser/validate_test.go` (the new subtest checks the remedy substring; `TestValidate_PathMissing` line ~1622 checks only that the detail contains a file name). `internal/planparser/extends_test.go:86` and `internal/websterengine/planfixer_test.go` use `Check`/`Card`/`Ref` or `"path-missing/NN: file"` digests, not `Detail`. No stencil or spec matches the old sentence. Covered, nothing left wrong.
2. **`checkRef` inlined and `report` closure removed.**
   - Set found: `Check`, `Card` and `Ref` are built identically at the single remaining site. The `checkPathMissing` doc comment and the header list in `validate.go` still hold (the check ID is unchanged). Covered.
3. **New helper `rootRelativeGlyphRemedy`.**
   - Callers: only `checkRef`. It is unexported and the `doc.go` and header comment lists name no helpers, so nothing else needs an entry. Covered.
4. **`loom-plan-spec.md` line 161 (worktree-relative self glyph under `root:`).**
   - Set found: consistent with line 160, line 149 and the line 414 example. No other spec or stencil restates the `root:` and glyph rule. The grep found no `contracts/stencils` text on it. Covered.
5. **`loom-plan-checks-spec.md` line 64.**
   - Set found: the `path-missing` bullet only. The `loom-parts-spec.md` line 112 and `loom-plan-checks-spec.md` line 113 mentions are unaffected. The PATTERN entry sole-parsers (ref shapes) is untouched, since the diff adds no classification or ref-shape decision. The diff changes no `PATTERN.md` or `pattern/` file and no declared PATTERN change exists, so nothing is missing there. There is no `Pattern-Departure:` trailer and none is needed.

**Findings**

- **LOW, `consistency`, `contracts/specs/loom-plan-checks-spec.md:64`.** The spec says the remedy applies when the `root:`-joined path "is present". The code (`validate.go:1370`) uses `satisfied`, which is also true for a Create target, a Rename destination or a part path, not only a path on disk. The decision record says "on disk, or as a Create or Rename destination". The spec line should match the code, for example "present on disk or created by a card". Evidence: `validate.go:1311-1313`, `1370`.
- No other affected-set finding. The decision record and landing summary claims about consumers hold. The commit message's "refuse with `path-missing` and `glyph-not-found`" is a description of existing behavior, and the diff does not touch `glyph-not-found`.

**Tool calls**
1. grep over `contracts` and `internal` markdown for other statements of the `root:`/glyph rule: my context had only the specs I had already read, not stencils or other docs.
