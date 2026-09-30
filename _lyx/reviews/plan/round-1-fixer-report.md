# Plan round 1 — fixer report

All eight findings are fixed.
`lyx loom validate-plan` passes on the fixed plan (`ok: true`), with four informational `create-new-unit` notes for the new files only.
`approved:` is still `false`.

## Changes

- **F1 (BLOCKING):** card 2 now runs `git check-ignore` through the checked `gitexec.Run`.
  Exit 0 means ignored, a `*gitexec.GitError` with `ExitCode` 1 means not ignored, and any other error is returned.
  The card now forbids `gitexec.RunGit` and names the pinned guard in `cmd/lyx/checkedcall_test.go`.
  The overview's `git-stays-in-fabricengine` decision now says no new raw site is added.
- **F2 (MEDIUM):** card 6 (was 5) adds `records_integration_test.go` as a target.
  After the approved step, that test asserts the pair's anchor `.vscode/tasks.json` holds the single `reed attach` `folderOpen` task and that `git status --porcelain` in the pair is empty.
  The card now names which integration tests reach `OpenIDE` and which never do.
- **F3 (MEDIUM):** `PathTracked` is split out into a new card 3, `fabric-path-tracked`, with its own file (`pathtracked.go`) and integration test file.
  Card 2 keeps `ExcludeAnchoredDir` only.
  The later cards are renumbered 4–6 (files, headings, Card Index, `Applies to`, cross-references).
- **F4 (MEDIUM):** card 2 adds a lock-held test.
  It takes the exclude lock, asserts the call blocks with the file unchanged, releases the lock, and asserts the line is added and the custom line survives.
- **F5 (LOW):** card 4 now specifies what the shared helper returns: worktree path, color, the prime's name, and whether it resolved.
  The helper logs the prime warning exactly once, and `Spawn`'s prime branch stays keyed on the returned values.
- **F6 (LOW):** card 1 declares `plan:internal/vscode#TaskChainInteractive` -> `const TaskChainInteractive TaskChain = 0`.
  Card 4 lists it in `Uses`, since it edits `Spawn`, which keeps passing it.
- **F7 (LOW):** card 4 now checks whether `tasks.json` is tracked before writing the exclude entry.
  When it is tracked, the card skips both the exclude write and `WriteConfig`, warns, and launches.
  The tracked-file test now also asserts the shared `info/exclude` is byte-for-byte unchanged.
- **F8 (LOW):** the untagged `go test` line in `## verify:` now includes `./cmd/lyx/...`.

## Deferred

None.
