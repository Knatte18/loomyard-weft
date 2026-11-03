---
round: 1
focus:
  - "internal/preflight/preflight.go `runStateExemptRels` and the shedrun path helpers: does the single exempt root `shedrun.RunsRootRel()` cover every file the stop and settle paths write under `_lym`, or does some stop/settle writer (settle log, tackler/arbiter attempts, stop markers, halt marker from #22, friction or notice files) land outside that root and still make worktree-clean refuse? Grep the writers rather than trusting the commit message."
  - "internal/fabricengine/warpclean.go `exemptPathspec` and `weftCommitPathspec`: with `AnchorRel` set to `.` and to a subpath, does the `.` plus `:(exclude)` pathspec exclude exactly the exempt root, and nothing else, relative to the weft worktree's cwd? Check path separators, the empty-anchor case, and that a dirty file outside the root, including a sibling under `_lym`, still counts."
  - "Claim that the exempt records are committed by the run's next status or records commit: find the code that makes it true. If an exempt file can stay uncommitted across a later step that does check cleanliness (merge, land, fabric commit on a clean-required path), say which one."
  - "Scope against the Goal in the decision record: it asks for a pattern fix decided once in one place, and a test that walks the files the stop and settle paths write. Does the diff decide once (exemption in preflight only) while other `fabricengine.Clean` callers keep the old behaviour, and is that consistent with the Goal? Does the test derive its file list from the writers, or hand-list shedrun helpers that can drift from what the stop and settle paths write?"
  - "internal/fabricengine/dirtiness.go `worktreeDirty` variadic `pathspec`: do all existing callers still behave the same, and is the `--` separator placement correct for both scopes?"
  - "Test economy and coverage in internal/preflight/preflight_integration_test.go: does `TestCheckResolved_RunStateUnderLymIsExempt` duplicate an existing Clean or CheckResolved test, and is the added `fabricengine` exemption parameter covered at the fabricengine level or only through preflight?"
  - "Change description and doc comments: check the `doc.go` paragraph and the `Clean` doc against what the code does, including the weft-side-only scope of the exemption."
---

Seed pass: focus is drawn from the diff and the decision record's Goal only, no prior round exists.
