```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function and branch with a concrete input.

**Case list**

- `buttonRunDirectory` (`internal/reedcli/button.go`), spec.Dir is the worktree root with the anchor marker "backend" written after the render. `ResolveWorktree` re-reads the marker through `readRecordedAnchor` on each call and returns `<hub>/<wt>/backend`. Correct, and nothing is cached.
- `buttonRunDirectory`, spec.Dir is an old anchor subdirectory or a parent's anchor on a task session. `dotgit.FindRoot` walks up to the worktree root and the anchor is re-read. Correct.
- `buttonRunDirectory`, spec.Dir is in no git repo. It returns `ErrNotAGitRepo`, which is unwrapped and passed to the caller. Correct.
- `buttonRunDirectory`, a stale or invalid marker. `readRecordedAnchor` returns `ErrStaleAnchorMarker` or an "unusable" error, which is passed on. Correct.
- `buttonRunDirectory`, spec.Dir is a subdirectory of a repo outside any hub, such as a `window`-verb directory. The code returns the repo root with anchor ".", not spec.Dir. I did not confirm that buttons render for such directories. If they do, the click runs in the wrong directory. Not confirmed.
- `runButtonVia`, the existence check, runs before `anchorOf` and on `spec.Dir`. When the new anchor subdirectory is missing on disk, nothing checks `runDir`.
  - A plain entry reaches `deps.run` in a missing directory. It fails to start and shows `failed (exit -1): chdir…`.
  - A confirm entry raises a prompt whose line opens in a missing directory. The directory-change error is discarded. The confirmed pass then fails the same way, after the operator has answered y.
  - Neither path gives the "reattach" guidance. Incorrect (finding F1).
- `runButtonVia`, `anchorOf` error. It logs a Warn, says the anchor does not resolve, and runs and prompts nothing. Correct.
- `runButtonVia`, `req.Confirmed` with a mismatching hash. `anchorOf` now runs before the hash check, so a stale confirm hits the anchor message first. Harmless.
- `runButtonVia`, the confirm path with `runDir`. `composeConfirmedButtonLine` opens in `runDir` and the verb is re-entered. The second pass re-resolves from the same `spec.Dir`, so it agrees with the first unless the marker changes between prompt and answer, which then runs in the newer anchor. Correct.
- `runButtonVia`, the log and message sites after the anchor read use `runDir` (prompt-failure message, line, run). The "gone" message and the Warn deliberately keep `spec.Dir`. Correct.
- `RunButton`, a nil `anchorOf`. It would panic at `deps.anchorOf`. The only production caller (`internal/reedcli/button.go:77`) passes a non-nil function, so it is unreachable. Correct.
- `runDir != spec.Dir` Info log, spec.Dir behind a symlink. `ResolveWorktree` calls `EvalSymlinks`, so `runDir` is the resolved path. The Info line then fires on every click and the process runs in the resolved path. This is functionally fine, because the verb's anchor gate uses `samePath`. It is only log noise (finding F2).

**Findings**

- F1, LOW, design, `internal/reedengine/buttonclick.go:101-115`. The existence check runs on `spec.Dir` and `runDir` is never checked. When the recorded anchor is a subpath that is missing on disk, the click ends in a start failure. The prompt path asks for confirmation first and then fails. The existing "is gone; reattach" message is not shown. Evidence: lines 101 and 107-112, and `readRecordedAnchor`, `internal/lymcwd/anchor.go:161`, which validates only the marker text, not existence.
- F2, NIT, consistency, `internal/reedengine/buttonclick.go:113`. The `runDir != spec.Dir` comparison also fires for a symlink-only difference, so the "not its worktree's anchor" Info log can be false. Evidence: `gitWorktreeRoot`, `internal/lymcwd/lymcwd.go:175`.

Tool calls: none; my context already held every definition I needed.
