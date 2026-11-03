---
round: 1
focus:
  - "Answer key is the board entry under `## Goal` of the decision record alone (other sections empty). Check that both dirty paths the entry names, the deleted `_lym/loom/friction/loom-halt.md` and the untracked `_lym/shed/<slug>/go-settle.jsonl`, are covered by the exempt set in `internal/preflight/preflight.go` `runStateExemptRels`. Settle question: does `shedrun.RunsRootRel()` actually contain the `_lym/shed/<slug>/go-settle.jsonl` path, or does the diff leave the settle-log half of the entry unhandled?"
  - "`internal/loomengine/config.go` `LoomFrictionArchivesRelPattern` and `internal/fabricengine/warpclean.go` `Clean`: settle how weftExempt entries become git pathspecs. Does the `<friction>-*` glob match the archive siblings and everything beneath them under git's pathspec rules, on a subpath anchor as well as a root anchor, and on Windows separators? Does the glob also over-match a non-archive sibling of the friction directory, or disagree with `IsLoomFrictionArchive`?"
  - "Does `Clean` treat a literal path entry and a glob entry correctly side by side, i.e. does the exempt list hide a deleted tracked file (the halt-note deletion) as well as untracked files? Read the exclusion code path in `warpclean.go` for both cases, since the diff changes only its doc comment."
  - "Scope: the friction directory also holds notes agents write, not only lym's. Does exempting the whole directory exceed what the entry licenses (lym-owned artifacts), and does the diff or its docs state that choice? Check the `preflight.go` and `doc.go` comments against what the code exempts."
  - "Test economy: `TestCheckResolved_LymOwnStateIsExempt` in `preflight_integration_test.go`. Settle whether each new fixture line (committed-then-deleted halt note, archive note, new friction note) drives a distinct exempt path, whether the test still checks that a file under `_lym` outside the exempt state refuses, and whether any behavior the diff adds is uncovered."
  - "Commit message and repository rules: the message claims the check skips the friction directory and its archives and closes #33. Check that claim against the diff. Check the repository's constraints document for docs or specs the change must update in the same diff, such as docs that describe the Preflight worktree-clean contract."
---

Seed pass: focus set from the board entry, the review-range diff and its single commit message.
No `_lym/landing/summary.md` or plan was read; the answer key is the decision record's `## Goal` alone.
