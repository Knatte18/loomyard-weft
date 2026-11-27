```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness self-check of the fix change set.

no findings

I traced these paths:
- the chained-argv `read()`
- the bounded backward loop in `answer`
- `Remaining`
- the OS skip, shell binding and uid bindings in `newSession` and `PlaySession`
- `checkNameBindings`
- the `ResolveShellPath` rename
- the moved unanswered-call cleanups
- the `session-ends` fatal
- `startedPanePID` against the `{pid:4}` pane
- the new F7 and F9 tests

None gave a wrong result or a broken caller, and the F7 stale-read row fails against the old backward loop.

Tool calls:
- `git grep` for stale `resolveShellPath` and `--tier tier2` references: it failed on pathspec syntax and read nothing. My context held only the Go caller search under `internal/`, not `docs/`, `pattern/` or `cmd/`.
- `grep -rn` for the same two strings over `internal`, `docs`, `pattern` and `cmd`: same reason. It found no matches (`cluster.go` excluded, as its `resolveShellPath` is an unrelated function).
