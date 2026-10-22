---
round: 1
focus:
  - "internal/planglyph DoneChecks disk route: does every target that reaches the disk check keep the Create/Rename-new exists and Delete/Rename-old absent verdicts, and can a path escape the worktree (absolute, `..`) or hit a directory instead of a file?"
  - "planparser.IsDiskCheckedTarget: does the shape-ledger gate match exactly the targets that canonicalization leaves path-shaped, with no overlap with glyph-spelled targets, so planglyph makes no ref-shape decision of its own?"
  - "The not_found listing: is it run only when a Create/Rename-new finding is due, are listing errors wrapped in ErrQuarryUnavailable, and does the detail correctly name the unit's files and same-name declarations elsewhere (issue #823)?"
  - "The glyph-rejected way forward: does the new text (rebaseline --card NN, then recover-batch, fresh-restart steps for uncheckable entries) match what the websterengine code actually allows, and does it stop pointing at recover-batch alone?"
  - "Test economy: the fold of TestDoneChecks_RootFilenameCreateLanded into TestDoneChecks_Verdicts, and coverage of each new behavior (spaced Create, Delete/Rename disk sides, not-found detail, listing error, glyph-rejected message)."
  - "Docs and summary: doc.go in planglyph and websterengine and contracts/specs/loom-plan-spec.md agree with the code, and every claim in _lyx/landing/summary.md (including the tripwire allowlist additions) is borne out by the diff."
---

Scope against the board entry: `webster validate` must stay unchanged and the `loop-exited` halt reason must not be touched.
