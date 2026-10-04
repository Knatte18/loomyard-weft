---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/pattern/pattern.go:52
    summary: New or changed Go comments are wrapped mid-sentence, against PATTERN-comment-line-breaks
---

### [LOW] Mid-sentence wrapped Go comments in new or changed text

**Location:** `internal/pattern/pattern.go:52-57,79-85`, `internal/pattern/doc.go:1-3,50-57`, `internal/loomengine/prompt.go:18-20`, `internal/burlerengine/geometry.go:23-24`, `internal/hubgeom/hubgeom.go:106-107`, `internal/lyxcwd/enforcement_test.go:848-857`, `internal/lyxcwd/vocabscan_test.go:1-2`.

**Issue:**
`PATTERN-comment-line-breaks` requires one sentence per line in new and changed Go comments, and says fixed-column wrapping is legacy rather than a convention to match.
The listed comments were written or rewritten in this diff, and they break a sentence at a column boundary.
Examples are "...which is not / the repo root...", "...means PATTERN is / inactive...", "...which the anchor path is not in a / subpath-anchored hub." and "...read from / stencilsDir...".

**Fix:**
Re-break each listed comment at sentence ends and independent-clause boundaries.
Do not touch untouched lines.

## Checks run, no finding

- Cards 7-10 and 13: the only remaining `CONSTRAINTS.md` strings are in the two negative tests (`contracts/stencils/citation_enforcement_test.go`, `internal/websterengine/template_test.go`), which assert its absence.
- Cards 14-18: `TestEnforcement_FabricVocabulary` and its fixtures pass.
  The exempt list has entries beyond the plan's table, each with a reason naming an owner declaration, as card 18 allows.
- Cards 2-4: every `pattern.Directive` call site passes the warp worktree root.
  `hubgeom` fills `RepoRoot` with `l.WorktreePath()` in hub mode and the standalone geometries fill it with the target.
- Card 5: the `pattern/PATTERN-*.md` set equals the entry-map's bg rows, and the text states code rules only.
- Card 20: no non-test `Role:` string literal remains, and the scan test passes.
- Card 23: `seedWire` decodes the retired `parent` key and discards it, and no `Seed.Parent` fallback remains.
- Card 1: no `PatternResidue` reference remains.
- Card 12: `manifest/` is gone, and only a string fixture in `tools/mdreflow` mentions it.
- `go build`, `go vet` (default, `integration`, `smoke` tags) and `go test ./...` are clean.
