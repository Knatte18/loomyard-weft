```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every claim the fix added or rewrote against the code at HEAD.

no findings

What I checked:
- **Card 8's goos claims:** `verifyrun.QuoteArg(goos, value)` returns `ErrPercentUnderCmd` only when `goos == "windows"` and the value holds `%`. Otherwise it single-quotes the value. So "refused under `windows`, … quoted under `linux`" holds (`internal/verifyrun/run.go:65`).
- **Card 9's `[TestFixture]` and `[Test]` claim:** card 4 defines `TestAttributed` as a type carrying `TestFixture` or `TestClass`, and `HasTestMethod` as a member carrying `Test` and the other test attributes. Card 5 makes such a class a test class. So the stated seed classes are test classes.
- **Card 6's wrapper claims:** both signatures match the code. `ChangedPaths` matches `changedPathsInTree(ctx, worktree, base) ([]string, error)` at `internal/impactset/targets.go:89`, and `AncestorBase` matches `resolveAncestorBase(worktree, base) (string, bool)` at `internal/impactset/derive.go:20`. The behaviour described also holds: tracked changes through `git diff --find-renames base`, a rename at both paths, and untracked files through `ls-files --others --exclude-standard`.
- **"in card 11, the inert check":** card 11 (inert-paths) lists `plan:internal/impactset#ChangedPaths` in its Uses and calls it from `allInertSince`.

Tool calls:
1. Read `QuoteArg` in `internal/verifyrun/run.go`, `changedPathsInTree` in `internal/impactset/targets.go`, and the attribute lines of cards 4 and 5, since my context held only their signatures or grep hits.
2. Read card 4's lines 23–24, since the grep output was cut short.
