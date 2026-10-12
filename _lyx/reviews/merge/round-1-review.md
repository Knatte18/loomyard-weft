---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: tools/tokencount/count.go:613
    summary: archivedSlugs re-parses the archive tag shape by hand with the literal refs/tags/archive/ while the merged-in parent now exports fabricengine.ArchiveTagPrefix and ArchiveTagSlug for the same shape
    fix: In archivedSlugs, strip "refs/tags/" from fields[0] and take the slug from fabricengine.ArchiveTagSlug, dropping the local CutPrefix/Cut and the empty-slug/empty-sha checks, or leave it as is and accept the duplicate; the shapes agree today, so this is optional.
---

### [NIT] tokencount keeps its own copy of the archive tag shape

**Location:** `tools/tokencount/count.go:613`

**Issue:** The parent's `ArchiveTagSlug` (`internal/fabricengine/archive.go:24`) accepts a short name with the `archive/` prefix, a non-empty slug and a non-empty final segment.
`archivedSlugs` accepts the same shape: it cuts `refs/tags/archive/`, then requires a slug and a sha, both non-empty.
The two agree, so the task's `-no-cutoff` path (`runBound`, `record.Older`) takes no wrong input from a skewed parse.
No `PATTERN.md` entry or changed pattern file makes the hand-parse a violation; `PATTERN-fabric-git` names the tag shape but assigns no sole reader.
The only cost is a second place to edit if the tag shape ever changes.

## Rejected

- Task `IsReadOnlyCommand` paren and dollar rules against the parent's other `fabricengine` edits: the parent's `archive.go`, `trailer.go`, `cleanupremotewarp.go` and `remoteleftover.go` changes only introduce `ArchiveTagPrefix` and `TaskCommitOf` and add no command forms, so no recorded command newly meets the rules.
- Parent-added callers of the `ReadOnly` hook: `loomcli/wiring.go`, `webstercli` and `websterengine` are the only users of `IsReadOnlyCommand`, and none is new in the parent's range; `reportengine`, `shedverbs` and `shortcutcli` do not classify commands through it, and `shortcuts.yaml` holds lyx words only.
- Vocabulary scan against the parent's new packages: the task's carve-out removal and the two new fixture rows touch only `enforcement_test.go` and `vocabscan_test.go`, and the plan's verify ran the scan on the merged tree.
- Changed pattern files: none of `PATTERN-leaf-packages`, `PATTERN-sole-parsers`, `PATTERN-glyph-conversion-chokepoint`, `PATTERN-cli-cobra`, `PATTERN-failure-stop-visible` or `PATTERN-told-geometry` constrains `tools/tokencount`, the `-no-cutoff` flag or its doc comment.
- Test economy: the task's new `IsReadOnlyCommand` rows, `time_integration_test.go` cases and fixture rows cover reader and cutoff behavior that the parent's `archivetag_test.go` and `trailer_test.go` do not; `count_test.go` still covers `archivedSlugs`, so the parent's `ArchiveTagSlug` leaves the tokencount cutoff covered.
