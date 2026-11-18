```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the merge resolution and 26b29bb8 add or change, find the test row that would fail if it broke.

**Per behaviour, the test that pins it:**
- **Merge e6e36704 keeps `idBytes = 16`**, which `NewID` uses: `TestNewID`, `internal/machinename/machinename_test.go:218-230`. It wants a 32-character hex id, so a wrong `idBytes` fails it.
- **Merge e6e36704 keeps `MaxNameLength = 63`**, exported from main and used by `CheckName`: `TestCheckName` rows `MaximumLength` and `OverMaximumLength`, `internal/machinename/machinename_test.go:245,252`. They check 63 and 64 characters. Its callers in `internal/creelengine/crossmachine.go:52,62` are main's own code and not this diff's subject.
- **26b29bb8 restores `Names()`**, which returns `resolveNames(machineFile, os.Hostname)`: no test calls `Names` directly. The resolution behaviour it hands back is pinned by `TestResolveNames` (`machinename_test.go:68-135`, including the row `FormerNamesDropEmptyEntriesAndKeepInvalidOnes`) and `TestResolveNames_UnreadableFileIsAnError` (`:144-155`). `Names` is a one-line wrapper with no logic of its own, and main shipped it the same way. Its two callers, `internal/creelcli/slackbridge.go:36` and `internal/slackcli/wiring.go:381`, wire it in production through a seam (`names func() (string, []string, error)`), and the tests replace that seam with a fake. Wiring it wrongly is a compile error, which the gate already decides.

**Findings:** none. Every behaviour the diff adds or changes has a test row that would fail if it broke. The unwrapped `Names` wrapper leaves no coverage gap, since it adds no behaviour beyond `resolveNames`, which is tested.

**Tool calls:**
1. Searched `internal/machinename/*_test.go` and the other `_test.go` files under `internal` for `MaxNameLength`, `NewID`, `idBytes` and `Names`. My context had only the test function names, not their rows or assertions.
