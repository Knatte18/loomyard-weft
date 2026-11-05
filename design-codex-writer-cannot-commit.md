# A Codex writer in a pair worktree cannot commit: the git dir is outside its sandbox roots

`codex-writer-cannot-commit` · Task · bug, codex, shuttle

## Found

In the sandbox hub `lym-test-LYMHUB`, run `codex-greet` (darn, writer `luna[medium]` on the Codex engine) stopped at Darn-Write with `gate did not pass after 3 attempts`.
The writer implemented the change and passed its own checks, but every `git add … && git commit` failed:

```
fatal: Unable to create '/home/knatte/Code/lym-test-LYMHUB/lym-test/.git/worktrees/codex-greet/index.lock': Read-only file system
```

It tried three times, once per re-prompt of the commit gate, and each turn ended with the five files uncommitted.
The rollout shows it then asked its parent for help through `lym creel post --parent`.

## Cause

The launch line runs `-s workspace-write` and adds only the records checkout as a writable root.
A linked worktree's git directory lies under the main checkout's `.git/worktrees/<slug>`, outside the worktree and outside every writable root, so the sandbox refuses the index lock.
It is deterministic: any Codex writer in a pair worktree cannot commit, so no darn or tapestry run with a Codex writer can land.

A second, smaller refusal of the same kind: the default Go build cache under `~/.cache/go-build` is read-only, so the first `go test` failed until the writer set a cache under `/tmp`.

## Way forward

`codexengine` should add the worktree's git directory, and the common git directory it links to, as writable roots next to the records checkout, resolved from the `.git` entries as the trusted roots are.
`go` cache and temp paths need a writable location too, such as `GOCACHE` under the lym temp root.
