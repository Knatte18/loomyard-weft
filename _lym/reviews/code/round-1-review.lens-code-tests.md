```yaml
lens: code-tests
outcome: reported
```

Task: list the test that pins each behaviour the diff adds or changes (lens code-tests), and report findings.

**Per behaviour**
- `resolveNames` with no directory, or with no file, falls back to the hostname rule. `TestResolveNames` rows `NoDirectoryReadsTheHostnameRule` and `NoFileReadsTheHostnameRule` pin it.
- A file name wins over the hostname. Row `FileNameWinsOverTheHostname` pins it.
- Unknown keys are ignored. Row `UnknownKeyIsIgnored` pins it.
- `Names` drops empty entries and the current name, and keeps a non-empty entry that fails the name rule. Row `FormerNamesDropEmptyEntriesAndKeepInvalidOnes` pins it.
- A file that does not decode gives an error naming `machine.json` and `lym machine name`. Row `UndecodableFileIsAnError` pins it.
- A file with an empty or invalid name gives the same kind of error. Rows `EmptyNameIsAnError` and `InvalidNameIsAnError` pin it.
- A file that exists but does not read gives an error naming the path. `TestResolveNames_UnreadableFileIsAnError` pins it, using a directory at the file's path.
- The error from a relative variable is passed through and never replaced by the hostname name. Row `RelativeVariableIsItsError` pins it.
- An unreadable hostname error carries its own way forward. Row `UnreadableHostnameNamesItsWayForward` pins it.
- `CheckName` rules: `TestCheckName` covers valid names, uppercase, dot, colon, underscore, leading and trailing hyphen, 63 and 64 characters, and empty.
- `FilePath` returns the `machine.json` and `machine.lock` names under the config directory. `TestFilePath` pins it.
- `FilePath` returning ok false when no variable is set: none. `TestFilePath` sets both variables.
- `FilePath` wrapping a relative-variable error with "set that variable to an absolute path": none. `RelativeVariableIsItsError` injects the error through the locator, so the wrap in `FilePath` is never run.
- `Name` and `Names` wiring to `machineFile` and `os.Hostname`: no direct test. This follows D8, since a test would read the developer's real file.
- `UserConfigDir` takes a relative `XDG_CONFIG_HOME` as given, on Linux and Windows. `TestUserConfigDir` rows `RelativeXDG` and `WindowsRelativeLocalAppData` pin it.
- `AbsoluteUserConfigDir` ignores a relative `XDG_CONFIG_HOME` and falls back to `$HOME/.config/lym`. Row `RelativeXDG` pins it.
- `AbsoluteUserConfigDir` errors naming `HOME` or `LOCALAPPDATA` when either is relative. Rows `RelativeHome` and `WindowsRelativeLocalAppData` pin it.
- `AbsoluteUserConfigDir` with a relative `XDG_CONFIG_HOME` and an empty `HOME` returns unset: none. The `Unset` row has `XDG_CONFIG_HOME` empty, so a regression in that branch would not fail any row.
- The `errRelativeHome` text now naming `HOME`: row `RelativeHome` checks that text. Its `LOCALAPPDATA` counterpart is pinned by the Windows row.
- `CredentialDir` returns the `UserConfigDir` result under a relative `XDG_CONFIG_HOME`. `TestCacheDirRedirection_HonoursOverride` pins it.
- `DobbyLandings` counts a landing under a former name and excludes other machines and unstamped entries. `TestDobbyLandings_CountsOnlyThisMachine` rows `renamed`, `elsewhere` and `unstamped` pin it.
- `machineLandings` passes the current name and then the former names to `read`. Sub-test `PassesTheCurrentAndFormerNamesToTheRead` pins it.
- `machineLandings` keeps the name error's way forward, adds no hostname fix, and never calls `read` on failure. Sub-test `FailingNameReadKeepsItsWayForwardAndReadsNothing` pins it.
- `thisMachine` keeps the name error's way forward and appends `lym batten run <slug>`. `TestRefuseStartOwnedElsewhere` row `UnreadableMachineNameRefusesKeepingItsWayForward` pins it.
- The empty-name branch removed from `thisMachine` needs no test, because `machinename.Name` never returns an empty name without an error.
- The shed build (`arm.go`) keeps the name error's way forward and appends `lym batten run`. `TestBuildShed_TellsTheMachine` row `RefusesAnUnknownName` pins it.
- The `machinename` import allowlist is pinned by `TestLeafInvariant_AllowlistOnly`. The `githubclient` allowlist is pinned by its own `leaf_enforcement_test.go`.
- The reworded `SetOwners` no-machine refusal has no test. It is a static message, and no test asserts the old wording.

**Findings**

1. **LOW / scope**: `FilePath` has no test for its unset case or for wrapping a relative-variable error with its way forward.
   - Location: `internal/machinename/machinename.go:42`; `internal/machinename/machinename_test.go` `TestFilePath`.
   - Evidence: the diff's `TestFilePath` sets both variables to one absolute directory and asserts only the success path. D3 and the Exports list make `FilePath` a public contract that the follow-up verb will rely on.
   - Fix: add two rows to `TestFilePath`, or a table version of it: all variables unset giving ok false with no error, and a relative `HOME` or `LOCALAPPDATA` giving an error that contains the variable name and "absolute path". Both need `t.Setenv`, so the test stays non-parallel.

2. **NIT / scope**: `absoluteUserConfigDirFor` has no row for a relative `XDG_CONFIG_HOME` with an empty `HOME`.
   - Location: `internal/standalonestate/standalonestate_test.go` `TestUserConfigDir`; `standalonestate.go:211`.
   - Evidence: the `Unset` row leaves `XDG_CONFIG_HOME` empty, so the branch where a discarded relative `XDG_CONFIG_HOME` leaves no fallback is never run.
   - Fix: add a row with `goos` "linux", `xdgConfigHome` ".xdg", empty `home`, `wantPlain` `filepath.Join(".xdg", "lym")`, and an empty `wantAbsolute`.

No redundant tests found. `TestResolveNames_UnreadableFileIsAnError` covers a read failure the table cannot express, and `TestCheckName` and `TestUserConfigDir` are single tables.

Tool calls: none; my context already held the full diff and the surrounding code.
