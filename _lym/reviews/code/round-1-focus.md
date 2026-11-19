---
round: 1
focus:
  - "internal/landingshed/finalize.go `landedDiffEmpty`: is `parentHeadMerged` the right base for the emptiness test, and does a failed or unavailable path read fail safe to not-empty rather than hiding or inventing an empty-landing flag?"
  - "internal/shedrun/landed.go landed-commit file format: do an old one-line file, a trailing newline and a missing file all read back correctly, and does every reader and writer of `LandedCommit` carry `EmptyDiff` through to `RunView.EmptyLanding` and the dobby line?"
  - "internal/darnengine `report_dir`: does `LoadConfig` refuse every directory that escapes the worktree, does the `docs` profile template set it, and does the `darn-fragment-report` passage render for every engine with a valid `report_path` when the task names none?"
  - "Scope against the board entry: it asks for the docs stencil to require a committed report and for Land to refuse or flag an empty diff in dobby's line; check the diff does that and nothing beyond it, such as a new refusal path in Land."
  - "Mutation survivors at internal/landingshed/finalize.go:741 and internal/shedrun/landed.go:43: could a test assert the `RecordLanding` empty-diff argument and the `empty-diff` mark in the file, and is that the diff's own behavior?"
  - "Commit-message and `_lym/landing/summary.md` claims: does the diff bear out each one, including the dobby notice wording?"
---

The change spans two parts: darn's report stencil and the landing's empty-diff flag.
The record's `## Goal` is the whole measuring stick; its other sections are empty.
