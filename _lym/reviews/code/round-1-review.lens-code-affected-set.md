```yaml
lens: code-affected-set
outcome: reported
```

Task: review the code-affected-set lens. For each signature, output or behaviour the diff changes, I checked whether everything that calls, reads or asserts it is left correct.

**Per change, the set found and whether it is covered:**

- **`commentlint` rename `endsSemanticBreak` → `IsSemanticBreak(before, after string)`.** It is called at `internal/commentlint/lint.go:349`, `tools/godocreflow/split.go:246,299,328`. No test or other file names the old symbol (grep found none). Covered.
- **`countedText(raw string)` and `isExemptFromLength(text, limit)` signatures.** Their only callers are in `TextReachesLimit` (`internal/commentlint/linelength.go:53-54`). No test calls them directly (grep). Covered.
- **New `TextReachesLimit`, `IsGenerated`, `IsDirective`, `IsProseLine`.**
  - Callers: `lint.go:97,144,186`, `linelength.go:75`, and `tools/godocreflow/reflow.go:33,139,144,155,198`, `split.go:315`.
  - `PATTERN-test-only-code` needs each to be reachable from a `main` package. The `tools/godocreflow` `main` reaches all of them, and `cmd/lym` reaches the lint path. Covered.
- **`internal/commentlint/doc.go`.** The paragraph that said the package copies `tools/godocreflow`'s rules is replaced with a generic statement, so the doc no longer names another package (`PATTERN-code-conventions` "doc.go documents only that package"). The rest of `doc.go` still describes the lint as the diff leaves it, including the directive set and the paragraph skip. Covered.
- **Removed `-max-width` flag and the `wrapped` count message in `tools/godocreflow`.**
  - The usage line, the package doc and the flag parse are all updated (`tools/godocreflow/main.go:20-22,39-44`).
  - A repo-wide grep for `max-width` and `godocreflow` finds only `.gitignore:7-9` (the build output name, unaffected), benchmark tables in `docs/benchmarks/*` (records of past runs, exempt), and a corpus fixture JSONL (a historical transcript, not a contract).
  - `_bolt/_lym/specs` and `_bolt/_lym/stencils` mention neither. Covered.
- **Removed helpers `isGenerated`, `wrapLongLine`, `equalLines`, the local regexes and the `edit.wrapped` field.** No references remain (grep). Their tests (`TestWrapLongLine`, `TestWidthFallbackAppliesToOverwideAtomicLine`) are deleted together with the code. Covered.
- **New `edit.longLines` and `reportLongLines` output.** Only `main.go:73` consumes it, and no doc or stencil describes the tool's stderr format. Covered.
- **The tool now skips the `lym:` and `testtiming:` directives through `commentlint.IsDirective`.** This matches the lint's directive set as `doc.go:28` describes it. Covered.
- **Lint error text that agents read (`internal/loomshed/verifygate.go:271-276`, `commentLintFindings`).** It does not point at godocreflow and its wording does not depend on the change. Not affected.
- **Rule documents.** `pattern/PATTERN-gate-slot.md:64` names `commentlint.Lint`, which the diff does not change. No `PATTERN.md` entry or `pattern/` file names godocreflow or the exported classifiers. The directive declares no PATTERN changes, so "Declared PATTERN changes landed" has nothing to check.
- **Commit messages.** Both commits in range carry `Root-Cause:` trailers that name defects the diff removes (`02ece2ee`: the written breaks lost in `layoutParagraph`; `34565dee`: whole-group skip, rejected clause splits, greedy column wrap). There are no `Pattern-Departure:` or `Loosened-Check:` trailers. The deleted tests go with the code they tested, so nothing was loosened in an existing test.
- **Stencil wording.** The diff changes no stencil. Not applicable.

**Findings:** none.

**Tool calls:**
1. One grep, repo-wide and over `_bolt` specs/stencils, PATTERN and `pattern/`, for the old and changed symbol names, `max-width` and `godocreflow`. The manifest held no search for leftover references to the removed or renamed symbols and flag, nor for `_bolt` specs and stencils pointing at the tool.
