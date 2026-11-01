---
round: 1
focus:
  - "Whole diff: confirm it only deletes `tools/boltrename` and licenses nothing beyond the Design section's deletion, with no refactor riding along."
  - "Repository-wide, outside `.git` and `_lym`: does any mention of `boltrename` remain, in docs, build files, CI config, module or workspace lists, scripts, or Makefile targets, that the Design section's 'every mention' asks removed?"
  - "Repository-wide: does the old directory name `_board` still appear in tracked files, for example docs, specs or `.gitignore`, as a reference to the old checkout directory rather than as an unrelated substring such as `custom_board`?"
  - "Does anything outside `tools/boltrename` import its package or call its exported symbols, so that `assert-no-callers` for the Delete holds?"
  - "Check the commit message and any `_lym/landing/summary.md` claim ('every hub is on _bolt') against the Goal's account, where Neve is migrated and OSL-1046 is re-cloned."
---

The diff is three file deletions under `tools/boltrename`.
A scan for `boltrename` outside `.git`, `_lym` and `.lym` finds nothing.
The `_board` substrings that remain under `internal` are unrelated names (`no_board_path`, `local_board`, `custom_board`).
Review should therefore concentrate on non-Go references, such as docs and config, which the scan of the working tree may have missed under excluded directories.
