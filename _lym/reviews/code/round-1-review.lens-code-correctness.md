```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens. I ran each changed function and branch on a concrete input.

**Per function, case run, verdict**

- `standalonestate.userConfigDirFor`
  - Windows with `LOCALAPPDATA` unset returns ("", false).
  - A relative `XDG_CONFIG_HOME` is taken as given.
  - An empty `XDG_CONFIG_HOME` falls back to HOME, and with HOME unset returns ("", false).
  - This is the old `cacheDir` rule line for line, so `CredentialDir` returns the same directory as before. **Correct.**
- `standalonestate.absoluteUserConfigDirFor`
  - Linux with an absolute XDG uses it.
  - A relative XDG with `HOME=/h` gives `/h/.config/lym`.
  - A relative XDG with HOME unset gives ("", false, nil).
  - A relative HOME gives an error wrapping `errRelativeHome`, whose text now starts "standalonestate: HOME must be absolute".
  - An absolute XDG with a relative HOME gives no error, which is correct because the fallback is unused.
  - Windows with a relative `LOCALAPPDATA` gives an error naming it.
  - The empty-string case goes through `filepath.IsAbs("")`, which is false, so it falls to the HOME branch. **Correct, and it matches D2.**
- `errRelativeHome` text change
  - The grep for the old text, "home directory must be absolute", found no remaining reference. **Correct.**
- `machinename.FilePath`
  - A relative `HOME` returns an error with the way forward, `ok` false and empty paths.
  - An unset variable returns "", "", false, nil.
  - A resolved directory returns `machine.json` and `machine.lock`. **Correct.**
- `machinename.ReadFile`
  - A missing file returns (File{}, false, nil).
  - A directory at the path returns (File{}, false, err). The `false` is harmless because callers check the error first.
  - Bad JSON returns exists true with the error.
  - The JSON `null` decodes to a zero `File` that exists, which `resolveNames` then rejects through `CheckName("")`.
  - Unknown keys are ignored. **Correct.**
- `machinename.resolveNames`, each branch:
  - A locate error is returned raw. `FilePath` already attaches the way forward, so this is **correct**.
  - `!ok` falls back to the hostname rule and returns nil former. **Correct.**
  - A read or decode error is wrapped with the path and `setNameWayForward`, and the hostname is never used. **Correct.**
  - A missing file falls back to the hostname rule. **Correct.**
  - An empty or invalid `name` is an error naming the path. **Correct.**
  - `names=["neve","","Old.Box","osl46"]` with `name=osl46` gives `former=[neve, Old.Box]`, which drops empties and the current name and keeps an invalid entry, as D3 asks. **Correct.**
  - Duplicate entries in `names` survive into `former`. This is harmless, because `slices.Contains` in `DobbyLandings` ignores duplicates.
- `machinename.nameFromHostname`
  - A hostname read failure returns an error with a way forward naming `lym machine name`.
  - An empty hostname goes to `FromHostname`, which errors. That message (`machinename.go:110`) says only "give the machine a hostname", without the `lym machine name` alternative. This is a minor inconsistency and still counts as a way forward (finding F1 below).
- `machinename.CheckName`
  - "" is invalid, and the `name[0]` index is not reached because `&&` short-circuits.
  - "-a" and "a-" are invalid.
  - "a" is valid.
  - A 63-character name is valid and a 64-character one is invalid.
  - A non-ASCII name is rejected by the rune loop.
  - `.` and `:` are rejected. **Correct.**
- `Name` and `Names` both call `resolveNames(machineFile, os.Hostname)`, read on every call with no cache. **Correct.**
- `battencli.DobbyLandings`
  - A history entry with `Machine` "" and a `machines` list holding no empty entry is never counted, because `Names` drops empties and the current name is non-empty.
  - A nil `machines` list counts nothing. **Correct.**
- `beamcli.machineLandings`
  - `append([]string{current}, former...)` builds a fresh slice with no aliasing.
  - On a name error it returns the error and never calls `read`. **Correct.**
- `battencli.thisMachine`
  - The empty-name branch is gone.
  - Production `Name` never returns ("", nil), because `FromHostname` errors on empty and `CheckName` rejects an empty file name.
  - `arm.go specFor` never had an empty check, so it is unchanged in behaviour.
  - Both wrappers keep the inner way forward and append only "once the name reads, re-run …". **Correct.**
- `boardengine.SetOwners` message edit: wording only. **Correct.**
- `githubclient.cacheDir` delegates to `UserConfigDir`, and `CredentialDir` is unchanged in result. **Correct.**

**Findings**

- **F1, LOW, consistency:** `internal/machinename/machinename.go:110`. The `FromHostname` empty-hostname error says "give the machine a hostname" but does not name `lym machine name`. `nameFromHostname` (`machinename.go:159`) names both ways forward for an unreadable hostname.
  - D3 says every `machinename` error names its own way forward, and a machine with an empty hostname can use the machine file instead.
  - This is not a defect against the record's letter, because a way forward is named. Fix: add "or name it with `lym machine name <name>`" to the line 110 error.
  - Counter-point: `FromHostname` also serves other callers, and a test may pin its text, so this may be left as is.

I found no `design`-class defect in the changed code.

**Tool calls**
1. grep for "home directory must be absolute": to see whether any test or doc asserted the old `errRelativeHome` text, which my context lacked. None remain.
2. Same call, second grep for "hostname" in non-test production code outside `machinename`: to see whether any other wrapper still appends a hostname fix. Only doc comments in `pairteardown` (`doc.go:253`, `originseams.go:89`) mention "an empty hostname included".
