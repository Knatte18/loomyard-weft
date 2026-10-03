---
round: 1
ledger:
  - key: seam-import-allowlists-lack-stale-detection
    rounds: [1]
    status: resolved
  - key: cli-walk-bogus-accepts-any-refusal
    rounds: [1]
    status: resolved
  - key: cli-tree-findings-never-notice-fix
    rounds: [1]
    status: resolved
  - key: dropped-concept-claims-case-sensitive
    rounds: [1]
    status: resolved
  - key: tmux-isolation-per-file-platform
    rounds: [1]
    status: resolved
  - key: tmux-isolation-kit-call-by-identifier
    rounds: [1]
    status: resolved
  - key: changed-comments-fixed-column-wrapped
    rounds: [1]
    status: open
  - key: card29-helpers-detached-doc-comments
    rounds: [1]
    status: resolved
  - key: leaf-test-counts-unrecorded
    rounds: [1]
    status: open
  - key: step-envelope-docs-restate-names
    rounds: [1]
    status: resolved
  - key: sectionof-trailing-clause-untested
    rounds: [1]
    status: resolved
---

Round 1 reviewed the whole branch over its merge base with `main` (cards 1–34), with no rework round.
The review found ten items; the fixer committed fixes for nine, plus one defect it found itself: card 29's per-file `StatusRel` helpers had split doc comments from the functions they documented.

Verified resolved on HEAD: stale detection on the import allowlists of cards 8 and 9, the CLI walk's unknown-subcommand pin, known-finding exemptions that fail once fixed, case-folded dropped-concept claims, per-platform tmux isolation with file-name suffixes, kit-call resolution by import path, the reattached doc comments, the removed step-envelope docs and the `sectionOf` trailing-clause test.

Open after round 1:

- **leaf-test-counts-unrecorded:** the before-and-after leaf test counts the `test-accounting` decision requires appear nowhere on the branch; the fixer left this item as a placeholder.
- **changed-comments-fixed-column-wrapped:** the bulk rewrap landed, but `cmd/lyx/seamsignature_test.go:7-8` and `internal/loomrecipe/shape_test.go:1-2` are rewritten lines still broken mid-sentence.
