```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the F2 `TestFilePath` table and the F3 `TestUserConfigDir` row.

No findings.

- **F3 row** `{"RelativeXDGWithEmptyHome", "linux", "", ".xdg", "", Join(".xdg","lym"), "", ""}`:
  - `userConfigDirFor` takes the non-empty `xdgConfigHome` and returns `.xdg/lym`, true. That matches `wantPlain`.
  - `absoluteUserConfigDirFor` finds `.xdg` is not absolute and `home == ""`, so it returns ("", false, nil). That matches the empty `wantAbsolute` and no error.
  - Returning the relative directory from that branch would fail the row.
- **F2 `ResolvedDirectory`**: with `LOCALAPPDATA` and `XDG_CONFIG_HOME` set to the absolute temp dir and `HOME` empty, both platforms resolve to `base/lym`. The row asserts `machine.json` and `machine.lock` under it.
- **F2 `NoVariableSet`**: all three variables are empty on either platform, so `AbsoluteUserConfigDir` returns ("", false, nil). `FilePath` returns "", "", false, nil, and the zero-value `wantOK` and `wantDir` expect exactly that.
- **F2 `RelativeVariableIsAnError`**: `LOCALAPPDATA` and `HOME` are both "relative".
  - Linux: `XDG_CONFIG_HOME` is empty, so the code reaches the `!IsAbs(home)` check and errors with `errRelativeHome`.
  - Windows: `!IsAbs(localAppData)` errors with `errRelativeLocalAppData`.
  - `FilePath` wraps either error with "...way forward: set that variable to an absolute path". That contains "absolute path", and the variable name comes from the wrapped error. The existing `TestUserConfigDir` already asserts that name for both errors.
  - Removing the wrap would fail the "absolute path" check, and ignoring the variable would fail the error check, so the row fails with the defect in place.
- **Test mechanics**:
  - Neither the parent nor its subtests call `t.Parallel`, which is right for `t.Setenv`.
  - The new `runtime` import is used.
  - `relativeVariable` is selected per platform.
  - `FilePath` and the standalonestate functions are unchanged, so no caller is affected.
- **F1**: a docs-only change, outside this lens.

Tool calls:
1. `grep` of `internal/standalonestate/standalonestate.go` for `userConfigDirFor`, `absoluteUserConfigDirFor` and `AbsoluteUserConfigDir`. My context held only the test file and `machinename.go`, not these implementations, which I needed to run the rows step by step.
