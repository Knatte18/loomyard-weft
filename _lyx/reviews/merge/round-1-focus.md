---
round: 1
focus:
  - "tools/tokencount/count.go parses `refs/tags/archive/<slug>/<sha>` by hand while the parent added `fabricengine.ArchiveTagPrefix` and `ArchiveTagSlug` in internal/fabricengine/archive.go: does the task's `-no-cutoff` path (runBound, record.Older) still agree with the archive-tag shape the parent now owns, and do PATTERN.md or the changed pattern files make the hand-parse a violation?"
  - "internal/fabricengine/refscanner.go (task: paren and dollar rules in IsReadOnlyCommand) against the parent's other fabricengine changes (archive.go, trailer.go, cleanupremotewarp.go, remoteleftover.go): do any parent-added command forms, such as git invocations with `$` or `(`, now run through a reader segment the task's rules reject or newly admit?"
  - "IsReadOnlyCommand is the `ReadOnly` hook in loomcli, webstercli and websterengine: did the parent add a caller or recorded command (reportengine, shedverbs, shortcutcli) whose command text contains an unquoted `$` or parenthesis and is now classified as not read-only?"
  - "internal/lyxcwd enforcement_test.go and vocabscan_test.go (task) against the parent's new packages (reportengine, shortcutcli, haltnote, shedadapters): do the parent's new files pass the fabric-vocabulary scan with only the one remaining `loomyard-weft` carve-out?"
  - "Changed pattern files (PATTERN-leaf-packages, PATTERN-sole-parsers, PATTERN-glyph-conversion-chokepoint, PATTERN-cli-cobra, PATTERN-failure-stop-visible, PATTERN-told-geometry): does any task file, such as the tokencount flag or its doc comment, break an invariant the parent changed?"
  - "Test economy: do the task's new IsReadOnlyCommand rows, tokencount time_integration_test.go cases and fabricVocabularyFailures rows duplicate coverage the parent's new fabricengine tests (archivetag_test.go, trailer_test.go) or lyxcwd scans now give, and does the parent's ArchiveTagSlug change leave the tokencount cutoff behavior uncovered?"
---

Seed focus for the merge review.
The overlap is `internal/fabricengine` only, with no overlapping files, so the interaction risk sits in shared symbols and patterns, not textual conflicts.
