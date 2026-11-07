```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, run on the changed shell, wrapper and fixture code. I made no tool calls.

**Per-branch list**

- `post-checkout.sh` `PARENT`/`BASE`, input `/x/hub`: `PARENT=/x`, `BASE=hub`, so `WEFT_WORKTREE=/x/hub-weft`. This matches `dirname` and `basename`. Correct.
- `post-checkout.sh`, root-level `/x`: `${..%/*}` gives an empty `PARENT`, so `WEFT_WORKTREE=/x-weft`. The old code built `//x-weft`, which is the same path on POSIX. Correct.
- `post-checkout.sh`, `/` itself: `PARENT` and `BASE` are both empty, so the path is `/-weft`, where the old code gave `//-weft`. `git rev-parse --show-toplevel` cannot answer `/` for a worktree, so this is unreachable.
- `post-checkout.sh`, Windows-style `C:/a/b`: gives `C:/a` and `b`. Correct.
- `post-checkout.sh`, the empty `WORKTREE_ROOT` guard, still runs before the expansion. Correct.
- `chainedHookPreamble`, `$0=.git/hooks/post-checkout`: `SCRIPT_DIR=.git/hooks`. This matches `dirname`. Correct.
- `chainedHookPreamble`, `$0=/post-checkout` (slash at the root): `${0%/*}` is empty, so the test runs `-x "/post-checkout.user"`, which is the right file. `dirname` would give `/`, so this is equivalent.
- `chainedHookPreamble`, `$0=post-checkout` (no slash): the `*)` branch sets `SCRIPT_DIR=.`. This matches `dirname`. Correct.
- `chainedHookPreamble`, Go raw string: it contains no backtick, so the literal is valid. Correct.
- `chainedHookPreamble`, idempotency: the sentinel is carried only by `postCheckoutScript`, and the preamble does not touch it. `InstallPostCheckoutHook` still returns early on the second install. Correct.
- `chainedHookPreamble`, old-format wrappers already installed: they keep their sentinel, so they are left alone and keep working. Correct.
- `TestInstallPostCheckoutHook_ChainIdempotent` probe: `git checkout -b` in the prime worktree fires the wrapper.
  - `post-checkout.user` is executable (0755) and echoes `user`.
  - git sends hook stdout to stderr, which `CombinedOutput` captures.
  - `"user\n"` does not occur in git's own "Switched to a new branch" line, so the match is specific.
  - The fabric body then warns on the branch mismatch and exits 0, so the checkout succeeds. Correct.
- The probe on a copied hub: the template's hook carries the sentinel, but the test overwrites `hookPath` with the plain user hook before installing, so the chain path is taken. Correct.
- `newRecordsFixture` and the adoptcli sites, `NewHub` swapped for `CopyHub` with the same `Shape{Anchor: "."}`: the copy carries the same hub state. A comment at `destructivegaps_integration_test.go:82` says the template comes from `NewHub`, and many other fabricengine tests already use `CopyHub` with `CloneSecond` and `AddPair`. Each test gets its own copy, so parallel tests stay isolated. Correct.

**Findings:** none.

**Tool calls:** none. My inherited context held everything this lens needed.
