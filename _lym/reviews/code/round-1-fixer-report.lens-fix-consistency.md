```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set for agreement with its surroundings (fix-consistency lens).

One finding:

- **LOW** — `internal/driftcli/cli.go`, the narrowed lasting check in `runCaptureChild`. `internal/driftcli/doc.go:20` and the `LastingFaultKey` comment (`internal/driftengine/spawn.go:26`) still describe the old scope.
  - **Old text:** `doc.go:20` says "A child whose worktree does not resolve from its cwd marks its error envelope with `driftengine.LastingFaultKey`". `spawn.go:26` says "such as a cwd its worktree does not resolve from".
  - **Why they now disagree:** An unresolvable cwd that is neither `ErrNotAGitRepo` nor `ErrCwdOutsideAnchor` no longer gets the mark. Cases are a `CwdFrom` failure or an unclassified `Resolve` error. The review's F2 text called those lasting, and the fixer's `fix` key narrowed the set to two sentinels. The doc sentence now over-claims a little. The `spawn.go` wording hedges with "such as", so it holds.
  - **Evidence:** The new condition is `errors.Is(err, lymcwd.ErrNotAGitRepo) || errors.Is(err, lymcwd.ErrCwdOutsideAnchor)`. `openWorktree` (`wiring.go:30-40`) returns `CwdFrom` and `Resolve` errors unchanged.
  - **Suggested fix:** Reword `doc.go:20` to "A child whose cwd is outside a git repository or is not the anchored directory".

Checks that held:
- `ErrCwdOutsideAnchor` is wrapped with `%w` (`anchor.go:140`), so `errors.Is` matches the integration assertion in `anchor_integration_test.go:77`.
- The `spawn_test.go` rows agree with `readChildReport`. The later `ok:false` envelope replaces the earlier one, and `lasting` resets to false.
- The `ok:true` envelope is skipped by the `ok != false` filter.

Tool calls:
1. `grep` of `openWorktree` and `sed` of `anchor.go:136-142`, to see the errors `openWorktree` returns and whether the anchor error is wrapped with `%w`. My context did not hold `wiring.go` or the wrapping line.
