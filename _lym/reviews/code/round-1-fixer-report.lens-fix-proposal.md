```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal, an absent-keys column in the hubforge test with an exit-1 assertion, as carried out in `internal/hubforge/hub_test.go`.

no findings

Cases I tried against the change:
- **Global or system config leaking in.** The call uses `--local`, so it reads only the repository's own config file. A developer's `user.name` cannot produce a false failure or a false pass.
- **Exit code other than 1.** The check requires exactly `ExitCode() == 1`. A git spawn failure, or exit 128 from a bad repository, fails the test instead of passing as "absent".
- **The defect in place.** If `initBareRepo` flipped its `true` literal to `false`, the bare repository would get `user.name`. The command would exit 0, `err` would be nil, `errors.As` would be false, and the test would report an error. The proposed assertion therefore fails with the defect in place.
- **Scratch row.** Its `absent` is nil, so the loop does nothing there and no existing row changes.
- **Bypassing the gitkit fixture environment.** The test calls `exec.Command("git", ...)` directly rather than through `gitkit.Git` or `gitExit`, which are unexported. A read of the repository's own config does not depend on that environment, and `hub.go:490` already shells out to git directly. Nothing breaks.

Further tool calls: none. My context already held every file I needed.
