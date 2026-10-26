```yaml
lens: code-pattern-fit
outcome: reported
```

**Rules checked, held or broken (no findings):**

- `PATTERN-sole-parsers`: held. The new block reads `cardID(card)` from the parsed `Plan` only (`fabricref.go:73`). It reads no file and does no parsing of its own.
- `PATTERN-refusal-way-forward`: held. The detail text names the way forward: "rename the card in the Card Index and its file name so its slug no longer ends in the fabric suffix". The new test row asserts `"rename the card"`, so a test reaches it.
- `PATTERN-test-speed`: held. The new rows go into the existing `TestCheckCardFabricReference`, which has `//go:build tier0` and `t.Parallel()`. The fabric suffix is derived (`strings.TrimPrefix(filepath.Base(siblingPath), "task")`), not hard-coded.
- The `FabricReferenceMatcher` seam: held. The check calls only `matcher.MatchPath`, and `planparser` gains no import.
- `PATTERN-fabric-vocabulary`: held. The new message and comments say "fabric" and "fabric suffix". "Weft" appears only in the test's row names, and only through the derived suffix.
- Docs in the same commit (`CLAUDE.md` "Docs land in the same commit"): held. The `card-fabric-reference` entry in `contracts/specs/loom-plan-spec.md`, the `internal/planparser/doc.go` text and the function's doc comment all describe the new hit. The commit does not touch `internal/fabricengine/refscanner.go`, as the decision record's Scope requires.
- Declared PATTERN changes: none are declared, and the commit changes neither `PATTERN.md` nor `pattern/`. Nothing is owed.
- Commit message and departures: held. The single commit has no `Pattern-Departure:` trailer, and no departure from the answer key is visible, so none is needed.
- Comment conventions: held. The new doc-comment lines have one sentence per line and are self-contained. The only symbol they name is the one they document. Line-break rules are the `lyx loom lint-comments` gate's, so I raise nothing on them.
- Markdown line breaks: held. The new spec lines are one sentence each.
- The board entry's own-entry rule: not applicable. The diff touches no board entry.

**Tool calls**
1. `head -5 internal/planparser/fabricref_test.go` and `git diff … --stat -- PATTERN.md pattern/`: I needed the test file's build tag, which the diff hunks did not show. I also needed to confirm the commit has no `PATTERN.md` or `pattern/` change.
