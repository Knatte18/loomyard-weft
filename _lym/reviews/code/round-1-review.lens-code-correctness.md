```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function and branch on a concrete input. No tool calls were needed.

**Per function or branch, the case I ran:**

- `ResolveWithin`, cwd is `hub/X-weft/_lym/plan`, `hub/X` exists, `X-weft/.lym-anchor` holds `backend`.
  - `gitWorktreeRoot` gives `hub/X-weft`.
  - `ownerOfRecordsSibling` gives (`X`, true).
  - `anchorRelOrRoot(hub, "X")` reads the marker through `weftname.SiblingPath(hub, "X")` and returns `backend`.
  - The anchor check is skipped.
  - `buildLocation` returns `WorktreeName` `X` and `AnchorRel` `backend`, the same as `Resolve` from `X/backend`.
  - Correct.
- `ResolveWithin`, cwd is `X/backend/_lym/plan` through the anchor's link into the records sibling.
  - `EvalSymlinks` resolves into the sibling, so this takes the records-sibling branch above.
  - Where a junction stays unresolved, the walk lands at root `X` and `pathWithin` accepts the cwd as below `backend`.
  - Correct either way.
- `ResolveWithin`, plain worktree `X` with no marker.
  - `anchorRelOrRoot` returns `.`.
  - `pathWithin` accepts any cwd under `X`.
  - Correct, because the anchor is the whole worktree.
- `ResolveWithin`, cwd is `X` while the anchor is `backend`, or `X/frontend`.
  - `pathWithin` returns false.
  - The result is a wrapped `ErrCwdOutsideAnchor` naming `X/backend` and returning a nil layout.
  - Correct.
- `ResolveWithin`, worktree named `foo-weft` with no `foo` beside it.
  - `ownerOfRecordsSibling` returns (`foo-weft`, false) because the stat fails.
  - It resolves as itself under the anchor check.
  - Correct.
- `ownerOfRecordsSibling` boundaries.
  - Name `-weft` gives an empty owner, so it returns (name, false).
  - A name without the suffix returns (name, false).
  - An owner path that is a file, not a directory, returns (name, false).
  - Correct.
- `ResolveWithin`, cwd outside any git repo, or missing.
  - `gitWorktreeRoot` returns the bare `ErrNotAGitRepo`.
  - Correct.
- `ResolveWithin`, stale `.fabric-anchor` marker.
  - `readRecordedAnchor` returns `ErrStaleAnchorMarker`, which propagates unwrapped, as in `Resolve`.
  - Correct.
- `ResolveWithin`, cwd reached through a symlinked path.
  - `workTreeRoot` is the real path. `pathWithin` normalizes both sides with `EvalSymlinks`, so the comparison matches.
  - Correct.
- `pathWithin`, boundary inputs.
  - child == parent gives rel `.`, so true.
  - A sibling directory gives rel `../x`, so false.
  - A directory named `..foo` gives rel `..foo`, which is neither `..` nor `../`-prefixed, so true. Correct.
  - A nonexistent child falls back to `Clean`, which is correct for an unresolved path.
  - Different volumes or a mixed absolute/relative pair make `Rel` error, so false.
  - Windows lowercasing is applied to both sides before `Rel`.
  - Correct.
- `checkCwdWithinAnchor`.
  - The wrapped error carries `ErrCwdOutsideAnchor` and both paths.
  - Correct.
- `anchorRelOrRoot`, used by `NamedWorktree`, `resolveCore` and `ResolveWithin`.
  - The refactor preserves the old behavior: found returns the anchor, not found returns `.`, and an error returns the error.
  - Correct.
- `shuttlecli/cli.go` pre-run.
  - Cobra's `Name()` for `Use: awaitForksVerb` is `await-forks`.
  - The swap applies only to that verb. The `awaitVerb` and hook-batching early return still comes first.
  - The `output.Err` and `Abort` passthrough is unchanged.
  - A layout from a weft cwd gives the same hub, worktree and anchor path as the anchored directory, so config, reed geometry and the strand lookup are unchanged.
  - Correct.
- `ErrCwdOutsideAnchor` text change.
  - The only text matcher is `internal/webstercli/cli_tmux_test.go:59`, which looks for the identifier name, not the message.
  - The other uses I saw use `errors.Is`: `internal/driftcli/cli.go:262`, `internal/preflight/predicates.go:109` and the gate tests.
  - The new text stays true for the strict gate.
  - Correct.

**Findings:** none.

Tool calls: none; the manifest already held every definition and caller I needed.
