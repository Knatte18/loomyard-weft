```yaml
lens: fix-code-correctness
outcome: reported
```

Task restated: check the correctness of the F2–F6 code changes (overview note, replayer test, settings row, comment edits).

no findings

- `RenderOverview` / `overviewText` (`overview.go`): with an `agents.yaml` that fails to load, `layers.DerivedWarning` is set and `Derived` is nil.
  The claude-settings section gets `\nNote: the effort defaults are left out of <file>: <err>.` and no `lym-agent-defaults` entry.
  The new test asserts exactly that prefix and the missing entry, so it fails without the note.
  `overviewText`'s signature is unchanged, so no caller is affected.
- The warning text names no absolute path.
  `unusable` builds it from the key, the detail and `wayForward`, so the committed `HEDDLE.md` does not differ between machines.
- `TestRenderOverview` rows: the text `Use ```go fences.` gives a four-backtick fence, which a three-backtick fence would not satisfy.
  `not json` in claude-settings falls through `decodeObject` and shows as written in a `json` block.
  Both assertions fail with the branch removed.
- `CommitsOwnOverviewNeverLands`: `Apply` ignores the `HEDDLE.md` file in the request.
  The result is compared to an overview rendered from `{a,b,c}`, so a stale or local overview landing on disk would fail the test.
- `RepositoryEntryWinsOverTheDerivedEffort`: the repository `effortLevel` is `high` and the derived one is `medium`.
  The expected object holds `high` plus the derived `env`, so the row fails if the derived entry wins.
- F5 (comment) and F6 (blank line) are text only.
  The gate on `./internal/heddleengine` passed (128 of 130, 2 skipped, 0 failed), and `go vet ./cmd/lym` was clean.

Further calls:
1. `grep` / `sed` on `internal/agentconfig/config.go`: my context lacked the text of `agentconfig.Load`'s errors, which I needed to check that the warning embedded in `HEDDLE.md` holds no machine-specific path.
2. `sed -n 215,236p` on the same file: to read `unusable`, the function that builds those errors.
